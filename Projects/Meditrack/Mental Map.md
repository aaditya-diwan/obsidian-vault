 ---
  MediTrack — Mental Map

  The Big Picture Analogy

  Think of MediTrack as a hospital with three departments, each operating independently with their own filing cabinets,
  but communicating via an internal announcement system (Kafka). A reception desk (Kong) handles all visitors from
  outside.

  ---
  Top-Level Layout

  MediTrack/
  ├── services/
  │   ├── patient-service/       ← Department 1: Patient Registration
  │   ├── labrotary-service/     ← Department 2: Lab & Testing
  │   └── insurance-service/     ← Department 3: Insurance & Billing
  ├── infrastructure/            ← Shared IT setup (DB init scripts)
  ├── monitoring/                ← CCTV & dashboards (Prometheus, Grafana)
  └── docker-compose.yml         ← Blueprint to spin everything up

  ---
  Inside Every Service — Same 4-Room Floor Plan

  Every service follows the exact same internal structure. Learn it once, know all three.

  src/main/java/com/meditrack/<service>/
  │
  ├── interfaces/          ← FRONT DOOR
  ├── application/         ← BRAIN
  ├── domain/              ← RULEBOOK
  └── infrastructure/      ← PLUMBING

  Analogy: A restaurant.
  - interfaces = the waiter (takes orders, delivers food)
  - application = the kitchen manager (coordinates the workflow)
  - domain = the recipe book (the rules of what a dish is)
  - infrastructure = the kitchen equipment (stoves, fridges, delivery trucks)

  ---
  Room 1: interfaces/ — The Front Door

  What's here: REST controllers, error handlers

  What it does: Receives HTTP requests from the outside world, validates the shape of the request, calls into
  application/, and formats the response back.

  Analogy: The waiter. Knows nothing about how food is made — just takes the order and brings it back.

  interfaces/rest/
  ├── PatientController.java        ← POST /patients, GET /patients/{id}
  ├── GlobalExceptionHandler.java   ← "Sorry, something went wrong" responses
  └── ErrorResponse.java            ← Standard error format

  Key rule: Controllers never touch the database or Kafka directly. They only call application services.

  ---
  Room 2: application/ — The Brain

  What's here: Use cases, application services, DTOs, mappers, exceptions

  What it does: Orchestrates the actual work. "A lab order came in — validate it, save it, then fire an event." This is
  where the workflow lives.

  Analogy: The kitchen manager. Doesn't cook the food themselves, but directs the chefs, manages timing, and ensures the
   right dish goes to the right table.

  application/
  ├── usecase/
  │   ├── CreateLabOrderUseCase.java     ← one file = one business operation
  │   └── SubmitLabResultUseCase.java
  ├── service/
  │   └── LabOrderApplicationService.java  ← implements the use cases
  ├── dto/
  │   ├── LabOrderRequest.java    ← what comes IN (from controller)
  │   └── LabOrderResponse.java   ← what goes OUT (to controller)
  ├── mapper/
  │   └── LabOrderMapper.java     ← converts between DTOs and domain models
  └── exception/
      └── LabOrderNotFoundException.java

  DTOs vs Domain models: DTOs are shaped for the API (what the outside world sees). Domain models are shaped for the
  business rules (what the code thinks in). The mapper translates between them — so your API shape and your internal
  logic can evolve independently.

  ---
  Room 3: domain/ — The Rulebook

  What's here: Models, value objects, repository interfaces

  What it does: Defines what things are and what rules they must follow. A Patient has an MRN. An SSN must be formatted
  correctly. A LabResult can be critical or not. No framework code lives here — pure Java.

  Analogy: The recipe book. It describes what a dish is, what ingredients it requires, what makes it valid. It doesn't
  care whether you're cooking on gas or electric.

  domain/
  ├── model/
  │   ├── Patient.java             ← core entity
  │   ├── MedicalRecord.java
  │   └── valueobjects/
  │       ├── MRN.java             ← Medical Record Number (typed, not just a String)
  │       ├── SSN.java             ← Social Security Number with validation
  │       └── PatientId.java
  └── repository/
      └── PatientRepository.java   ← interface only — "I need to save/find patients"
                                      (HOW is defined in infrastructure/)

  Why value objects? Instead of storing SSN as a plain String everywhere, you wrap it in SSN.java which enforces format
  rules. Bugs become compile errors instead of runtime surprises.

  ---
  Room 4: infrastructure/ — The Plumbing

  What's here: Kafka producers/consumers, JPA entities, repository implementations, external clients, outbox

  What it does: All the messy real-world integrations. This is where the domain's abstract PatientRepository gets
  implemented using actual JPA/SQL. Where Kafka messages get sent and received.

  Analogy: The kitchen equipment. The recipe says "refrigerate overnight" — the fridge is what actually does it. Swap
  the fridge brand, the recipe stays the same.

  infrastructure/
  ├── persistence/
  │   ├── entity/
  │   │   └── PatientEntity.java         ← JPA/DB representation (has @Column, @Table etc.)
  │   ├── repository/
  │   │   └── JpaPatientRepository.java  ← Spring Data JPA interface
  │   ├── impl/
  │   │   └── PatientRepositoryImpl.java ← bridges domain interface → JPA
  │   └── mapper/
  │       └── PersistencePatientMapper.java ← converts domain model ↔ DB entity
  │
  ├── messaging/
  │   ├── PatientEventProducer.java      ← sends events to Kafka
  │   ├── consumer/
  │   │   └── LabOrderEventConsumer.java ← receives events from Kafka
  │   ├── event/
  │   │   └── LabTestOrderedEvent.java   ← the event's data shape
  │   └── config/
  │       └── KafkaConsumerConfig.java   ← retry logic, error handling, DLT
  │
  ├── outbox/                            ← Lab Service only
  │   ├── OutboxEvent.java               ← "unsent mail" stored in DB
  │   └── OutboxRelay.java               ← background job that delivers it
  │
  └── external/                          ← Patient Service only (stubs)
      ├── InsuranceApiClient.java        ← future: call insurance APIs
      └── NotificationApiClient.java     ← future: send SMS/email

  Why two mappers? Notice there are mappers in both application/ and infrastructure/persistence/. That's intentional:
  - application/mapper — translates between DTO ↔ Domain model (API boundary)
  - persistence/mapper — translates between Domain model ↔ DB entity (storage boundary)

  The domain model is never directly exposed to the API or the database.

  ---
  How Data Flows End-to-End

  Using "Doctor orders a blood test" as the example:

  1. HTTP POST /api/v1/patients/{ssn}/order-labs
          ↓
  2. interfaces/ PatientLabOrderController
     — parses the request body into a DTO
          ↓
  3. application/ PatientApplicationService
     — validates the request
     — calls domain logic
     — tells infrastructure to publish an event
          ↓
  4. infrastructure/messaging/ PatientEventProducer
     — builds a LabTestOrderedEvent
     — sends it to Kafka topic: "patient-events"
          ↓
  5. [Kafka carries the message]
          ↓
  6. infrastructure/messaging/consumer/ LabOrderEventConsumer  (Lab Service)
     — receives the event
     — calls into Lab Service's application layer
          ↓
  7. application/ LabOrderApplicationService
     — creates a LabOrder domain object
     — persists it via the domain repository interface
          ↓
  8. infrastructure/persistence/ LabOrderRepositoryImpl
     — converts domain LabOrder → LabOrderEntity
     — saves to PostgreSQL via JPA
          ↓
  9. Lab technician submits result via POST /api/v1/lab/results
          ↓
  10. Lab Service writes result to DB + writes to outbox_events (same transaction)
          ↓
  11. OutboxRelay (runs every 5 seconds)
      — reads pending outbox events
      — publishes "lab.results.available.v1" to Kafka
      — marks outbox event as PROCESSED

  ---
  The Supporting Cast

  ┌────────────────────┬────────────────────────────┬──────────────────────────────────────────────────────────┐
  │     Component      │          Analogy           │                           Role                           │
  ├────────────────────┼────────────────────────────┼──────────────────────────────────────────────────────────┤
  │ Kong               │ Reception desk             │ Routes external HTTP traffic to the right service        │
  ├────────────────────┼────────────────────────────┼──────────────────────────────────────────────────────────┤
  │ Kafka              │ Hospital intercom          │ Carries announcements between departments asynchronously │
  ├────────────────────┼────────────────────────────┼──────────────────────────────────────────────────────────┤
  │ PostgreSQL         │ Each dept's filing cabinet │ Separate DB per service — they never share               │
  ├────────────────────┼────────────────────────────┼──────────────────────────────────────────────────────────┤
  │ Redis              │ Sticky notes on the wall   │ Short-term cache so you don't re-fetch the same data     │
  ├────────────────────┼────────────────────────────┼──────────────────────────────────────────────────────────┤
  │ Jaeger             │ Security camera replay     │ Records the full path a request took across services     │
  ├────────────────────┼────────────────────────────┼──────────────────────────────────────────────────────────┤
  │ Prometheus/Grafana │ Vital signs monitor        │ Tracks metrics (request rate, error rate, latency)       │
  └────────────────────┴────────────────────────────┴──────────────────────────────────────────────────────────┘

  ---
  One-Line Summary Per Layer

  ┌─────────────────┬────────────────────────────────────────────────────────┐
  │      Layer      │                      One-line job                      │
  ├─────────────────┼────────────────────────────────────────────────────────┤
  │ interfaces/     │ Accept requests, return responses — nothing else       │
  ├─────────────────┼────────────────────────────────────────────────────────┤
  │ application/    │ Orchestrate the workflow for one business operation    │
  ├─────────────────┼────────────────────────────────────────────────────────┤
  │ domain/         │ Define what things are and what rules they follow      │
  ├─────────────────┼────────────────────────────────────────────────────────┤
  │ infrastructure/ │ Connect the domain to the real world (DB, Kafka, HTTP) │
  └─────────────────┴────────────────────────────────────────────────────────