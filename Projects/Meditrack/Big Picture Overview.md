MediTrack — High-Level Overview

  What problem does it solve?

  MediTrack is a healthcare management backend — it digitizes and coordinates the core workflows of a medical facility:
  managing patients, ordering lab tests, processing results, and handling insurance policies.

  ---
  The Three Core Services

  Think of each service as an independent mini-application with its own database, its own API, and its own
  responsibilities:

  1. Patient Service (port 8081)
  The front door. Manages patient records (demographics, medical history). When a doctor orders a lab test through this
  service, it fires off an event to kick off the rest of the workflow.

  2. Lab Service (port 8082)
  Receives lab test orders, tracks their progress, and publishes results when they're ready. It also flags critical
  results — abnormal values that need immediate attention.

  3. Insurance Service (port 8083)
  Listens for new patients and manages their insurance policies and eligibility. Currently reacts to events; direct REST
   calls from other services aren't wired up yet.

  ---
  How They Talk to Each Other

  Two communication styles:

  - Kafka (async) — the primary path between services. Patient Service publishes events like "lab test ordered"; Lab
  Service and Insurance Service subscribe and react. No service waits on another.
  - REST APIs (sync) — only used by the UI/external clients hitting Kong (the API gateway). Services don't call each
  other's REST APIs directly.

  One reliability trick worth knowing: the Outbox Pattern in Lab Service. Before sending a Kafka event, it first writes
  the event to its own database in the same transaction. A background job then forwards it to Kafka. This ensures an
  event is never lost even if Kafka is temporarily down.

  ---
  The Supporting Infrastructure

  ┌──────────────────────┬────────────────────────────────────────────────────────┐
  │      Component       │                          Role                          │
  ├──────────────────────┼────────────────────────────────────────────────────────┤
  │ Kong                 │ API Gateway — single entry point for the UI/clients    │
  ├──────────────────────┼────────────────────────────────────────────────────────┤
  │ Kafka                │ Message bus between services                           │
  ├──────────────────────┼────────────────────────────────────────────────────────┤
  │ PostgreSQL           │ Each service has its own isolated database             │
  ├──────────────────────┼────────────────────────────────────────────────────────┤
  │ Redis                │ Per-service caching (patients, lab results, policies)  │
  ├──────────────────────┼────────────────────────────────────────────────────────┤
  │ Jaeger               │ Distributed tracing — follow a request across services │
  ├──────────────────────┼────────────────────────────────────────────────────────┤
  │ Prometheus + Grafana │ Metrics and dashboards                                 │
  └──────────────────────┴────────────────────────────────────────────────────────┘

  ---
  A Concrete End-to-End Flow

  Doctor submits a lab order via UI
    → Kong routes it to Patient Service (REST)
    → Patient Service publishes "lab.test.ordered" event to Kafka
    → Lab Service consumes it, creates the order in its DB
    → Lab technician submits results (REST → Lab Service)
    → Lab Service writes to outbox → Outbox Relay → Kafka
    → "lab.results.available" event published
    → (Future) Patient Service / Insurance Service react

  ---
  Architecture Philosophy

  The codebase follows hexagonal architecture — each service has a clean split between:
  - interfaces/ — REST controllers (entry points)
  - application/ — business logic / use cases
  - domain/ — core models
  - infrastructure/ — Kafka, DB, Redis, external clients

  This means you can change how a service talks to Kafka without touching the business logic, and vice versa.