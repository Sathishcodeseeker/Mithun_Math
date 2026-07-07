# SATHISH K

**AI Integration Specialist | Forward Deployed Engineer | Distributed Systems Architect | Agentic AI Lead Developer**

Mobile: +91 6380245564  
Email: sathishr951@gmail.com  
Location: Bengaluru, India  
Education: B.E. Electronics and Communication Engineering, Anna University

## Profile

Forward-deployed AI and integration specialist with 12+ years of hands-on experience designing, building, and delivering scalable APIs, distributed backend systems, enterprise data platforms, and production-grade AI systems across aviation, banking, and insurance domains.

Currently leading AI engineering for aeronautical data extraction and live human-in-the-loop verification workflows at Jeppesen ForeFlight, building agentic and deterministic systems for AIRAC source documents using LangGraph, FastAPI, WebSocket streaming, Redis, Azure AI Document Intelligence, and frontier LLM platforms. Experienced in translating operational pain points into deployable AI-assisted workflows through direct collaboration with customers, domain experts, product stakeholders, and engineering teams.

Strong background in microservices architecture, REST APIs, OAuth-based security, event-driven systems, ELT pipelines, Databricks Lakehouse, feature engineering, statistical modeling, and regulated system design. Regularly evaluates and integrates OpenAI, Azure OpenAI, and Anthropic Claude capabilities for enterprise AI use cases, with focus on reliability, governance, observability, model evaluation, human-in-the-loop controls, and production readiness.

## Core Competencies

- AI Integration Engineering
- Forward Deployed Engineering
- Agentic AI Engineering
- Distributed Systems
- Microservices Architecture
- LLM Application Architecture
- Human-in-the-Loop AI Systems
- LLM Evaluation and Observability
- Enterprise API Integration
- Event-Driven Architecture
- Databricks Lakehouse
- Feature Engineering
- Technical Product Ownership

## Domain Knowledge

- Aviation: Flight planning, VFR, IFR, NOTAMs, weather, airspace restrictions, route optimization, operational compliance
- Banking: Partner integrations, card transaction risk scoring, fraud detection, secure API ecosystems
- Insurance: Actuarial experimentation, statistical analysis, underwriting and claims analytics

## Legal and Compliance

- GDPR
- DO-200B aviation data quality and processing awareness
- Aviation safety governance
- Secure API authentication and authorization
- Auditability and traceability for regulated workflows

## Technical Skills

| Area | Skills |
| --- | --- |
| Programming | Java 21, Python, SQL |
| Backend and APIs | Spring Boot, FastAPI, REST APIs, Angular novice |
| AI and LLM Platforms | OpenAI, Azure OpenAI, Anthropic Claude |
| Agentic AI | LangGraph, MCP, OpenEval, LangChain, A2A, RAG, human-in-the-loop workflows |
| Data Engineering | PySpark, Databricks, Delta Lake, ELT pipelines |
| Data Analysis | Pandas, GeoPandas, Shapely, statistical modeling |
| Databases | PostgreSQL, Oracle |
| Cloud | Microsoft Azure |
| Security | Azure Entra ID, OAuth 2.0, SAML, MFA |
| Observability | Databricks UI, Azure Log Analytics, LangSmith |
| Streaming | Azure Event Hub via Kafka, Apache Kafka |
| Containers | Kubernetes, Docker, AKS |
| DevOps | GitLab, Bitbucket, GitHub, Azure DevOps, GitHub Actions, CI/CD pipelines |
| Delivery | Agile, JIRA, ServiceNow, technical product ownership |

## Current Assignment

Lead a 7-member engineering team across two parallel workstreams: BAU and post-production support for an enterprise microservices and data platform built on Spring Boot, PySpark, and Databricks, alongside iterative development of an Agentic AI aeronautical data extraction system using LangGraph, frontier LLM models, FastAPI, WebSocket streaming, Redis, Azure AI Document Intelligence, and PostgreSQL.

- Work as Technical Product Owner and hands-on Lead Developer, defining NFRs, shaping architecture, building evaluation frameworks, and mentoring engineers through delivery.
- Operate in a forward-deployed model by working directly with aviation stakeholders, operational users, and engineering teams to identify workflow bottlenecks and convert them into deployable AI-assisted capabilities.
- Lead enterprise AI integration across OpenAI, Azure OpenAI, and Anthropic Claude capabilities, focusing on model evaluation, safety, explainability, governance, observability, and production deployment.
- Translate customer-facing operational requirements into agentic workflows, deterministic validation layers, human-in-the-loop approval mechanisms, and measurable product outcomes.

## Professional Experience

### Jeppesen ForeFlight

**Technical Lead / Product Owner**  
June 2022 - Present | Bengaluru, India  
Jeppesen ForeFlight spun off as an independent entity from Boeing India on November 1, 2025.

#### Assignment 1: AI-Driven Aeronautical Data Extraction

**Live Human-in-the-Loop Verification for AIRAC Source Documents**

Architected a real-time agentic AI extraction system for aeronautical source documents and AIRAC cycle data, replacing static batch-extraction-then-review workflows with a live streaming interface that enables human verification of AI-extracted data as it is generated, while preserving deterministic accuracy controls and data quality standards modeled on DO-200A/B principles.

**Business Impact**

- Reduced document verification cycle time from an estimated 3-4 hours of manual page-by-page review to under 30 minutes using a live WebSocket-streamed extraction interface with in-flight human verification.
- Enabled 20+ page AIRAC cycle-over-cycle comparison through a deterministic diff engine, cutting manual cross-referencing effort by an estimated 80%+ using automated, color-coded change detection.
- Achieved 90%+ field-level extraction accuracy on structured text fields including coordinates, frequencies, and identifiers using a hybrid OCR and LLM reasoning pipeline validated against a 100+ example evaluation set.
- Established a self-improving reliability loop where every human correction enriches the evaluation dataset, supporting continuous prompt and model performance tracking without separate labeling effort.
- Reduced human review burden to an estimated 10-15% of extracted fields through confidence-tiered routing.
- Maintained full correction provenance including original AI value, corrected value, reviewer, and timestamp to support audit-trail and regulatory governance requirements.

**Technical Contributions**

- Architected a LangGraph-based agentic pipeline with native checkpointing, enabling stateful pause, correct, and resume behavior during extraction without data loss or reprocessing.
- Designed a dual-channel WebSocket protocol unifying live extraction-state streaming and conversational chatbot interaction over a single connection.
- Built a synced-scroll split-pane UI combining PDF source view, live extraction panel, and chat with auto-follow and manual-override navigation.
- Implemented a deterministic diff engine for structured cycle-over-cycle comparison, decoupling exact-match logic in code from LLM-based narrative summarization.
- Evaluated and benchmarked extraction backends including Claude and Gemini VLM reasoning, specialized OCR models, and Azure AI Document Intelligence to select accuracy, cost, and latency tradeoffs per field type.
- Designed Redis-backed session state and pub/sub architecture to synchronize the extraction pipeline, correction handler, and chatbot around a single source of truth.

**Key Technologies**  
LangGraph, Python, FastAPI, WebSocket, Redis, Claude API, Azure AI Document Intelligence, Pydantic, React, PostgreSQL

#### Assignment 2: JAD Online

**VFR and IFR Domain Modernization**

Led modernization of the VFR and IFR domains within JAD, migrating legacy aviation data, screens, and workflows from a monolithic Oracle-based system into a cloud-native ecosystem built on a governed Databricks Lakehouse and domain-driven microservices.

**Business Impact**

- Modernized VFR and IFR data flows by migrating data to a Databricks Lakehouse, transforming screens into domain-driven microservices, and re-architecting workflows using event-driven patterns.
- Automated VFR and IFR reconciliation across 295 airport regions, eliminating 10-15 manual analyst hours per week per analyst.
- Improved reliability, fault isolation, and near real-time aviation data propagation to airline customers.
- Established a scalable analytics-ready VFR and IFR data foundation for future ML and advanced airspace analytics.

**Technical Contributions**

- Designed and implemented a medallion Lakehouse architecture across Bronze, Silver, and Gold layers for governed VFR and IFR data processing.
- Built PySpark ETL pipelines for incremental ingestion, validation, transformation, and publishing of aviation datasets.
- Implemented change data capture mechanisms for near real-time VFR and IFR synchronization.
- Developed geospatial airspace classification logic using GeoPandas and Shapely.
- Optimized Spark workloads, reducing VFR and IFR query times by approximately 60-70%.

**Key Technologies**  
Databricks, PySpark, Oracle, Python, GeoPandas, Shapely, Java, Spring Boot, Azure Event Hub, AKS, PostgreSQL, Azure Entra ID, Feature Engineering, GitLab CI/CD

### Citi Bank, Singapore

**Senior Software Engineer**  
August 2018 - June 2022 | Chennai, India

#### Assignment 1: Partnership Solutions

**ESB Modernization and Microservices Transformation**

Contributed to modernization of a tightly coupled ESB-based integration layer into a domain-driven microservices architecture supporting external partner ecosystems including Diners Club, PayPal, and Kogan.

**Business Impact**

- Resolved scalability and maintainability constraints in legacy ESB integrations by decomposing monolithic orchestration flows into domain-aligned microservices.
- Strengthened partner integration security through OAuth 2.0, MFA, and secure API gateway configurations.
- Improved resilience by transitioning synchronous ESB orchestration into event-driven communication patterns.

**Technical Contributions**

- Contributed to domain decomposition and bounded context definition for the partner integration layer.
- Designed and implemented event-driven components to replace synchronous ESB orchestration flows.
- Developed REST APIs secured with OAuth 2.0 and MFA flows.
- Collaborated with Information Security teams to remediate cyber vulnerabilities and harden API gateway configurations.
- Participated in continuity-of-business activities, disaster recovery planning, UAT, and production rollouts.

**Key Technologies**  
Java, Spring Boot, Apache Kafka, REST APIs, OAuth 2.0, Microservices, Domain-Driven Design

#### Assignment 2: Threat Modeling

**Machine Learning-Based Credit Card Transaction Risk Scoring**

Worked on a real-time fraud detection system designed to evaluate credit card transactions at swipe time using machine learning-based risk scoring models.

**Business Impact**

- Enabled low-latency transaction-level risk scoring before authorization to reduce fraud at point of swipe.
- Improved fraud detection precision by operationalizing behavioral and historical transaction patterns.
- Supported fraud operations teams by improving alert accuracy, threshold tuning, and response efficiency.

**Technical Contributions**

- Designed and implemented real-time inference pipelines to serve XGBoost risk scores at transaction swipe time.
- Built feature engineering components to extract behavioral patterns from high-volume transaction streams.
- Collaborated with data scientists on model training workflows, feature selection, validation logic, and production readiness.
- Integrated trained model artifacts into the scoring engine and connected outputs with downstream fraud detection and alerting systems.
- Developed custom business rules layered on top of ML model scores to tune risk thresholds.

**Key Technologies**  
Python, FastAPI, Feature Engineering, Apache Spark, PySpark MLlib, Spring Boot, CI/CD

### Cognizant - Client: Liberty Mutual Insurance, USA

**Associate**  
July 2016 - August 2018 | Coimbatore, India

#### Actuarix

**Statistical Analysis Platform for Actuarial Experimentation**

Designed and developed a self-service statistical analysis platform that enabled actuaries to perform A/B testing, t-tests, hypothesis testing, and statistical experimentation on large-scale medical insurance datasets.

**Business Impact**

- Built a web-based statistical analysis platform used by 50+ actuaries for hypothesis testing and experimentation.
- Enabled actuaries to validate pricing strategies, underwriting assumptions, and claim trends without engineering dependency.
- Improved consistency and accuracy of actuarial calculations through standardized statistical methods and automated validation checks.

**Technical Contributions**

- Built REST APIs and UI workflows using Spring Boot and jQuery to support actuarial experiment lifecycle management.
- Implemented asynchronous processing for long-running statistical jobs.
- Developed statistical modules for t-tests, ANOVA, chi-square tests, correlation analysis, and linear regression diagnostics.
- Built data profiling and validation frameworks covering missing values, outliers, range checks, business rules, and imputation logic.
- Collaborated with senior actuaries to clarify statistical requirements and validate outputs.

**Key Technologies**  
Java, Spring Boot, jQuery, Inferential Statistics, Oracle, Apache Commons Math

### Tech Mahindra - Client: Ford, Venezuela

**Junior Java Developer**  
July 2013 - February 2016 | Chennai, India

#### Ford Venezuela OAS

**Mainframe-to-Java Migration**

Worked on modernization of a legacy mainframe CICS application into a Java-based web platform for Ford Venezuela's Order Administration System.

**Business Impact**

- Migrated user-facing CICS terminals to a Java Struts web application, enabling browser-based access for 200+ users.
- Replaced 20+ year-old COBOL business logic with maintainable Java implementations.
- Improved security through modern authentication, session management, and encrypted data transmission.

**Technical Contributions**

- Analyzed legacy COBOL programs to extract business rules, data flows, and transaction logic.
- Translated core algorithms into object-oriented Java implementations while preserving functional equivalence.
- Designed and implemented 10+ web forms using a Struts-based MVC framework with jQuery validation.
- Authored technical specifications covering architecture, API contracts, database schemas, and migration mapping from COBOL copybooks to Java entities.

**Key Technologies**  
Java, Struts 2.x, jQuery, Oracle
