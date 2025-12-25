
## Problem Statement

Users want to monitor cryptocurrency prices and receive alerts when prices cross
user-defined thresholds, without manually checking dashboards or charts.

## Goals

- Provide near real-time price monitoring for selected cryptocurrencies
- Allow users to define customizable alert rules (e.g., price > X, price < Y)
- Deliver alerts reliably via supported channels (email, webhook)
- Support gradual scaling from a small user base to a large one

## Non-Goals

- High-frequency trading or sub-second price updates
- Financial advice or investment recommendations
- Complex charting or analytics dashboards

## Assumptions

- Price data is obtained from a third-party public API
- Users tolerate up to 1-minute alert latency
- Alerts are best-effort and may be retried on failure
- Initial deployment targets a single geographic region

## Functional Requirements

- Users can create, update, and delete alert rules
- Each alert rule specifies:
  - Cryptocurrency
  - Threshold value
  - Comparison operator (>, <)
- The system periodically fetches latest prices
- Alerts are triggered when rule conditions are met
- Users receive notifications through configured channels

## Non-Functional Requirements

- Availability: ≥ 99.9%
- Alert latency: ≤ 60 seconds
- System should scale horizontally
- Observability: logging, metrics, and alerting must be supported
- Secure handling of user data and credentials

## Success Metrics

- Alert delivery success rate
- Average alert latency
- System uptime
- Number of active users and alert rules

## High-Level Architecture

The system is composed of the following core components:

- API Gateway
- User Management Service
- Alert Management Service
- Price Ingestion Service
- Alert Evaluation Service
- Notification Service
- Persistent Storage
- Message Queue

## Component Responsibilities

### API Gateway
- Entry point for all client requests
- Handles authentication, authorization, and rate limiting

### User Management Service
- Manages user accounts and preferences
- Stores notification channels (email, webhook)

### Alert Management Service
- CRUD operations for alert rules
- Persists alert configurations

### Price Ingestion Service
- Periodically fetches price data from third-party APIs
- Normalizes and publishes price updates

### Alert Evaluation Service
- Evaluates incoming prices against stored alert rules
- Emits alert events when conditions are met

### Notification Service
- Sends alerts via email or webhook
- Retries on transient failures

### Persistent Storage
- Stores users, alert rules, and alert history

### Message Queue
- Decouples ingestion, evaluation, and notification
- Buffers spikes in traffic

## Data Flow

1. User creates alert rule via API Gateway
2. Alert Management Service stores rule in database
3. Price Ingestion Service fetches latest prices on a schedule
4. Price updates are published to a message queue
5. Alert Evaluation Service consumes price events
6. Matching alert rules trigger alert events
7. Notification Service delivers alerts to users
8. Alert delivery results are recorded for auditing

## Data Storage

- Relational Database
  - Users
  - Alert Rules
  - Notification Preferences

- Time-Series or Key-Value Store
  - Latest prices
  - Alert history (optional)

## Design Rationale

- Message queues are used to decouple services and improve scalability
- Price ingestion is separated from alert evaluation to allow independent scaling
- Stateless services enable horizontal scaling
- Persistent storage ensures durability and recovery

## Scaling Strategy

### Horizontal Scaling
- All stateless services scale horizontally behind load balancers
- Auto-scaling based on CPU usage and queue depth

### Price Ingestion Scaling
- Shard ingestion jobs by asset or market
- Use distributed schedulers for large asset sets

### Alert Evaluation Scaling
- Partition alert rules by asset or hash of rule ID
- Consumers scale independently based on queue backlog

## Reliability & Fault Tolerance

- Message queues buffer traffic during downstream failures
- Retry mechanisms with exponential backoff for transient errors
- Dead-letter queues capture failed alert events
- Idempotent alert processing prevents duplicate notifications
- Health checks enable fast failover and recovery

## Failure Modes & Mitigations

| Failure | Impact | Mitigation |
|------|------|-----------|
| Price API outage | No fresh data | Use cached prices and retry |
| Message queue backlog | Alert delays | Auto-scale consumers |
| Notification provider failure | Alerts not delivered | Retry and fallback channels |
| Database overload | Rule access delays | Read replicas and caching |


## Consistency & Delivery Guarantees

- Alert delivery is at-least-once
- Deduplication handled via alert IDs
- Eventual consistency between ingestion and notification

## Trade-offs

- Real-time streaming increases cost; scheduled polling reduces complexity
- Caching reduces API calls but may increase alert latency
- Strong consistency is sacrificed for availability and scalability

## Execution Plan

### Phase 1 — MVP (4–6 weeks)
- Define APIs and data models
- Implement price ingestion and alert evaluation
- Support single notification channel (email)
- Basic monitoring and logging

### Phase 2 — Scalability & Reliability
- Introduce message queues
- Add retry and dead-letter handling
- Horizontal scaling and auto-scaling
- Expand notification channels

### Phase 3 — Operational Excellence
- Alerting and dashboards
- Load testing and capacity planning
- Security hardening
- Documentation and runbooks

## Risks & Mitigation

| Risk | Impact | Mitigation |
|-----|--------|------------|
| Third-party API instability | Data gaps | Multi-provider fallback |
| Alert spam | User churn | Rate limits and deduplication |
| Cost overruns | Budget issues | Usage caps and monitoring |
| Team dependency delays | Missed deadlines | Clear ownership and milestones |

## Success Metrics

### Reliability
- Alert success rate (%)
- Mean alert latency

### Scale
- Active users
- Alerts processed per minute

### Quality
- Duplicate alert rate
- Failed notification rate

### Business
- User retention
- Alert engagement rate

## Ownership & Responsibility

- Engineering: system implementation and reliability
- Product: feature prioritization and UX
- TPM: cross-team coordination, timeline tracking, risk management
- Operations: monitoring, on-call, incident response









