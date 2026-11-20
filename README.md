# rdv

Medical appointment booking service built to practice **observability** and **SRE** on a real-world application: instrumentation, SLOs, alerting, resilience, and progressive delivery.

> 🚧 Project under active development. This README describes what currently exists.

## Objectives

- Build a containerized web service along with its database.
- Instrument it end-to-end: metrics, logs, and traces.
- Make it reliable and prove it: SLOs, load testing, and chaos engineering.

## Datadog agent 

When enabled, the Datadog Agent collects and forwards service telemetry to Datadog. Configure it with a Datadog API key and site, and keep credentials in environment variables rather than committing them to the repository.