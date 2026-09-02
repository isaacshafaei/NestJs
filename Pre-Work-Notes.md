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
### DDD — 3 concepts

* **Entity** → An object with a **unique identity** that stays the same even if its data changes.
  Example: `User(id=123)` — the user's name can change, but it's still the same user.

* **Value Object** → An object defined by its **value, not identity**. Usually immutable.
  Example: `Money(€50)` or `Address("Padova", "Italy")`.

* **Aggregate** → A **group of related entities/value objects** treated as one unit, with one main entity called the **Aggregate Root**.
  Example:

  ```text
  Order (Aggregate Root)
   ├── OrderItem
   ├── OrderItem
   └── ShippingAddress
  ```

**Easy way to remember:**

> **Entity = who/which one**
> **Value Object = what value**
> **Aggregate = group managed together**
---
### CQRS — very simple

**CQRS = Command Query Responsibility Segregation**

It means **separating operations that change data from operations that only read data**.

```text
Command → changes data
          Create / Update / Delete

Query   → reads data
          Get / Search
```

Example:

```text
CreateOrder()  → Command
GetOrder()     → Query
CancelOrder()  → Command
GetOrders()    → Query
```

### Why use CQRS?

> We use CQRS to **separate reading and writing logic**, making complex systems easier to scale, maintain, and optimize.

**Easy to remember:**

> **Command = Change**
> **Query = Read**
---
### Testing in DDD — very simple

In DDD, we mainly test the **business rules in the Domain layer**.

```text
Domain
  ↓
Unit Tests
  → Test business rules
```

For example:

```text
Order
 ├── addItem()
 ├── removeItem()
 └── calculateTotal()
```

You test things like:

* Can an order be created?
* Can you add an item?
* Is the total calculated correctly?
* Can you cancel an already completed order?

### Different types

* **Unit tests** → test Domain logic independently. **Most important in DDD.**
* **Integration tests** → test things like Repository + Database.
* **End-to-End (E2E)** → test the whole application from API → database.

**Easy way to remember:**

> **Unit → Domain rules**
> **Integration → Infrastructure**
> **E2E → Whole system**
---
### Architecture Tests

Architecture tests verify that your **code follows the architectural rules**.

For example, you might have this rule:

```text
Presentation → Application → Domain ← Infrastructure
```

An architecture test can ensure that:

```text
❌ Domain → Database
❌ Domain → NestJS Controller
❌ Domain → Infrastructure

✅ Application → Domain
✅ Infrastructure → Domain
```

So you're testing **how the code is structured**, not whether a business calculation is correct.

### Why is this important for a Junior?

When you join a company, you might understand the architecture, but after working on the project for a few months, it's easy to accidentally write:

```text
Controller
   ↓
Database
```

instead of following:

```text
Controller
   ↓
Application
   ↓
Domain
   ↓
Infrastructure
```

Architecture tests **automatically catch these violations**.

Think of it like:

> **Unit tests:** "Does my code work correctly?"
> **Architecture tests:** "Did I put my code in the right place?"

For a junior, this is very useful because the **architecture tests act like a guardrail** while you're learning the company's codebase and architectural conventions.
---
