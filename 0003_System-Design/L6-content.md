# 🚦 Rate Limiting: Complete Guide

A comprehensive guide to understanding rate limiting, error handling, and various algorithms used in production systems.

---

## 📋 Table of Contents

- [What is Rate Limiting?](#-what-is-rate-limiting)
- [Understanding Error 429](#-understanding-error-429)
- [Why Rate Limiting is Needed](#-why-rate-limiting-is-needed)
- [System Architecture](#-system-architecture)
- [Rate Limiting Algorithms](#-rate-limiting-algorithms)
  - [Token Bucket](#1-token-bucket-algorithm-)
  - [Leaky Bucket](#2-leaky-bucket-algorithm-)
  - [Sliding Window Log](#3-sliding-window-log-algorithm-)
  - [Fixed Window Counter](#4-fixed-window-counter-algorithm-)
  - [Sliding Window Counter](#5-sliding-window-counter-algorithm-)
- [Algorithm Comparison](#-algorithm-comparison)
- [Storage Solutions](#-storage-solutions)
- [Distributed Rate Limiting](#-distributed-rate-limiting)
- [Best Practices](#-best-practices)

---

## 🔍 What is Rate Limiting?

Rate limiting is a technique that controls how many requests a client can make within a specific time window.

**Example:**
```
Maximum: 100 requests per minute per user
```

When the limit is exceeded, the server returns:
```
HTTP 429 Too Many Requests
```

---

## ⚠️ Understanding Error 429

**429 Too Many Requests** means: *"You have sent too many requests in a given amount of time."*

### Response Headers

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1700000000
```

| Header | Description |
|--------|-------------|
| `Retry-After` | Seconds to wait before retrying |
| `X-RateLimit-Limit` | Maximum requests allowed |
| `X-RateLimit-Remaining` | Requests remaining in current window |
| `X-RateLimit-Reset` | Timestamp when the limit resets |

---

## 💡 Why Rate Limiting is Needed

| Benefit | Description |
|---------|-------------|
| 🛡️ **Prevent DDoS** | Protect against distributed denial-of-service attacks |
| 🤖 **Prevent Abuse** | Stop bots and malicious actors |
| 💾 **Protect Database** | Prevent database overload |
| ⚖️ **Fair Usage** | Ensure equitable access for all users |
| 💰 **Cost Control** | Manage infrastructure costs |

> ⚠️ **Without rate limiting:** One bad user → crashes entire system

---

## 🏗️ System Architecture

### High-Level Rate Limiting Flow

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     ▼
┌─────────────────────────────┐
│ API Gateway / Load Balancer │
└─────────────┬───────────────┘
              │
              ▼
┌──────────────────────────────┐
│   Rate Limiter Service       │
│   (Redis / In-Memory)        │
└──────┬──────────────┬────────┘
       │              │
       │ Allowed      │ Exceeded
       ▼              ▼
┌─────────────┐  ┌─────────┐
│   Backend   │  │ Return  │
│   Service   │  │   429   │
└─────────────┘  └─────────┘
```

### Request Dropping Logic

```python
if requests_in_window > threshold:
    DROP_REQUEST()  # Return 429
else:
    ALLOW_REQUEST()  # Forward to backend
```

**Example with Threshold = 5 req/sec:**

```
Request 1 ✅ Allowed
Request 2 ✅ Allowed
Request 3 ✅ Allowed
Request 4 ✅ Allowed
Request 5 ✅ Allowed
Request 6 ❌ Dropped (429)
Request 7 ❌ Dropped (429)
```

---

## 🔢 Rate Limiting Algorithms

### Comparison Overview

| Algorithm | Accuracy | Memory Usage | Burst Support | Complexity |
|-----------|----------|--------------|---------------|------------|
| Token Bucket | ⭐⭐⭐⭐ | Low | ✅ Yes | Medium |
| Leaky Bucket | ⭐⭐⭐ | Low | ❌ No | Low |
| Fixed Window Counter | ⭐⭐ | Very Low | ❌ No | Very Low |
| Sliding Window Log | ⭐⭐⭐⭐⭐ | High | ✅ Yes | High |
| Sliding Window Counter | ⭐⭐⭐⭐ | Medium | ⚠️ Partial | Medium |

---

## 1. Token Bucket Algorithm 🪣

### Concept

Imagine a bucket holding tokens that are added at a fixed rate. Each request consumes one token.

```
       Tokens added (5/sec)
             │
             ▼
       ┌──────────────┐
       │   TOKEN      │
       │   BUCKET     │  Max capacity: 10
       │  🪙🪙🪙🪙🪙  │
       └──────┬───────┘
              │
       Request needs 1 token
              │
         If empty → 429
```

### How It Works

**Configuration:**
- Bucket capacity: 10 tokens
- Refill rate: 5 tokens/second

**Scenario:**

```
Time 0s: Bucket has 10 tokens
User sends 12 requests instantly:
  - Requests 1-10: ✅ Allowed (tokens: 10 → 0)
  - Requests 11-12: ❌ Rejected (no tokens)

Time 1s: +5 tokens added (bucket now has 5 tokens)
User sends 3 requests:
  - All 3: ✅ Allowed (tokens: 5 → 2)
```

### Pseudocode

```python
def allow_request(bucket):
    if bucket.tokens > 0:
        bucket.tokens -= 1
        return True  # Allow
    else:
        return False  # Reject (429)

def refill_tokens(bucket, rate, time_elapsed):
    bucket.tokens = min(
        bucket.capacity,
        bucket.tokens + (rate * time_elapsed)
    )
```

### ✅ Pros

- Allows burst traffic
- Smooth traffic distribution
- Easy to implement
- Industry standard (used by AWS, Stripe, etc.)

### ❌ Cons

- Requires persistent storage
- Slightly more complex than fixed window

---

## 2. Leaky Bucket Algorithm 💧

### Concept

Requests enter a bucket and leak out at a constant rate. Overflow requests are dropped.

```
   Incoming Requests
   (variable rate)
         │
         ▼
   ┌─────────────┐
   │   BUCKET    │
   │  🌊🌊🌊🌊  │
   │             │───────► Process at constant rate
   └─────────────┘         (e.g., 2 req/sec)
         │
    Overflow ❌
```

### How It Works

**Configuration:**
- Bucket capacity: 10 requests
- Leak rate: 2 requests/second

**Scenario:**

```
20 requests arrive instantly:
  - Requests 1-10: Stored in bucket
  - Requests 11-20: ❌ Dropped (overflow)

Processing:
  - Every 0.5s: 1 request leaks out and is processed
```

### ✅ Pros

- Smooth, consistent output rate
- Simple to understand
- Protects downstream services

### ❌ Cons

- No burst handling
- Requests may experience delays
- Less flexible than token bucket

---

## 3. Sliding Window Log Algorithm 📝

### Concept

Store the timestamp of every request and count how many fall within the time window.

```
Time Window: Last 60 seconds
├────────────────────────────────────┤
                               NOW (12:00:50)

Request Timestamps:
  12:00:01 ✅
  12:00:05 ✅
  12:00:10 ✅
  12:00:40 ✅

New request at 12:00:50:
  Count = 4 (all within 60s window)
  If limit = 5 → ✅ Allow
```

### How It Works

```python
def allow_request(user_id, timestamp, limit, window_size):
    # Remove timestamps older than window
    logs = filter_logs(user_id, timestamp - window_size)
    
    if len(logs) < limit:
        add_log(user_id, timestamp)
        return True  # Allow
    else:
        return False  # Reject (429)
```

### ✅ Pros

- Very accurate
- No boundary issues
- Precise rate limiting

### ❌ Cons

- High memory usage (stores all timestamps)
- Expensive at scale
- Slower lookup times

---

## 4. Fixed Window Counter Algorithm 🪟

### Concept

Divide time into fixed windows and count requests per window.

```
Window 1           Window 2           Window 3
12:00:00          12:01:00           12:02:00
│                 │                  │
├─────────────────┼──────────────────┼────►
│   Count: 100    │   Count: 0       │
└─────────────────┴──────────────────┘
```

### The Boundary Problem ⚠️

```
11:59:59 → User sends 100 requests ✅
12:00:01 → User sends 100 requests ✅

Result: 200 requests in 2 seconds! 😱
(But limit was 100 per minute)
```

### Implementation

```python
def allow_request(user_id, window_key, limit):
    count = get_counter(f"{user_id}:{window_key}")
    
    if count < limit:
        increment_counter(f"{user_id}:{window_key}")
        return True
    else:
        return False
```

### ✅ Pros

- Very simple
- Low memory usage
- Fast lookups

### ❌ Cons

- Burst at window boundaries
- Inaccurate rate limiting
- Can allow 2× the limit

---

## 5. Sliding Window Counter Algorithm 🎯

### Concept

Hybrid approach combining fixed windows with weighted calculation.

```
Previous Window        Current Window
├──────────────────┼──────────────────┤
│                  │                  │
│  40% overlap     │  60% progress    │
│                  │                  │
└──────────────────┴──────────────────┘
     Count: 80          Count: 40
```

### Formula

```python
effective_count = current_count + (previous_count × overlap_percentage)

# Example:
overlap = 0.4  # 40% of previous window overlaps
effective_count = 40 + (80 × 0.4) = 40 + 32 = 72

if effective_count < limit:
    allow_request()
```

### ✅ Pros

- Near-perfect accuracy
- Lower memory than sliding log
- Smooth rate limiting

### ❌ Cons

- More complex calculations
- Slight estimation (not 100% accurate)

---

## 📊 Algorithm Comparison

### When to Use Each Algorithm

| Use Case | Recommended Algorithm | Reason |
|----------|----------------------|---------|
| 🌐 API Gateway | Token Bucket | Handles bursts, industry standard |
| 🔐 Login Attempts | Sliding Window Log | High accuracy for security |
| 🚀 Simple Microservice | Fixed Window | Low complexity, fast |
| 💳 Billing Systems | Sliding Window Counter | Balance accuracy and performance |
| 📹 Video Streaming | Leaky Bucket | Smooth, constant output |

---

## 💾 Storage Solutions

### Where Counters Are Stored

| Storage Type | Speed | Distribution | Use Case |
|--------------|-------|--------------|----------|
| **In-Memory (Local)** | ⚡ Very Fast | ❌ Single server only | Development, testing |
| **Redis** | ⚡ Fast | ✅ Distributed | Production (recommended) |
| **Memcached** | ⚡ Fast | ✅ Distributed | Production |
| **Database** | 🐢 Slow | ✅ Distributed | Not recommended for rate limiting |

---

## 🌍 Distributed Rate Limiting

### Real-World Implementation

Most production systems use **Redis + Lua Scripts** for distributed rate limiting.

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│ Server 1 │     │ Server 2 │     │ Server 3 │
└────┬─────┘     └────┬─────┘     └────┬─────┘
     │                │                │
     └────────────────┼────────────────┘
                      │
                      ▼
              ┌───────────────┐
              │     Redis     │
              │  (Centralized │
              │   Rate Limit  │
              │    Counter)   │
              └───────────────┘
```

### Why Redis + Lua?

✅ **Atomic operations** - No race conditions  
✅ **Fast** - In-memory performance  
✅ **Shared state** - All servers see same counters  
✅ **TTL support** - Automatic cleanup  

### Example Lua Script

```lua
-- Token Bucket in Redis
local key = KEYS[1]
local capacity = tonumber(ARGV[1])
local rate = tonumber(ARGV[2])
local now = tonumber(ARGV[3])

local bucket = redis.call('HMGET', key, 'tokens', 'last_refill')
local tokens = tonumber(bucket[1]) or capacity
local last_refill = tonumber(bucket[2]) or now

-- Refill tokens
local elapsed = now - last_refill
tokens = math.min(capacity, tokens + (elapsed * rate))

if tokens >= 1 then
    tokens = tokens - 1
    redis.call('HMSET', key, 'tokens', tokens, 'last_refill', now)
    redis.call('EXPIRE', key, 60)
    return 1  -- Allow
else
    return 0  -- Deny
end
```

---

## 🎯 Best Practices

### 1. Choose the Right Algorithm

```
Simple API → Token Bucket
Security-critical → Sliding Window Log
High traffic → Sliding Window Counter
```

### 2. Set Appropriate Limits

```
Read operations:  Higher limits (e.g., 1000/min)
Write operations: Lower limits (e.g., 100/min)
Login attempts:   Very low (e.g., 5/min)
```

### 3. Implement Graceful Degradation

```python
if rate_limited:
    return {
        "error": "Rate limit exceeded",
        "retry_after": 30,
        "limit": 100,
        "remaining": 0,
        "reset": 1700000000
    }
```

### 4. Use Multiple Layers

```
Layer 1: CDN/Edge rate limiting
Layer 2: API Gateway rate limiting
Layer 3: Application-level rate limiting
Layer 4: Database-level throttling
```

### 5. Monitor and Alert

Track metrics:
- 429 error rate
- Average requests per user
- Peak traffic patterns
- Rate limit violations

---

## 🎓 Interview Quick Reference

### Key Takeaway

> **"Most modern systems use Token Bucket or Sliding Window Counter implemented on Redis for distributed rate limiting."**

### Common Questions & Answers

**Q: What's the difference between Token Bucket and Leaky Bucket?**  
A: Token Bucket allows bursts (tokens accumulate), while Leaky Bucket enforces constant output rate (no bursts).

**Q: Why not use Fixed Window?**  
A: Boundary issue - can allow 2× the limit at window transitions.

**Q: How do you handle rate limiting across multiple servers?**  
A: Use centralized storage like Redis with atomic operations (Lua scripts).

**Q: What happens when Redis goes down?**  
A: Fail open (allow all requests) or fail closed (deny all) - depends on criticality. Use Redis Cluster for HA.

---

## 📚 Additional Resources

- [Redis Rate Limiting Patterns](https://redis.io/docs/manual/patterns/rate-limiting/)
- [NGINX Rate Limiting](https://www.nginx.com/blog/rate-limiting-nginx/)
- [AWS API Gateway Throttling](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-request-throttling.html)

---

