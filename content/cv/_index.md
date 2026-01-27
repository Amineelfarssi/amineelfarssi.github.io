---
title: "Curriculum Vitae"
description: "Full CV - Amine El Farssi"
showDate: false
showReadingTime: false
showWordCount: false
showAuthor: false
sharingLinks: false
showTableOfContents: true
---

{{< button href="/files/Amine_El_Farssi_Resume.pdf" target="_blank" >}}
Download PDF
{{< /button >}}

---
<div style="float:right; width:140px; margin:0 0 1rem 1rem;">
  <img
    src="/img/amine.jpg"
    alt="Amine El Farssi"
    style="width:100%; height:auto; border-radius:9999px; object-fit:cover;"
  />
</div>

## Professional Summary

Senior AI Engineer with 5 years of AI/ML/DS experience, currently building and hardening production AI agents at one of Belgium's largest banks. Strong focus on agentic systems, AWS architecture, and operational excellence (evaluation, observability, and security).

---

## Professional Experience

### AI Engineer (GenAI / Agentic Systems) | KBC Bank & Insurance
**Leuven, Belgium** | 2023 - Present

- Built and shipped production AI agents for core operations, including an insurance claims-handling agent that automated information gathering and follow-ups; improved handler throughput by ~2–3× across ~300 claims/month (target claim type).
- Designed an event-driven agent runtime with state synchronized to enterprise backends using EventBridge (decoupled integration) and DynamoDB (state management) to meet reliability and consistency constraints.
- Delivered an autonomous mortgage/credit review agent that retrieves and applies credit policies/SOPs, integrates with CRM dispatching, supports human-in-the-loop, and handles ~150 questions/month (refinancing, variable-rate reviews, simulations).
- Built a reusable RAG platform on AWS enabling business units to ingest policies/FAQs, generate vector snapshots, and serve consistent retrieval for downstream agents and assistants.
- Implemented AgentOps + observability: golden datasets as CI/CD deployment gates, post-deploy evaluation for prompt changes, OTEL tracing/logging, and circuit breakers for integration failures.
- Led security hardening for agentic apps: prompt-injection guardrails, policy enforcement in system prompts, and red-teaming to identify vulnerabilities pre-release.

**Technologies:** Python, AWS (Bedrock, AgentCore, Lambda, EventBridge, SQS, DynamoDB, API Gateway, ECS/ECR, IAM), Docker, Terraform, OpenTelemetry (OTEL)

### Data Scientist (AML / Graph ML) | KBC Bank & Insurance
**Leuven, Belgium** | 2021 - 2023

- Built AML transaction-monitoring models using an ensemble (GBM / Random Forest / XGBoost) combined with graph-based scoring (diffusion over transaction/customer networks) to increase hit-rate and reduce investigation time versus rule-based detection.
- Engineered network features (e.g., PageRank, Jaccard similarity, neighborhood overlap) and applied community detection (Leiden) to surface suspicious clusters and patterns beyond single-transaction signals.
- Operated the system with governance-aligned thresholding (≈~2% of transactions flagged) on high-volume transaction flows (order of magnitude: 10^6/day).
- Implemented daily batch scoring pipelines using PySpark + Airflow, with experiment tracking and monitoring in MLflow.

**Technologies:** Python, PySpark/Spark, XGBoost, SQL, MLflow, Airflow, Graph analytics

### Big Data Engineer | JEMS Group
**France** | Mar 2021 - Aug 2021

- Migrated and standardized data pipelines on Databricks using the medallion architecture (bronze/silver/gold), delivering a clean curated layer ready for analytics warehousing.

**Technologies:** Databricks, Spark, SQL, Python

### Data Scientist  | Bioceanor
**Valbonne, France** | Jul 2020 - Mar 2021

- Built time-series forecasting models (LSTM) for sea-water quality signals (e.g., turbidity/chlorophyll/temperature), deployed via API and integrated with LoRa-based sensor pipelines; evaluated with MAE.

**Technologies:** Python, TensorFlow/Keras, LSTM, Time series, APIs, IoT/LoRa

---

## Education

### Master of Science in Applied Data Science
**Data ScienceTech Institute (DSTI)** | France | 2020 - 2021

Focus: Machine Learning, Deep Learning, Statistical Modeling, Big Data Technologies

### Diplome d'Ingenieur in Civil Engineering
**Strasbourg** | France

Equivalent to Master of Science. Foundation in analytical thinking, mathematical modeling, and complex systems engineering.

---

## Certifications

- AWS Certified Solutions Architect – Associate (2022)

---

## Technical Skills

| Category | Technologies |
|----------|-------------|
| **Agentic / GenAI** | AI Agents, RAG Systems, LangChain, LangGraph, Bedrock, Guardrails, Eval harnesses (golden datasets) |
| **AWS** | Bedrock, AgentCore, Lambda, API Gateway, EventBridge, SQS, DynamoDB, ECS/ECR, IAM, S3 |
| **Programming** | Python, SQL, TypeScript, Bash, HCL (Terraform) |
| **Data & MLOps** | PySpark/Spark, Airflow, MLflow, Docker, OpenTelemetry (OTEL), CI/CD |

---

## Languages

| Language | Proficiency |
|----------|-------------|
| French | Native |
| English | Fluent |
| Arabic | Fluent |
| Dutch | B1 (Currently Learning) |
