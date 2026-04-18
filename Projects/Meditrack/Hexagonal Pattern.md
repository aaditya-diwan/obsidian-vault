Hexagonal Architecture (aka "Ports & Adapters")

  The Core Idea

  Your business logic should not know or care how it receives input or where it sends output. It shouldn't know if it's
  talking to Kafka or RabbitMQ, PostgreSQL or MongoDB, a REST API or a CLI.

  You achieve this by putting the domain at the center and making everything else plug into it:

  [ Kafka Consumer ]  ──┐
  [ REST Controller ] ──┤──→  [ Application / Domain ]  ──┤──→  [ PostgreSQL ]
  [ CLI / Test ]      ──┘                                  └──→  [ Redis ]
                                                            └──→  [ Kafka Producer ]
      "Driving" side                                            "Driven" side
    (triggers your app)                                   (your app triggers these)

  The ports are interfaces defined by your domain. The adapters are the concrete implementations (Spring controller,
  Kafka listener, JPA repository, etc.).

  ---
  How It Looks in MediTrack

  In Lab Service, for example:

  interfaces/rest/LabResultController.java     ← adapter (driving)
  application/LabResultService.java            ← business logic (knows nothing about HTTP or Kafka)
  domain/LabResult.java                        ← pure model
  infrastructure/messaging/LabResultPublisher  ← adapter (driven)
  infrastructure/persistence/LabResultRepo     ← adapter (driven)

  LabResultService only depends on interfaces — it calls LabResultRepository (a port), not JPA directly. The JPA
  implementation is swapped in at runtime.

  ---
  Why Use It Over Other Patterns?

  vs. Layered / N-Tier (Controller → Service → Repository)

  The classic layered architecture is simpler, but the layers still leak. Your service layer ends up importing JPA
  annotations, Kafka types, or HTTP response objects. Change your database and you're touching business logic.

  Hexagonal enforces the boundary with dependency inversion — the domain defines the interface, infrastructure
  implements it. Not the other way around.

  vs. MVC

  MVC organizes around the UI concern (Model-View-Controller). It works fine for monoliths with a single UI, but doesn't
   scale well when you have multiple entry points (REST, Kafka, gRPC, tests) and multiple outputs (DB, cache, message
  bus).

  ---
  Practical Benefits

  ┌──────────────┬─────────────────────────────────────────────────────────────────────────────────────────────┐
  │   Benefit    │                                  What it means day-to-day                                   │
  ├──────────────┼─────────────────────────────────────────────────────────────────────────────────────────────┤
  │ Testability  │ Test business logic with no DB, no Kafka — just swap in a fake adapter                      │
  ├──────────────┼─────────────────────────────────────────────────────────────────────────────────────────────┤
  │ Swappability │ Switch from Kafka to RabbitMQ by writing one new adapter, zero domain changes               │
  ├──────────────┼─────────────────────────────────────────────────────────────────────────────────────────────┤
  │ Clarity      │ New developer can read application/ and understand what the system does without knowing how │
  ├──────────────┼─────────────────────────────────────────────────────────────────────────────────────────────┤
  │ Isolation    │ A bug in Kafka config doesn't contaminate your domain model                                 │
  └──────────────┴─────────────────────────────────────────────────────────────────────────────────────────────┘

  ---
  The Real Tradeoff

  Hexagonal adds more files and more indirection. For a simple CRUD app, it's overkill — you'll write an interface, an
  implementation, and a service just to do SELECT * FROM patients.

  It pays off when:
  - Multiple entry points (REST + Kafka + batch jobs)
  - You expect to swap infrastructure (e.g., migrate from SQL to NoSQL)
  - You want fast unit tests that don't spin up a DB

  MediTrack hits all three, which is why it's a reasonable fit here