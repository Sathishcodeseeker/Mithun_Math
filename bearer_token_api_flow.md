# Bearer Token & Microsoft Graph API Flow

## Records vs Classes (DTO Pattern)

### How Records Work

Java `record` classes do **not** have traditional `getXxx()` / `setXxx()` methods. The compiler generates:
- **Accessor methods named after the component** (not `get`-prefixed): `email()`, `firstName()`, `entraOid()`
- Records are **immutable** — no setters exist

### Project Usage

**Request DTO (record):**
```java
public record CreateUserRequest(
    String entraOid, String email, String firstName,
    String lastName, String location, String module, String domain) {}
```

**Service consuming the record:**
```java
User newUser = User.builder()
    .entraOid(request.entraOid())    // accessor, not getEntraOid()
    .email(request.email())           // accessor, not getEmail()
    .firstName(request.firstName())
    .build();
```

**Record with custom method:**
```java
public record UpdateUserRequest(String domain, String module, Boolean active) {
  public boolean hasNoFields() {
    return domain == null && (module == null || module.isBlank()) && active == null;
  }
}
```

### Layer Pattern

| Layer | Type | Accessors | Setters |
|-------|------|-----------|---------|
| DTOs (request/response) | `record` | `fieldName()` (compiler-generated) | None (immutable) |
| Entities | `class` + Lombok | `getFieldName()` (Lombok) | `setFieldName()` (Lombok) |

### Re-assignment Scenario

Records are set once at construction and never mutated. For transformations:

```java
String domain = request.domain();                                         // read once
domain = (domain == null || domain.isBlank()) ? null : domain.strip();   // re-assign local variable
```

The record itself is never changed. Mutation flows through:
1. **Local variables** — for transforming request values
2. **Entity objects** — for persisting state changes
3. **New record construction** — for responses

---

## Graph API Flow (End to End)

### Architecture

```
┌──────────┐         ┌──────────────┐         ┌────────────────────┐
│ Frontend │──JWT───→│ Spring Boot  │──OBO───→│ Microsoft Graph    │
│ (Angular)│         │ (this app)   │         │ API                │
└──────────┘         └──────────────┘         └────────────────────┘
      ▲                                              
      │ login                                        
      ▼                                              
┌──────────────────────┐                             
│ Azure AD / Entra ID  │                             
│ (token issuer)       │                             
└──────────────────────┘                             
```

### Step 1: HTTP Request Arrives

```
GET /api/v1/users/lookup?email=john@company.com
Authorization: Bearer <user's-access-token>
```

### Step 2: Controller

```java
public ResponseEntity<ApiResponse<UserLookupResponse>> lookupUser(
    @RequestParam String email, JwtAuthenticationToken principal) {
    
    String bearerToken = principal.getToken().getTokenValue();
    UserLookupResponse result = userService.lookupByEmail(email, bearerToken);
}
```

Spring Security validates the JWT first. The raw token value is passed downstream.

### Step 3: Service Delegates to GraphService

```java
return graphService.lookupByEmail(email, bearerToken);
```

### Step 4: OBO Token Exchange

The user's token is scoped for this API, not Graph. The OBO flow exchanges it:

```java
private String exchangeToken(String bearerToken) {
    OnBehalfOfParameters parameters = OnBehalfOfParameters.builder(
        Set.of("https://graph.microsoft.com/.default"),
        new UserAssertion(bearerToken)
    ).build();
    
    IAuthenticationResult result = msalClient.acquireToken(parameters).get();
    return result.accessToken();
}
```

This calls Azure AD token endpoint:
```
POST https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token
  grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
  client_id=<api-client-id>
  client_secret=<api-client-secret>
  assertion=<user's-bearer-token>
  scope=https://graph.microsoft.com/.default
  requested_token_use=on_behalf_of
```

### Step 5: Call Microsoft Graph

```java
graphRestClient
    .get()
    .uri(uriBuilder -> uriBuilder
        .path("/users")
        .queryParam("$filter", "mail eq 'john@company.com'")
        .queryParam("$select", "id,mail,givenName,surname,userPrincipalName,city,country")
        .build())
    .header("Authorization", "Bearer " + oboToken)
    .retrieve()
    .body(JsonNode.class);
```

Actual HTTP:
```
GET https://graph.microsoft.com/v1.0/users?$filter=...&$select=...
Authorization: Bearer <obo-token>
```

### Step 6: Map Response

```java
private UserLookupResponse mapToResponse(JsonNode user) {
    String id = textOrNull(user, "id");
    String mail = textOrNull(user, "mail");
    if (mail == null) mail = textOrNull(user, "userPrincipalName");
    String givenName = textOrNull(user, "givenName");
    String surname = textOrNull(user, "surname");
    String location = buildLocation(user);
    return new UserLookupResponse(id, mail, givenName, surname, location);
}
```

### Step 7: JSON Response

```json
{
  "data": {
    "entra_oid": "abc-123-...",
    "email": "john@company.com",
    "first_name": "John",
    "last_name": "Doe",
    "location": "Copenhagen, Denmark"
  },
  "meta": { "timestamp": "...", "request_id": "..." }
}
```

### GraphClientConfig Beans

```java
@Bean
RestClient graphRestClient(GraphApiProperties properties) {
    return RestClient.builder().baseUrl(properties.baseUrl()).build();
    // baseUrl = "https://graph.microsoft.com/v1.0"
}

@Bean
ConfidentialClientApplication msalClient(...) {
    // authority = "https://login.microsoftonline.com/{tenant-id}"
    // Used for OBO token exchange
}
```

### GraphApiProperties (record)

```java
@ConfigurationProperties(prefix = "entra.graph")
public record GraphApiProperties(String baseUrl, String scope) {}
```

Bound from `application.yml`:
```yaml
entra:
  graph:
    base-url: https://graph.microsoft.com/v1.0
    scope: https://graph.microsoft.com/.default
```

---

## Bearer Access Token — Full Lifecycle

### Phase 1: Token Acquisition (Frontend)

The Angular frontend authenticates via MSAL:

```
POST https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token
  scope=api://{api-client-id}/user_impersonation
```

The returned JWT contains:
- `iss`: `https://login.microsoftonline.com/{tenant-id}/v2.0`
- `aud`: `api://{api-client-id}` (this backend's audience)
- `oid`: user's Entra object ID
- `roles`: app roles assigned in Entra
- Signed by Microsoft's private key

### Phase 2: Token Validation (Spring Security — automatic)

**Config:**
```yaml
spring.security.oauth2.resourceserver.jwt:
  issuer-uri: https://login.microsoftonline.com/${ENTRA_TENANT_ID}/v2.0
  audiences: ${ENTRA_API_CLIENT_ID}, api://${ENTRA_API_CLIENT_ID}
```

**SecurityConfig:**
```java
.oauth2ResourceServer(oauth2 ->
    oauth2.jwt(jwt -> jwt.jwtAuthenticationConverter(jwtAuthenticationConverter())))
```

On every request, Spring Security automatically:

1. **Extracts** the `Authorization: Bearer <token>` header
2. **Fetches OIDC metadata** from `issuer-uri/.well-known/openid-configuration` (cached)
3. **Downloads JWKS** (public keys) from the `jwks_uri` in that metadata (cached)
4. **Validates signature** — verifies the JWT was signed by Microsoft's private key
5. **Validates claims** — checks `iss`, `aud`, `exp`, `nbf`
6. **Creates** a `JwtAuthenticationToken` and puts it in SecurityContext

If any step fails → **401 Unauthorized** (request never reaches controller).

### Phase 3: JwtDecoder Warmup

Steps 2-3 require HTTP calls on first use. Without warmup, the first request times out with a 401.

```java
@EventListener(ApplicationReadyEvent.class)
public void warmup() {
    jwtDecoder.decode(dummyToken);  // triggers OIDC + JWKS fetch
    // Exception is expected — keys are now cached
}
```

### Phase 4: Authority Extraction

```java
converter.setJwtGrantedAuthoritiesConverter(jwt -> {
    List<String> roles = jwt.getClaimAsStringList("roles");
    return roles.stream()
        .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
        .toList();
});
```

Entra app roles (e.g., `Admin`, `User`) become Spring authorities (`ROLE_Admin`, `ROLE_User`).

### Phase 5: Controller Access

```java
public ResponseEntity<?> lookupUser(..., JwtAuthenticationToken principal) {
    String bearerToken = principal.getToken().getTokenValue();   // raw token for OBO
    String adminOid = principal.getToken().getClaimAsString("oid"); // user's Entra OID
}
```

### Phase 6: OBO Token Exchange

The user's JWT is scoped for this API (`aud = api://{client-id}`). It cannot call Graph directly. OBO exchanges it for a Graph-scoped token.

### Two Tokens, Two Audiences

| Token | Audience | Issued by | Used for |
|-------|----------|-----------|----------|
| **User's JWT** | `api://{api-client-id}` (this app) | Azure AD → frontend | Authenticating requests to this backend |
| **OBO token** | `https://graph.microsoft.com` | Azure AD → this backend | Calling Graph API as the user |

### Security Constraints

| Check | Where | Effect |
|-------|-------|--------|
| Signature valid | Spring `JwtDecoder` | Rejects forged tokens |
| Issuer matches tenant | `issuer-uri` config | Rejects tokens from other tenants |
| Audience matches this app | `audiences` config | Rejects tokens meant for other APIs |
| Not expired | Spring auto-check | Rejects stale tokens |
| Stateless sessions | `SessionCreationPolicy.STATELESS` | No session cookies — every request must carry a fresh JWT |
| CORS restricted | `LocalSecurityConfig` | Only `localhost:4200/4210` in local profile |

### Session Policy: Stateless

```java
.sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
```

No server-side session is created. Every request must carry its own Bearer token. This is the standard for REST APIs behind SPAs.

### Permitted Endpoints (no token required)

```java
.requestMatchers("/api/health", "/swagger-ui/**", "/v3/api-docs/**", "/actuator/**")
    .permitAll()
```

All other `/api/**` endpoints require a valid Bearer token.
