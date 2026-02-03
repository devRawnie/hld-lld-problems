# URL Shortener — System Design (HLD)

## Interview Context
- Level: Mid-level Software Engineer
- Round: System Design / HLD
- Duration: 45–60 minutes
- Focus: Scalability, availability, trade-offs, clarity

---

<img width="1400" height="994" alt="image" src="https://github.com/user-attachments/assets/331efc04-2e18-402b-aa31-79f08deb939d" />

## 1. Problem Statement
Design a system that:
- Converts a long URL into a short URL
- Redirects the short URL to the original long URL
- Is highly available, low latency, and scalable

---

## 2. Functional Requirements
- Generate a short URL from a long URL
- Redirect short URL → long URL

### Optional (mention briefly if asked)
- Custom alias
- Expiry time
- Click analytics

---

## 3. Non‑Functional Requirements
- Low latency redirects (< 100 ms)
- High availability (reads >> writes)
- Horizontal scalability
- Availability preferred over strong consistency for redirects

---

## 4. High‑Level Architecture

### Components
- API Gateway
- URL Shortening Service
- Redirect Service
- Database (Key‑Value store)
- Cache (Redis)
- Optional: Analytics pipeline (async)

---

## 5. API Design

### Create Short URL

```
POST /shorten
{
"longUrl": "https://example.com/very/long/url"
}
→ https://sho.rt/aZ93k
```

### Redirect

```
GET /aZ93k
→ 302 Redirect → long URL
```

---

## 6. ID Generation (MOST IMPORTANT)

### Chosen Approach: Auto‑Increment ID + Base62 Encoding

#### Flow
1. DB generates an auto‑increment integer ID
2. Encode the ID using Base62
3. Store mapping: `short_id → long_url`

---

## 7. Why Auto‑Increment Integer ID Alone Is NOT Enough

### 1. Predictability (Security Issue)
Sequential IDs like:

```
sho.rt/12345
sho.rt/12346
```

Allow:
- Enumeration attacks
- Crawling of private URLs

**Interview line:**
> Sequential IDs allow enumeration attacks.

---

### 2. Leaks Internal Implementation
- DB IDs are internal details
- Short URLs are public API
- Tight coupling blocks future migrations

**Principle:**
> Public identifiers should not expose internal state.

---

### 3. Longer URLs (Base‑10 inefficiency)
- Decimal IDs are longer
- Base62 encodes same value in fewer characters

---

### 4. Multi‑Region Scaling Issues
- Auto‑increment IDs complicate global writes
- Coordination overhead increases

---

### Conclusion
> Auto‑increment IDs are fine internally, but must be encoded before exposing externally.

---

## 8. Why Base62 and NOT Base64

### Base62 Character Set
[a‑z][A‑Z][0‑9] = 62 characters


---

### Problems with Base64

#### 1. URL‑Unsafe Characters
Base64 includes:
/ =

These require URL encoding → longer, uglier URLs.

---

#### 2. Padding Characters (`=`)
- No value in short URLs
- Adds noise and complexity

---

#### 3. Wrong Use‑Case
- Base64 is for binary data
- URL shorteners deal with numeric IDs

---

### Why Base62 Is Ideal
- URL‑safe
- No encoding required
- Compact
- Easy to implement and explain

**One‑liner:**
> Base62 gives maximum compactness without URL‑unsafe characters.

---

## 9. Database Design

### Schema
short_id (PK)
long_url
created_at
expiry (optional)


### Database Choice
- Key‑Value Store (DynamoDB / Cassandra)
- Access pattern: point lookups
- High read throughput

---

## 10. Caching Strategy

### Why Cache?
- Redirects are the hot path
- Majority traffic is reads

### Cache Design
- Redis: `short_id → long_url`
- Read‑through cache
- TTL optional

### Gotcha
> URLs are immutable → cache invalidation is easy.

---

## 11. Redirect Flow
1. User hits short URL
2. Check Redis cache
3. If hit → 302 redirect
4. If miss → DB → populate cache → redirect

---

## 12. Scaling Strategy

### Read Scaling
- Cache absorbs most traffic
- Stateless services → horizontal scaling

### Write Scaling
- ID generation service
- DB partitioned by `short_id`

---

## 13. Failure Scenarios & Handling

### Cache Down
- Fallback to DB
- Higher latency, system still works

### DB Down
- Redirects fail
- Use replication and multi‑AZ setup

### Hot Keys (Viral URLs)
- Redis handles hot keys well
- Optional CDN in front of redirect service

---

## 14. Preventing Malicious URLs

### Threats
- Phishing
- Malware
- Spam campaigns
- Open redirect abuse

---

### Layered Defense Strategy

#### 1. Input Validation
- Allow only `http` / `https`
- Reject `javascript:`, `data:`
- Block internal IPs (SSRF)

---

#### 2. Rate Limiting (MOST IMPORTANT)
- Apply on `/shorten`
- Per IP / per user

**Interview line:**
> Rate limiting eliminates most abuse.

---

#### 3. Reputation / Safe‑Browsing Check
- Google Safe Browsing or internal denylist
- Async check (don’t block user flow)

---

#### 4. Warning Page (Optional)
“You are leaving sho.rt → example.com”


---

#### 5. Abuse Monitoring
- Detect click spikes
- Geo anomalies
- Auto‑disable suspicious links

---

## 15. Analytics (Optional Extension)
- Async logging via queue
- Never block redirect path
- Track:
  - Click count
  - Timestamp
  - Country (optional)

---

## 16. Common Interview Follow‑Up Questions

### Why not UUID?
- Too long (36 chars)
- Poor UX
- Worse cache locality

---

### How many URLs can Base62 support?
- 62⁷ ≈ 3.5 trillion

---

### Same long URL shortened twice?
- Allow multiple short URLs
- Optional deduplication via hash

---

### Why 302 and not 301?
- Avoid browser caching
- Enable analytics
- 301 can be enabled later

---

### What if cache goes down?
- Fallback to DB
- Increased latency, system survives

---

### How to scale to 10B URLs?
- Sharded KV store
- Aggressive caching
- Async cleanup
- Multi‑region reads

---

## 17. Final Interview Summary (30 seconds)
> We generate short URLs using auto‑increment IDs encoded with Base62 to keep them compact and URL‑safe. We store mappings in a key‑value store, cache redirects in Redis for low latency, and scale horizontally. Abuse is mitigated using rate limiting, validation, and async reputation checks while keeping the redirect path fast.

