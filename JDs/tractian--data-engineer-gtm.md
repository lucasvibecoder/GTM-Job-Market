---
company: "TRACTIAN"
title: "Data Engineer (GTM)"
salary_low:
salary_high:
salary_type: base
url: "https://www.linkedin.com/jobs/view/gtm-analytics-engineer-at-tractian-4243116017"
location: "Atlanta, GA"
work_style:
company_stage: "growth"
company_type:
  - vertical-saas
implicit_stack: []
yoe_required: 5
archetype: "gtm-systems-architect"
archetype_secondary:
topics:
  - data-engineering
  - agentic-engineering
tools:
  - BigQuery
  - Clay
  - ZoomInfo
  - Python
  - JavaScript
  - Kafka
  - Airflow
  - dbt
skills_required:
  - 5+ years in data engineering, data analytics, or data science (preferably high-growth B2B SaaS)
  - API integrations and at least one scripting language (Python, Go)
  - Managing large datasets and transforming GTM insights into tactical recommendations
  - Problem-solving with bias for automation and scale
  - Cross-functional collaboration and communicating technical concepts to non-technical stakeholders
  - Comfortable working autonomously in a fast-paced environment
skills_preferred: []
lucas_annotations: []
fit_score:
gtm_adjacent: true
date_parsed: "2026-03-12"
---

## Summary

TRACTIAN builds an industrial maintenance platform (hardware + software) serving frontline workers — growth-stage, global GTM team across US, Brazil, and Mexico. The LinkedIn listing title is "GTM Analytics Engineer" but Clay extracted "Data Engineer (GTM)." This role architects the revenue data infrastructure: managing a customer data lake in BigQuery, building enrichment pipelines with Clay and ZoomInfo, incorporating LLM-based enrichment for predictive targeting, and orchestrating batch/streaming ingestion with Kafka and Airflow. Heavy data engineering applied to GTM — not a traditional GTM Engineer role.

## Responsibilities

- Architect and manage data pipelines and infrastructure for GTM systems (BigQuery, Clay, ZoomInfo, cloud storage, ETL frameworks)
- Develop, test, and maintain scripts and microservices (Python, JavaScript/Node.js) for data acquisition and processing workflows
- Design and manage a unified GTM data lake merging raw and enriched datasets into a single source of truth for segmentation, ICP scoring, and territory assignments
- Incorporate LLM-based enrichment pipelines to augment account/contact metadata and drive predictive targeting
- Build and maintain batch and streaming ingestion pipelines using Apache Beam, Kafka, or Pub/Sub with Airflow orchestration
- Develop semantic layers and star/snowflake schemas for GTM analytics in dbt
- Partner with RevOps, Enablement, and Sales Leadership to improve seller workflows, define KPIs, and ensure data consistency
- Identify areas for automation and workflow efficiency; build scalable solutions for repetitive GTM processes
- Design high-cardinality indexing strategies for real-time ICP scoring and territory assignment

## Requirements

### Must-Have
- 5+ years in data engineering, data analytics, or data science (preferably high-growth B2B SaaS)
- Comfort with API integrations and at least one scripting language (Python, Go)
- Experience managing large datasets and transforming GTM insights into tactical recommendations
- Excellent problem-solving skills and bias for automation and scale
- Cross-functional collaboration and communicating technical concepts to non-technical stakeholders
- Comfortable working autonomously in a fast-paced environment

### Nice-to-Have
- None explicitly listed

## Key Concepts

- **GTM data lake**: A unified repository merging raw and enriched datasets (from Clay, ZoomInfo, CRM, product usage) into a single source of truth. Enables ICP scoring, territory assignment, and segmentation from one place rather than scattered across tools.
- **LLM-based enrichment pipelines**: Using large language models to automatically augment account and contact metadata — inferring company attributes, categorizing leads, or generating targeting signals that traditional rule-based enrichment can't capture.
- **Star/snowflake schema**: Data modeling patterns that organize GTM analytics into fact tables (events, transactions) surrounded by dimension tables (accounts, contacts, campaigns). Built in dbt to ensure BI tools query from pre-aggregated, materialized datasets rather than raw data.

## Links
- **Archetype**: [[gtm-systems-architect]]
- **Company Type**: [[vertical-saas]]
- **Topics**: [[data-engineering]] | [[agentic-engineering]]
- **Tools**: [[BigQuery]] | [[Clay]] | [[ZoomInfo]] | [[Python]] | [[JavaScript]] | [[Kafka]] | [[Airflow]] | [[dbt]]
