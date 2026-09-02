### Kafka — very short

* **Kafka** → a system for sending/storing messages between applications.
* **Producer** → sends a message to Kafka.
* **Consumer** → receives/reads a message.
* **Topic** → where messages are stored/grouped.
* **Broker** → Kafka server that stores the messages.
* **Consumer Group** → a group of consumers sharing the work.
* **Partition** → splits a topic to make Kafka faster and scalable.

### Why do we use Kafka?

> We use Kafka to **connect services without making them directly depend on each other**, while allowing messages/events to be stored, processed asynchronously, and handled at large scale.

### README note

```md
## Kafka

Apache Kafka is a distributed event streaming platform used to send, store, and process messages between services.

### Why Kafka?
- Decouples services
- Enables asynchronous communication
- Stores messages reliably
- Handles high volumes of events
- Scales using partitions and consumer groups

Example:

Order Service → Kafka → Payment Service
                    → Notification Service
                    → Analytics Service
```
---
Yes — **you are right**, but there are two different meanings of "layered architecture."

What I gave before was the **simpler/common 3–4 layer version** used in many backend projects:

```text
Controller → Service → Repository → Database
```

But in **DDD / Clean Architecture**, the layers are usually described as:

```text
Presentation
     ↓
Application
     ↓
Domain
     ↓
Infrastructure
```

### Very simply

* **Presentation** → HTTP/API, controllers, user input
* **Application** → coordinates use cases/business operations
* **Domain** → core business rules and entities
* **Infrastructure** → database, external APIs, Kafka, etc.

Example:

```text
POST /orders
     ↓
Presentation
     ↓
Application
     ↓
Domain
     ↓
Infrastructure → MySQL
```

**Important:** This structure is closer to **Clean Architecture / DDD**, not the basic traditional layered architecture I initially described.

For your **DDD + Hexagonal Architecture** learning, you should remember:

> **Presentation → Application → Domain ← Infrastructure**

The arrow toward Domain is important: **Domain should not depend on infrastructure.**
---

