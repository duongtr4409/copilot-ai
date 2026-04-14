---
name: performance-profiling-analysis
description: "Profile and optimize application performance: API response time analysis, DB query profiling with EXPLAIN plans, memory usage patterns, connection pool tuning, and load test result interpretation. Use for performance bottleneck detection, optimization recommendations, and baseline establishment."
argument-hint: "Target endpoints/queries, performance metrics, environment, load profile, and optimization constraints"
user-invocable: true
---

# Performance Profiling and Analysis (Phan tich va Toi uu Hieu nang)

## What This Skill Produces
- Performance bottleneck identification with evidence and root cause analysis.
- DB query optimization recommendations with EXPLAIN plan evidence.
- API response time analysis with P50/P95/P99 breakdown.
- Memory and resource usage pattern analysis.
- Actionable optimization recommendations prioritized by impact.
- Performance baseline documentation for regression tracking.

## When To Use
- API response times exceed SLA thresholds.
- Database queries are slow or consuming excessive resources.
- Memory usage is growing abnormally (potential leak).
- Load test results need interpretation and action planning.
- Establishing performance baseline before major release.
- Code review raises performance concerns.
- Production monitoring shows degradation patterns.

## Required Inputs
- Target scope: specific endpoints, queries, services, or modules.
- Current metrics: response times, query execution times, memory usage.
- Environment context: hardware, DB engine, connection pool config, JVM settings.
- Load profile: expected concurrent users, requests per second, data volume.
- SLA targets: latency, throughput, error rate thresholds.

## Standard Workflow

### Step 1: Define Performance Context
1. Identify the performance concern (latency, throughput, memory, CPU, I/O).
2. Collect current metrics and establish measurement baseline.
3. Define target metrics (what "good" looks like).
4. Identify the measurement environment and tooling.

### Step 2: API Response Time Profiling
1. Break down total response time into components:
   - Network latency
   - Controller/middleware processing
   - Service layer business logic
   - Database query execution
   - External service calls
   - Serialization/deserialization
2. Identify the dominant time contributor.
3. Measure P50, P95, P99 for each endpoint.

#### API Profiling Report Format
```
## Endpoint: GET /api/users
Total Response Time (P95): 450ms

### Time Breakdown
| Component         | Time   | % of Total |
|-------------------|--------|------------|
| Auth Middleware    | 15ms   | 3%         |
| Controller Logic  | 5ms    | 1%         |
| Service Logic     | 30ms   | 7%         |
| DB Query (main)   | 350ms  | 78%        |
| DB Query (count)  | 40ms   | 9%         |
| Serialization     | 10ms   | 2%         |

### Bottleneck: DB Query (main) — 78% of response time
### Root Cause: Missing index on user.department_id + full table scan
### Recommendation: Add composite index idx_users_department_status
```

### Step 3: Database Query Profiling
1. Identify slow queries (execution time > 100ms at P95).
2. Run `EXPLAIN ANALYZE` (PostgreSQL) or `EXPLAIN` (MySQL) for each slow query.
3. Analyze execution plan for:
   - Sequential scans (table scan) on large tables.
   - Missing or unused indexes.
   - Inefficient join strategies (nested loop on large sets).
   - Sort operations without index support.
   - High row estimates with low actual rows (stats outdated).
4. Propose index additions or query restructuring.

#### Query Analysis Template
```
## Query: Q-001
SQL: SELECT u.*, d.name FROM users u JOIN departments d ON u.dept_id = d.id
     WHERE u.status = 'ACTIVE' AND d.region = 'APAC'
     ORDER BY u.created_at DESC LIMIT 20

Current Execution Time (P95): 380ms
Rows Scanned: 50,000
Rows Returned: 20

### EXPLAIN ANALYZE Output
<paste explain output>

### Analysis
- Problem: Sequential scan on users table (no index on status + dept_id).
- Problem: Sort operation on created_at without index support.
- Estimated cardinality mismatch: planner expects 5,000 rows, actual 50,000.

### Recommendations
1. CREATE INDEX idx_users_status_dept_created ON users(status, dept_id, created_at DESC);
2. Run ANALYZE users; to update table statistics.
3. Consider partial index: CREATE INDEX idx_users_active ON users(dept_id, created_at DESC) WHERE status = 'ACTIVE';

### Expected Improvement
- Index scan replaces seq scan: ~380ms → ~15ms
- Sort eliminated by index ordering
```

### Step 4: N+1 Query Detection
1. Enable SQL logging in development environment.
2. Execute target API endpoint and capture all SQL queries.
3. Count queries per request and identify N+1 patterns.
4. Propose solutions:

| Pattern | Solution | Example |
|---|---|---|
| N+1 lazy load | `JOIN FETCH` or `@EntityGraph` | `@EntityGraph(attributePaths = {"orders"})` |
| N+1 on collection | `@BatchSize` annotation | `@BatchSize(size = 50)` |
| Multiple queries for same data | DTO projection | `SELECT new UserDTO(u.id, u.name)` |
| Over-fetching | Select only needed columns | `findByIdProjectedBy(...)` |

### Step 5: Memory and Resource Analysis
1. Identify memory usage patterns:
   - Heap usage trend (growing? stable? cyclic?)
   - GC frequency and pause times
   - Large object allocation hotspots
2. Check for common memory issues:
   - Unbounded caches without TTL or max size
   - Large collection in-memory processing (should be streamed or paginated)
   - Connection/resource leaks (DB connections, HTTP clients, file handles)
   - Static collection accumulation
3. JVM tuning recommendations:
   - Heap size: `-Xms` and `-Xmx` based on workload
   - GC algorithm: G1GC (default), ZGC (low latency), Shenandoah
   - Metaspace: adjust if reflection/proxy-heavy

### Step 6: Connection Pool Analysis
1. Check current pool configuration:
   - Pool size (min/max/idle)
   - Connection timeout
   - Idle timeout
   - Max lifetime
   - Leak detection threshold
2. Analyze pool metrics:
   - Active connections vs pool size
   - Wait time for connection acquisition
   - Connection creation rate
   - Timeout/failure rate
3. Tuning guidelines:
   - Pool size formula: `connections = (core_count * 2) + effective_spindle_count`
   - Typical starting point: 10-20 connections for web apps
   - Set `leak-detection-threshold` to catch unreturned connections

### Step 7: Load Test Result Interpretation
1. Parse load test output (JMeter, k6, Gatling).
2. Identify:
   - Throughput ceiling (max sustained RPS without error increase)
   - Latency inflection point (where P95 starts spiking)
   - Error onset point (load level where errors begin)
   - Resource saturation point (CPU/memory/connection exhaustion)
3. Compare against SLA targets.
4. Recommend capacity plan.

#### Load Test Summary Template
```
## Load Test Summary
- Tool: k6
- Duration: 10 min ramp-up + 5 min sustained
- Max Virtual Users: 200
- Target: GET /api/orders

### Results
| Metric      | Value  | Target | Status |
|-------------|--------|--------|--------|
| RPS (avg)   | 450    | 500    | ⚠️     |
| P50 Latency | 85ms   | 100ms  | ✅     |
| P95 Latency | 320ms  | 300ms  | ⚠️     |
| P99 Latency | 890ms  | 500ms  | ❌     |
| Error Rate  | 0.3%   | <0.1%  | ❌     |
| CPU Peak    | 78%    | <80%   | ✅     |
| Memory Peak | 82%    | <85%   | ⚠️     |

### Bottleneck Analysis
- P99 latency spike at >150 VUs → DB connection pool exhaustion
- Error rate increase at >180 VUs → timeout on connection wait
- Recommendation: Increase pool size from 10 to 20, add read replica for queries
```

## Decision Logic And Branching

1. If bottleneck is in DB query layer (>50% of response time):
   - Focus on query profiling, indexing, and N+1 detection.
   - Apply Step 3 and Step 4 thoroughly.

2. If bottleneck is in application logic:
   - Profile code execution paths.
   - Look for synchronous blocking, inefficient algorithms, unnecessary processing.

3. If bottleneck is in external service calls:
   - Add circuit breaker, timeout, and retry with backoff.
   - Consider async processing, caching, or event-driven patterns.

4. If memory is the concern:
   - Focus on Step 5.
   - Enable heap dump analysis for production-like load.

5. If performance is already good but baseline is needed:
   - Focus on Step 7 to establish documented baseline.
   - Set up automated performance regression detection in CI.

## Output Contract
When invoked, return these sections in order:

1. Performance Context and SLA Targets
2. Current Metrics Summary
3. Bottleneck Analysis (with evidence)
4. DB Query Profiling Results
5. N+1 Detection Results
6. Memory and Resource Analysis
7. Optimization Recommendations (prioritized)
8. Expected Impact After Optimization
9. Performance Baseline Documentation
10. Risks and Follow-up Actions

## Completion Criteria
- Every identified bottleneck has measurable evidence (not guesswork).
- DB recommendations include EXPLAIN plan analysis.
- Optimization priorities are ranked by impact and effort.
- Performance baselines are documented for regression tracking.
- Recommendations are actionable within normal sprint capacity.

## Example Invocations
- /performance-profiling-analysis Profile GET /api/orders endpoint — P95 is 800ms, target is 200ms.
- /performance-profiling-analysis Analyze slow queries in the order module and propose index strategy.
- /performance-profiling-analysis Interpret k6 load test results and identify capacity limits.
- /performance-profiling-analysis Check for N+1 queries in user-management module.
- /performance-profiling-analysis Establish performance baseline for payment service before v2.0 release.
