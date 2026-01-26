# 🔹 1. Event Sourcing

## ✅ What is Event Sourcing?

Event Sourcing is a design pattern where:

> Instead of storing the **current state** of an entity, we store **all changes as a sequence of events**.

Example:

Instead of storing:

```
Balance = 500
```

We store:

```
AccountCreated
MoneyDeposited(1000)
MoneyWithdrawn(500)
```

Current balance is calculated by replaying events.

---

### 🧠 Why Use Event Sourcing?

* Full history of changes
* Auditability
* Easy debugging
* Time travel
* Rebuild state anytime

---

### 🧱 Architecture

```
Command → Validate → Create Event → Store Event → Publish Event
                                      ↓
                               Event Store
                                      ↓
                               Projector → State View
```

---

## 🎯 Interview Questions

### Q1. What is Event Sourcing?

**Answer:**
Event Sourcing is a pattern where state changes are stored as immutable events instead of storing only the latest state. The current state is reconstructed by replaying events.

---

### Q2. How is Event Sourcing different from CRUD?

| CRUD             | Event Sourcing    |
| ---------------- | ----------------- |
| Store latest row | Store all changes |
| Data overwritten | Data append-only  |
| Limited history  | Full history      |

---

### Q3. Advantages?

* Complete audit log
* Easy debugging
* Supports replay
* Enables temporal queries

---

### Q4. Disadvantages?

* Storage grows
* Complex queries
* Harder modeling
* Versioning events

---

# 🔹 2. Event Stream

## ✅ What is Event Stream?

A continuous, ordered sequence of events over time.

Example:

```
E1 → E2 → E3 → E4 → ...
```

In Kafka:

```
Topic = Event Stream
```

---

### Characteristics

* Append-only
* Ordered per partition
* Immutable

---

## 🎯 Interview Questions

### Q1. What is an event stream?

A continuously produced sequence of events representing changes in a system.

---

### Q2. Difference between event stream and message queue?

| Event Stream       | Queue                 |
| ------------------ | --------------------- |
| Retained           | Removed after consume |
| Replayable         | Not replayable        |
| Multiple consumers | Usually single        |

---

# 🔹 3. State

## ✅ What is State?

State is the **current condition** of an entity.

Example:

```
User:
name = A
email = a@gmail.com
status = ACTIVE
```

---

In Event Sourcing:

```
State = Reduce(Events)
```

Like:

```
state = fold(events)
```

---

## 🎯 Interview Questions

### Q1. How is state obtained in event sourcing?

By replaying all events in order.

---

### Q2. What is derived state?

State that is computed from events instead of stored directly.

---

# 🔹 4. Hydration (State Reconstruction)

## ✅ What is Hydration?

Hydration means:

> Rebuilding an object’s state by replaying its events.

Example:

```
events = [Created, NameChanged, EmailChanged]
user = hydrate(events)
```

---

### Why Needed?

* Service restart
* Cache loss
* Migration
* Debugging

---

## 🎯 Interview Questions

### Q1. What is hydration?

Process of reconstructing entity state by replaying stored events.

---

### Q2. When is hydration required?

* On startup
* After crash
* For debugging
* For projections

---

# 🔹 5. Replay

## ✅ What is Replay?

Reprocessing events again from event store or Kafka.

---

### Why Replay?

* Build new projections
* Fix bug
* Create new view
* Recover data

---

Example:

```
Replay from offset 0
```

---

## 🎯 Interview Questions

### Q1. Difference between hydration and replay?

| Hydration        | Replay                |
| ---------------- | --------------------- |
| Build one entity | Process many events   |
| Local            | System-wide           |
| State rebuild    | Reprocessing pipeline |

---

# 🔹 6. Audit Trail

## ✅ What is Audit Trail?

Complete history of:

* Who did what
* When
* What changed

Event sourcing naturally provides audit trail.

---

Example Event:

```
OrderShipped {
 orderId,
 userId,
 timestamp
}
```

---

## 🎯 Interview Questions

### Q1. How does event sourcing help auditing?

Every change is stored as event → nothing lost.

---

# 🔹 7. Time Machine (Temporal Query)

## ✅ What is Time Machine?

Ability to see state **at any point in time**.

Example:

```
State at 10:00 AM
State at 5:00 PM
```

Achieved by replaying events up to timestamp.

---

## 🎯 Interview Questions

### Q1. How to implement time travel?

Replay events until given time.

---

# 🔹 8. Kafka Topics

## ✅ Topic

A named stream of events.

Example:

```
user-events
order-events
```

---

---

# 🔹 9. Kafka Partitions

## ✅ Partition

Topic is split into partitions.

```
Topic
  ├─ Partition 0
  ├─ Partition 1
  └─ Partition 2
```

Each partition is ordered.

---

### Why Partitions?

* Parallelism
* Scalability
* High throughput

---

## 🎯 Interview Questions

### Q1. Are messages ordered in Kafka?

Yes, within a partition.

---

### Q2. How is message routed to partition?

* Key hash
* Round robin
* Custom partitioner

---

# 🔹 10. Kafka Consumer Groups

## ✅ Consumer Group

A group of consumers working together to consume a topic.

Rules:

* One partition → One consumer inside group
* Same group = load balancing
* Different group = independent read

---

Example:

```
Topic: 4 partitions
Group A: 4 consumers → each gets one partition
```

---

## 🎯 Interview Questions

### Q1. What happens if consumers > partitions?

Extra consumers stay idle.

---

### Q2. Why consumer groups?

Scalability and fault tolerance.

---

# 🔹 11. Kafka Offsets

Each consumer tracks:

```
Partition + Offset
```

Offset = position in stream.

---

---

# 🔹 12. Mapping Everything Together

```
Kafka Topic → Event Stream
Kafka Event → Domain Event
Event Store → Kafka / DB
Hydration → Replaying entity events
Replay → Reprocessing stream
Audit Trail → Stored events
Time Machine → Replay until time
State → Fold(events)
```

---

# 🔹 Advanced Interview Questions

### Q1. How do you version events?

* Add new fields
* Never remove fields
* Schema registry

---

### Q2. How do you handle schema evolution?

Backward compatible changes.

---

### Q3. How do you improve hydration performance?

Use **Snapshots**

```
Snapshot + New Events
```

---

### Q4. What is Snapshot?

Periodic saved state so you don’t replay from beginning.

---

---

# 🔹 Mini Example (Pseudo)

```python
state = initial_state
for event in events:
    state = apply(state, event)
```

---

# 🔹 Real-World Use Cases

* Banking
* E-commerce
* Logistics
* IoT
* Financial trading
* Microservices


