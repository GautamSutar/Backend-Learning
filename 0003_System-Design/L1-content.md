# Backend Architecture Guide

A comprehensive guide covering fundamental concepts of backend system architecture, from basic connectivity to advanced scaling and performance optimization.

---

## Table of Contents

1. [The Foundation: Connectivity](#1-the-foundation-connectivity)
2. [Scaling: Handling Growth](#2-scaling-handling-growth)
3. [API & Traffic Management](#3-api--traffic-management)
4. [Asynchronous Messaging](#4-asynchronous-messaging-the-email-scenario)
5. [Database Performance](#5-database-performance)
6. [Complete System Flow](#complete-system-flow)

---

## 1. The Foundation: Connectivity

### Client & Server

**Client:** The "requester" component that initiates communication
- Web browsers (Chrome, Safari)
- Mobile applications
- Desktop applications

**Server:** The "responder" component that processes requests
- Computer or cluster of computers
- Listens for requests
- Processes business logic
- Accesses databases
- Sends responses back to clients

### IP Address (Public vs. Private)

**Public IP:**
- Like your home address
- Unique across the entire internet
- Required for servers that need direct public access (e.g., web servers)

**Private IP:**
- Like a room number inside a hotel
- Only unique within a specific network (office network, AWS VPC)
- Used for internal server communication (e.g., App Server to Database)
- Provides security by preventing direct external access

### DNS (Domain Name System)

**The Concept:** DNS is the phonebook of the internet

**Purpose:** Translates human-readable domain names to IP addresses
- Computers communicate using IP addresses (e.g., `192.0.2.1`)
- Humans prefer memorable names (e.g., `google.com`)

**How it works:**
1. User types `google.com` in browser
2. Browser queries DNS resolver: "What is the IP for google.com?"
3. DNS server responds with the IP address
4. Browser connects using the IP address

---

## 2. Scaling: Handling Growth

When traffic increases, servers can crash without proper scaling strategies.

### Vertical Scaling (Scaling Up)

Adding more power to your **existing** machine.

**Upgrades include:**
- CPU enhancement
- Additional RAM
- Increased storage

**Pros:**
- Simple to implement
- No code changes required

**Cons:**
- **Hard Limit:** Maximum capacity constraints per machine
- **Downtime:** Server must be shut down for upgrades (service interruption)
- **Single Point of Failure:** If the machine crashes, entire service goes down

### Horizontal Scaling (Scaling Out)

Adding **more** machines instead of making one machine stronger.

**Pros:**
- Infinite scalability (add servers as needed)
- No downtime (servers can be updated individually)

**Cons:**
- More complex management
- Requires a Load Balancer

### Load Balancer (ELB)

In Amazon AWS: **Elastic Load Balancer (ELB)**

**Role:**
- Stands in front of horizontal server fleet
- Receives client requests
- Acts as traffic cop, distributing requests evenly across servers

**Health Checks:**
- Continuously monitors server health
- Stops routing traffic to failed servers
- Resumes traffic when servers recover

---

## 3. API & Traffic Management

### API Gateway

While Load Balancers handle traffic distribution, API Gateways handle traffic **management**.

**Key Functions:**

**Routing:**
- Directs requests to appropriate services
- Example: `/payment` → Payment Service, `/user` → User Service

**Authentication:**
- Validates user login status before requests reach servers
- Ideal location for Auth Service integration

**AWS Integration:**
- Register with **Route 53** (AWS DNS)
- Maps domain (e.g., `api.yourdomain.com`) to Gateway

### Rate Limiting

Protects servers from being overwhelmed by excessive requests from single users or bots (DDoS attacks).

**Algorithms:**

**1. Token Bucket:**
- Bucket receives tokens at constant rate (e.g., 5 tokens/second)
- Each request requires one token
- Full bucket: tokens overflow and are discarded
- Empty bucket: user must wait
- **Allows traffic bursts**

**2. Leaky Bucket:**
- Requests enter bucket
- Requests leak out at constant rate
- Full bucket: new requests are discarded
- **Enforces smooth, constant traffic flow (no bursts)**

---

## 4. Asynchronous Messaging (The Email Scenario)

**The Problem:** Sending 1 million emails after payment processing

**Bad Approach:**
- Payment Service sends emails directly
- Slow performance
- Payment fails if email server is down

**Good Approach:** Decoupling using Queues

### SQS (Simple Queue Service) - One-to-One

**Workflow:**
1. User completes payment
2. Payment Service pushes message to SQS ("Send Email to User X")
3. Payment Service immediately responds "Success" to user
4. Separate **Worker Service** picks up message from SQS
5. Worker sends email in background

**Polling Mechanisms (Cost vs. Speed):**

**Short Polling:**
- Worker asks SQS: "Any messages?"
- SQS replies immediately (even if queue is empty)
- **Downside:** Wastes money and CPU with repeated empty requests

**Long Polling:**
- Worker asks SQS: "Any messages?"
- If queue is empty, SQS **waits** (e.g., 20 seconds) before replying "No"
- **Upside:** Reduces requests, saving money and network resources

### SNS (Simple Notification Service) - Pub/Sub

SQS is point-to-point. SNS is a **megaphone**.

**Pub/Sub Model:**
- **Publisher** sends message to **Topic**
- Multiple **Subscribers** listen to that topic

**Use Case:**
- Send receipt to client
- Notify vendor
- Log transaction for analytics
- All from single event

### Fanout Architecture (SNS + SQS)

The gold standard for complex systems.

**The Setup:**
- Send **one** message to SNS Topic

**The Fanout:**
- SNS Topic automatically pushes message copies to multiple SQS queues:
  - Queue A (Email Worker) receives copy
  - Queue B (Vendor Notification) receives copy
  - Queue C (Analytics) receives copy

**Benefit:**
- All processes run in parallel
- Slow Email worker doesn't affect Analytics worker performance

### Dead Letter Queue (DLQ)

Handles persistent message processing failures.

**The Problem:**
- Worker attempts to process message and fails (e.g., email provider down)
- Worker returns message to queue
- Continuous failures create infinite loop (Poison Pill)

**The Solution:**
- After defined failure threshold (e.g., 3 attempts)
- Queue moves message to **Dead Letter Queue**
- Engineers inspect DLQ to debug failures
- Main system continues operating without blockage

---

## 5. Database Performance

### Primary vs. Read Replicas

**Primary Node (Master):**
- Source of truth
- Handles all **Write** operations (Insert, Update, Delete)

**Read Replicas (Slaves):**
- Copies of Primary Node
- Handle **Read-only** operations

**Strategy:**
- Most applications have significantly more reads than writes
- Example: View Instagram photos 100x more than posting them
- Route all "View" requests to Replicas
- Reduces load on Primary Node

### Caching

Database reads from disk are slow. RAM reads are fast.

**Technology:** Redis or Memcached

**Strategy:**
1. App queries Cache: "Do you have User X profile?"
2. **Cache Hit:** Data found! Return immediately (Microseconds)
3. **Cache Miss:** Data not found
   - App queries Database (Milliseconds)
   - Serves data to user
   - Writes data to Cache for future requests

---

## Complete System Flow

### End-to-End Request Flow

1. **Client** resolves DNS and calls **API Gateway**
2. Gateway checks **Authentication** and routes to **ELB**
3. **ELB** balances load across **Web Servers**
4. Server checks **Cache (Redis)**
   - If empty, reads from **Read Replica**
5. Server processes payment (Writes to **Primary DB**)
6. Server publishes event to **SNS**
7. **SNS** fans out to multiple **SQS** queues
8. **Workers** poll SQS using Long Polling and send emails
9. If email fails repeatedly, message moves to **DLQ**

---

## Key Takeaways

- **Scaling:** Choose between Vertical (simpler, limited) and Horizontal (complex, infinite) based on needs
- **Decoupling:** Use message queues (SQS/SNS) to separate concerns and improve reliability
- **Performance:** Implement caching and read replicas to optimize database operations
- **Reliability:** Use Dead Letter Queues to handle failures gracefully
- **Security:** Use private IPs for internal communication and API Gateways for authentication

---

## Technologies Referenced

- **AWS Services:** ELB, SQS, SNS, Route 53
- **Caching:** Redis, Memcached
- **Databases:** Primary/Replica architecture
- **Load Balancing:** Elastic Load Balancer (ELB)
- **DNS:** Domain Name System, Route 53

---

*This guide provides a foundational understanding of backend architecture principles applicable across various cloud platforms and technology stacks.*