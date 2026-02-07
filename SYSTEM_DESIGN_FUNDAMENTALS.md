# System Design Fundamentals Wiki

A comprehensive guide to system design concepts, architectural patterns, and technology comparisons to help you build scalable systems.

---

## Table of Contents

### 📚 Foundation: Core Infrastructure (Start Here)
1. [Horizontal vs Vertical Scaling](#horizontal-vs-vertical-scaling) - *Understand how systems grow*
2. [Stateful vs Stateless Services](#stateful-vs-stateless-services) - *Critical for scaling decisions*
3. [Object Storage vs Block Storage vs File Storage](#object-storage-vs-block-storage-vs-file-storage) - *Where to store your data*
4. [SQL vs NoSQL Databases](#sql-vs-nosql-databases) - *Choosing the right database*
5. [Database Indexing Strategies](#database-indexing-strategies) - *Making databases fast*
6. [Database Replication vs Sharding](#database-replication-vs-sharding) - *Scaling databases*

### 🏗️ Architecture Patterns
7. [Monolithic vs Microservices Architecture](#monolithic-vs-microservices-architecture) - *Application structure*
8. [Load Balancer vs API Gateway](#load-balancer-vs-api-gateway) - *Traffic management*
9. [CDN vs Edge Computing](#cdn-vs-edge-computing) - *Global content delivery*

### 🔌 Communication Patterns
10. [REST vs GraphQL vs gRPC](#rest-vs-graphql-vs-grpc) - *API protocols*
11. [Synchronous vs Asynchronous Communication](#synchronous-vs-asynchronous-communication) - *Request patterns*
12. [Push vs Pull Architecture](#push-vs-pull-architecture) - *Real-time vs polling*
13. [Message Queues vs Event Streaming](#message-queues-vs-event-streaming) - *Async messaging*

### ⚡ Performance & Reliability
14. [Caching Strategies](#caching-strategies) - *Speed up your system*
15. [Rate Limiting and Throttling](#rate-limiting-and-throttling) - *Protect your APIs*
16. [Circuit Breaker vs Retry vs Timeout](#circuit-breaker-vs-retry-vs-timeout) - *Resilience patterns*

### 🔐 Security & Access
17. [Authentication vs Authorization](#authentication-vs-authorization) - *Identity and permissions*

### 🌐 Distributed Systems
18. [Consistency Patterns (CAP Theorem)](#consistency-patterns-cap-theorem) - *Trade-offs in distributed systems*
19. [Service Discovery Patterns](#service-discovery-patterns) - *Finding services dynamically*

### 🚀 Deployment & Operations
20. [Blue-Green vs Canary vs Rolling Deployment](#blue-green-vs-canary-vs-rolling-deployment) - *Safe deployments*

---

## 🎯 Recommended Learning Path

This guide is organized into **6 progressive phases**. Follow this sequence for the best learning experience:

### Phase 1: Foundation (Start Here) 
**Goal**: Understand core infrastructure concepts that everything else builds upon.

1. **[Horizontal vs Vertical Scaling](#horizontal-vs-vertical-scaling)** - Learn how systems grow and scale
2. **[Stateful vs Stateless Services](#stateful-vs-stateless-services)** - Critical concept affecting all architecture decisions
3. **[Object Storage vs Block Storage vs File Storage](#object-storage-vs-block-storage-vs-file-storage)** - Understand where and how to store data
4. **[SQL vs NoSQL Databases](#sql-vs-nosql-databases)** - Choose the right database for your use case
5. **[Database Indexing Strategies](#database-indexing-strategies)** - Make your databases performant
6. **[Database Replication vs Sharding](#database-replication-vs-sharding)** - Scale your databases effectively

**Why this order?** These are the building blocks. You can't design a system without understanding how it will scale, where data lives, and how to make it fast.

### Phase 2: Architecture Patterns
**Goal**: Learn how to structure applications and manage traffic.

7. **[Monolithic vs Microservices Architecture](#monolithic-vs-microservices-architecture)** - Understand application architecture evolution
8. **[Load Balancer vs API Gateway](#load-balancer-vs-api-gateway)** - Manage and distribute traffic effectively
9. **[CDN vs Edge Computing](#cdn-vs-edge-computing)** - Deliver content globally with low latency

**Why this order?** Once you understand data and scaling, you need to know how to structure your application and route traffic to it.

### Phase 3: Communication Patterns
**Goal**: Master how services communicate with each other and with clients.

10. **[REST vs GraphQL vs gRPC](#rest-vs-graphql-vs-grpc)** - Choose the right API protocol
11. **[Synchronous vs Asynchronous Communication](#synchronous-vs-asynchronous-communication)** - Understand request/response patterns
12. **[Push vs Pull Architecture](#push-vs-pull-architecture)** - Real-time updates vs polling
13. **[Message Queues vs Event Streaming](#message-queues-vs-event-streaming)** - Async messaging at scale

**Why this order?** With your architecture in place, you need to know how components talk to each other efficiently.

### Phase 4: Performance & Reliability
**Goal**: Make your system fast and resilient.

14. **[Caching Strategies](#caching-strategies)** - Speed up your system dramatically
15. **[Rate Limiting and Throttling](#rate-limiting-and-throttling)** - Protect your APIs from abuse and overload
16. **[Circuit Breaker vs Retry vs Timeout](#circuit-breaker-vs-retry-vs-timeout)** - Handle failures gracefully

**Why this order?** Now that services can communicate, make them fast and resilient to failures.

### Phase 5: Security & Access
**Goal**: Secure your system and control access.

17. **[Authentication vs Authorization](#authentication-vs-authorization)** - Verify identity and manage permissions

**Why this order?** Security should be understood before diving into complex distributed systems.

### Phase 6: Advanced Distributed Systems
**Goal**: Master the complexities of distributed architectures.

18. **[Consistency Patterns (CAP Theorem)](#consistency-patterns-cap-theorem)** - Understand fundamental trade-offs
19. **[Service Discovery Patterns](#service-discovery-patterns)** - Services finding each other dynamically

**Why this order?** These concepts require understanding of all previous topics. CAP theorem ties together databases, scaling, and architecture decisions.

### Phase 7: Deployment & Operations
**Goal**: Deploy safely and operate at scale.

20. **[Blue-Green vs Canary vs Rolling Deployment](#blue-green-vs-canary-vs-rolling-deployment)** - Deploy without downtime or risk

**Why this order?** Once you've designed and built your system, you need to deploy it safely.

---

## 💡 How to Use This Guide

- **Beginners**: Follow the phases in order, 1-2 topics per day
- **Interview Prep**: Focus on Phases 1, 2, 3, and 6 (most commonly asked)
- **Experienced Engineers**: Jump to specific topics as reference material
- **Quick Review**: Read the comparison tables and "Best For" sections in each topic

---

## Load Balancer vs API Gateway


### Load Balancer
**Purpose**: Distributes incoming network traffic across multiple servers to ensure no single server bears too much load.

**Key Features**:
- Traffic distribution (Round Robin, Least Connections, IP Hash)
- Health checks and automatic failover
- SSL termination
- Session persistence (sticky sessions)

**Best For**:
- Distributing traffic across identical backend servers
- High availability and fault tolerance
- Simple traffic routing based on network-level metrics

**Scaling Benefits**:
- Enables horizontal scaling by adding more servers
- Prevents server overload and improves response times
- Provides redundancy and fault tolerance

**Example Use Cases**:
- Web application with multiple identical server instances
- Database read replicas distribution
- Microservices with multiple instances of the same service

### API Gateway
**Purpose**: Acts as a single entry point for all client requests, providing routing, composition, and protocol translation.

**Key Features**:
- Request routing and composition
- Authentication and authorization
- Rate limiting and throttling
- Request/response transformation
- Protocol translation (REST to gRPC, etc.)
- API versioning
- Analytics and monitoring

**Best For**:
- Microservices architecture with multiple backend services
- Complex routing logic based on request content
- Centralized security and authentication
- API management and versioning

**Scaling Benefits**:
- Reduces client complexity by providing a unified interface
- Enables independent scaling of backend services
- Centralizes cross-cutting concerns (auth, logging, rate limiting)
- Facilitates service decomposition and evolution

**Example Use Cases**:
- Microservices architecture with 10+ services
- Mobile/web apps needing a simplified API interface
- Multi-tenant SaaS applications
- Systems requiring request aggregation from multiple services

### When to Use Both
Many modern architectures use **both**:
- **API Gateway** handles application-level routing, authentication, and business logic
- **Load Balancer** distributes traffic to multiple API Gateway instances and backend services

---

## SQL vs NoSQL Databases

### SQL (Relational Databases)
**Examples**: PostgreSQL, MySQL, Oracle, SQL Server

**Key Characteristics**:
- Structured schema with tables, rows, and columns
- ACID transactions (Atomicity, Consistency, Isolation, Durability)
- Strong consistency
- Complex queries with JOINs
- Vertical scaling (traditionally)

**Best For**:
- Structured data with clear relationships
- Complex queries and reporting
- Financial transactions requiring ACID guarantees
- Applications needing strong consistency
- Data integrity is critical

**Scaling Strategies**:
- **Vertical Scaling**: Upgrade hardware (CPU, RAM, SSD)
- **Read Replicas**: Distribute read traffic
- **Sharding**: Partition data across multiple databases
- **Connection Pooling**: Reuse database connections

**Example Use Cases**:
- Banking and financial systems
- E-commerce order management
- CRM and ERP systems
- Inventory management
- User authentication systems

### NoSQL Databases

#### Document Stores (MongoDB, Couchbase)
**Key Characteristics**:
- Flexible schema with JSON-like documents
- Horizontal scaling built-in
- Eventual consistency (configurable)
- Fast reads and writes

**Best For**:
- Rapidly evolving schemas
- Hierarchical data structures
- Content management systems
- User profiles and preferences

**Scaling Benefits**:
- Easy horizontal scaling through sharding
- No complex JOINs means better performance at scale
- Flexible schema allows rapid iteration

#### Key-Value Stores (Redis, DynamoDB)
**Key Characteristics**:
- Simple key-value pairs
- Extremely fast reads/writes
- In-memory options for ultra-low latency
- Limited query capabilities

**Best For**:
- Caching
- Session management
- Real-time analytics
- Leaderboards and counters

**Scaling Benefits**:
- Linear scalability
- Sub-millisecond latency
- Simple data model enables massive throughput

#### Column-Family Stores (Cassandra, HBase)
**Key Characteristics**:
- Optimized for write-heavy workloads
- Distributed and highly available
- Eventual consistency
- Time-series data support

**Best For**:
- Time-series data (IoT, metrics, logs)
- Write-heavy applications
- Large-scale analytics
- Event logging

**Scaling Benefits**:
- Masterless architecture (no single point of failure)
- Linear scalability by adding nodes
- Handles petabytes of data

#### Graph Databases (Neo4j, Amazon Neptune)
**Key Characteristics**:
- Optimized for relationship queries
- Native graph storage and processing
- Traversal-based queries

**Best For**:
- Social networks
- Recommendation engines
- Fraud detection
- Knowledge graphs

**Scaling Benefits**:
- Efficient relationship queries at scale
- Better performance than SQL JOINs for connected data

### SQL vs NoSQL Decision Matrix

| Factor | SQL | NoSQL |
|--------|-----|-------|
| **Data Structure** | Fixed schema, relational | Flexible schema, various models |
| **Transactions** | ACID guaranteed | Eventual consistency (mostly) |
| **Scaling** | Vertical (primary) | Horizontal (native) |
| **Query Complexity** | Complex JOINs supported | Limited cross-document queries |
| **Consistency** | Strong | Eventual (tunable) |
| **Use Case** | Structured, transactional | Unstructured, high-throughput |

---

## Caching Strategies

### Cache-Aside (Lazy Loading)
**How it Works**:
1. Application checks cache first
2. If miss, fetch from database
3. Store result in cache
4. Return data

**Best For**:
- Read-heavy workloads
- Data that doesn't change frequently
- Unpredictable access patterns

**Scaling Benefits**:
- Reduces database load significantly
- Only caches data that's actually requested

**Example**: User profile data, product catalogs

### Write-Through Cache
**How it Works**:
1. Application writes to cache
2. Cache writes to database synchronously
3. Confirm write completion

**Best For**:
- Data consistency is critical
- Read-heavy with occasional writes
- Acceptable write latency

**Scaling Benefits**:
- Cache always has fresh data
- Reduces read load on database

**Example**: Configuration data, user settings

### Write-Behind (Write-Back) Cache
**How it Works**:
1. Application writes to cache
2. Cache acknowledges immediately
3. Cache writes to database asynchronously

**Best For**:
- Write-heavy workloads
- Can tolerate eventual consistency
- Need low write latency

**Scaling Benefits**:
- Extremely fast writes
- Can batch database writes
- Reduces database write load

**Example**: Analytics events, logging, metrics

### Cache Invalidation Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **TTL (Time-to-Live)** | Cache expires after fixed time | News feeds, weather data |
| **Event-Based** | Invalidate on specific events | User updates profile |
| **LRU (Least Recently Used)** | Remove least accessed items | Limited cache memory |
| **Write Invalidation** | Clear cache on write | Ensure consistency |

### Popular Caching Technologies

- **Redis**: In-memory, supports complex data structures, pub/sub
- **Memcached**: Simple, fast, distributed memory caching
- **CDN**: Caches static assets geographically close to users
- **Application-Level**: In-process caching (Caffeine, Guava)

---

## Message Queues vs Event Streaming

### Message Queues
**Examples**: RabbitMQ, Amazon SQS, Azure Service Bus

**Key Characteristics**:
- Point-to-point or publish-subscribe
- Messages consumed and removed
- Guarantees delivery order (FIFO queues)
- Supports acknowledgments and retries

**Best For**:
- Task distribution and background jobs
- Decoupling microservices
- Request/response patterns
- Guaranteed message delivery

**Scaling Benefits**:
- Asynchronous processing improves responsiveness
- Multiple consumers can process in parallel
- Handles traffic spikes by queuing requests
- Enables independent service scaling

**Example Use Cases**:
- Email sending service
- Image processing pipeline
- Order processing system
- Payment processing

### Event Streaming
**Examples**: Apache Kafka, Amazon Kinesis, Apache Pulsar

**Key Characteristics**:
- Append-only log of events
- Events retained for configured period
- Multiple consumers can read same events
- High throughput and low latency
- Event replay capability

**Best For**:
- Real-time analytics and monitoring
- Event sourcing architectures
- Log aggregation
- Stream processing
- Multiple consumers need same data

**Scaling Benefits**:
- Handles millions of events per second
- Horizontal scaling through partitions
- Decouples producers from consumers
- Enables real-time data pipelines

**Example Use Cases**:
- User activity tracking
- IoT sensor data processing
- Real-time fraud detection
- Microservices event-driven architecture
- Change Data Capture (CDC)

### Comparison Table

| Feature | Message Queue | Event Streaming |
|---------|---------------|-----------------|
| **Message Retention** | Deleted after consumption | Retained for configured period |
| **Consumers** | Typically one consumer per message | Multiple consumers can read same event |
| **Throughput** | Moderate | Very high |
| **Use Case** | Task processing | Real-time analytics, event sourcing |
| **Replay** | Not supported | Supported |
| **Ordering** | Queue-level | Partition-level |

---

## Horizontal vs Vertical Scaling

### Vertical Scaling (Scale Up)
**Definition**: Adding more resources (CPU, RAM, disk) to a single server.

**Advantages**:
- Simpler to implement (no code changes)
- No distributed system complexity
- Maintains data consistency easily
- Lower software licensing costs

**Disadvantages**:
- Hardware limits (can't scale infinitely)
- Single point of failure
- Downtime required for upgrades
- Expensive at high-end hardware

**Best For**:
- Monolithic applications
- Databases requiring strong consistency
- Applications not designed for distribution
- Early-stage applications with predictable growth

**Example**: Upgrading from 16GB to 64GB RAM, 4 cores to 16 cores

### Horizontal Scaling (Scale Out)
**Definition**: Adding more servers/instances to distribute the load.

**Advantages**:
- Nearly unlimited scaling potential
- Better fault tolerance and redundancy
- Cost-effective (use commodity hardware)
- No downtime for scaling
- Geographic distribution possible

**Disadvantages**:
- Application must be designed for distribution
- Data consistency challenges
- Increased operational complexity
- Network latency between nodes

**Best For**:
- Microservices architectures
- Web applications with variable traffic
- Stateless services
- Cloud-native applications

**Scaling Benefits**:
- Handle millions of concurrent users
- Automatic scaling based on demand
- Regional/global distribution
- Pay only for what you use (cloud)

**Example**: Running 10 identical web server instances behind a load balancer

### Hybrid Approach
Most modern systems use **both**:
- **Vertical scaling** for databases and stateful services
- **Horizontal scaling** for stateless application servers and microservices

---

## Monolithic vs Microservices Architecture

### Monolithic Architecture
**Definition**: Single unified application where all components are interconnected and interdependent.

**Advantages**:
- Simpler to develop initially
- Easier to test (single deployment unit)
- Simpler deployment process
- Better performance (no network calls)
- Easier debugging and monitoring

**Disadvantages**:
- Difficult to scale specific components
- Long deployment cycles
- Technology stack lock-in
- Codebase becomes complex over time
- Single point of failure

**Best For**:
- Small teams (< 10 developers)
- Simple applications with limited scope
- Startups validating product-market fit
- Applications with tightly coupled business logic

**Example**: Traditional e-commerce site with product catalog, cart, checkout, and user management in one application

### Microservices Architecture
**Definition**: Application composed of small, independent services that communicate over network protocols.

**Advantages**:
- Independent scaling of services
- Technology diversity (polyglot)
- Faster deployment cycles
- Better fault isolation
- Team autonomy
- Easier to understand individual services

**Disadvantages**:
- Increased operational complexity
- Network latency and reliability issues
- Distributed system challenges (consistency, transactions)
- More difficult to test end-to-end
- Requires DevOps maturity

**Best For**:
- Large, complex applications
- Multiple teams working independently
- Different scaling requirements per service
- Need for technology diversity
- Frequent deployments

**Scaling Benefits**:
- Scale only the services that need it
- Independent deployment reduces risk
- Better resource utilization
- Enables continuous delivery

**Example**: Netflix, Amazon, Uber - each business capability (user service, payment service, notification service) is a separate microservice

### Migration Path
**Start Monolithic → Extract Microservices Gradually**:
1. Begin with monolith for speed
2. Identify bounded contexts
3. Extract high-value services first (frequently changing, different scaling needs)
4. Use strangler fig pattern for gradual migration

---

## Synchronous vs Asynchronous Communication

### Synchronous Communication
**Definition**: Caller waits for the response before continuing.

**Protocols**: HTTP/REST, gRPC, GraphQL

**Advantages**:
- Simpler to understand and debug
- Immediate feedback
- Easier error handling
- Natural request-response flow

**Disadvantages**:
- Tight coupling between services
- Cascading failures
- Reduced availability (all services must be up)
- Poor performance under high load

**Best For**:
- User-facing operations requiring immediate response
- Simple CRUD operations
- Operations requiring immediate confirmation
- Low-latency requirements

**Example**: User login, fetching product details, real-time search

### Asynchronous Communication
**Definition**: Caller doesn't wait for response; continues processing.

**Protocols**: Message queues (RabbitMQ, SQS), Event streaming (Kafka), Webhooks

**Advantages**:
- Loose coupling between services
- Better fault tolerance
- Handles traffic spikes gracefully
- Improved system resilience
- Better resource utilization

**Disadvantages**:
- More complex to implement
- Eventual consistency
- Harder to debug and trace
- Requires message broker infrastructure

**Best For**:
- Background processing
- Long-running operations
- Event-driven architectures
- Operations that don't need immediate response

**Scaling Benefits**:
- Services can scale independently
- Natural backpressure mechanism
- Prevents cascading failures
- Enables retry and dead-letter queues

**Example**: Email notifications, order processing, image resizing, analytics events

### Hybrid Approach
**Best Practice**: Use synchronous for user-facing operations, asynchronous for background tasks
- User submits order → Synchronous response (order created)
- Order processing, inventory update, email notification → Asynchronous

---

## CDN vs Edge Computing

### CDN (Content Delivery Network)
**Examples**: Cloudflare, Akamai, Amazon CloudFront, Fastly

**Definition**: Geographically distributed network of servers that cache static content close to users.

**Key Features**:
- Caches static assets (images, CSS, JS, videos)
- Reduces latency by serving from nearby locations
- Offloads traffic from origin servers
- DDoS protection and security

**Best For**:
- Static content delivery
- Media streaming
- Global user base
- High-traffic websites

**Scaling Benefits**:
- Reduces origin server load by 60-90%
- Improves page load times globally
- Handles traffic spikes automatically
- Reduces bandwidth costs

**Example Use Cases**:
- News websites with global audience
- Video streaming platforms
- E-commerce product images
- Software downloads

### Edge Computing
**Examples**: Cloudflare Workers, AWS Lambda@Edge, Fastly Compute@Edge

**Definition**: Running compute workloads at edge locations close to users, not just caching.

**Key Features**:
- Execute code at edge locations
- Dynamic content generation
- Personalization at the edge
- API request processing
- A/B testing and feature flags

**Best For**:
- Dynamic content requiring low latency
- Personalization based on location
- Authentication and authorization
- Request routing and transformation
- IoT data processing

**Scaling Benefits**:
- Ultra-low latency (< 50ms globally)
- Reduces load on central servers
- Enables global applications without regional infrastructure
- Processes data closer to source (IoT)

**Example Use Cases**:
- Personalized content delivery
- Edge authentication
- Bot detection and security
- Real-time bidding (ad tech)
- IoT data aggregation

### Comparison

| Feature | CDN | Edge Computing |
|---------|-----|----------------|
| **Primary Function** | Cache static content | Execute dynamic code |
| **Content Type** | Static files | Dynamic, personalized |
| **Latency** | Low (cached content) | Ultra-low (computed nearby) |
| **Use Case** | Asset delivery | Application logic |
| **Complexity** | Low | Moderate to High |

---

## Database Replication vs Sharding

### Database Replication
**Definition**: Copying data from one database (primary) to one or more databases (replicas).

#### Types of Replication

**1. Master-Slave (Primary-Replica)**
- One primary handles writes
- Multiple replicas handle reads
- Asynchronous or synchronous replication

**2. Master-Master (Multi-Primary)**
- Multiple primaries accept writes
- Bi-directional replication
- Conflict resolution required

**Advantages**:
- Improved read performance
- High availability and failover
- Geographic distribution
- Backup and disaster recovery

**Disadvantages**:
- Replication lag (eventual consistency)
- Write scaling limited to primary
- Storage cost (duplicate data)
- Complexity in conflict resolution

**Best For**:
- Read-heavy workloads (90% reads, 10% writes)
- High availability requirements
- Geographic distribution
- Reporting and analytics

**Scaling Benefits**:
- Distribute read traffic across replicas
- Reduce load on primary database
- Enable maintenance without downtime

**Example**: E-commerce product catalog (many reads, few writes)

### Database Sharding
**Definition**: Partitioning data across multiple databases, each holding a subset of the data.

#### Sharding Strategies

**1. Horizontal Sharding (Range-Based)**
- Partition by key range (user_id 1-1000 → Shard 1)
- Simple but can create hotspots

**2. Hash-Based Sharding**
- Use hash function on key (hash(user_id) % num_shards)
- Better distribution but harder to add shards

**3. Geographic Sharding**
- Partition by location (US users → US shard)
- Reduces latency for regional users

**4. Directory-Based Sharding**
- Lookup table maps keys to shards
- Flexible but adds lookup overhead

**Advantages**:
- Scales writes horizontally
- Reduces query load per database
- Can use commodity hardware
- Improves query performance (smaller datasets)

**Disadvantages**:
- Application complexity (shard routing)
- Cross-shard queries are difficult
- Rebalancing shards is complex
- No ACID transactions across shards

**Best For**:
- Write-heavy workloads
- Very large datasets (> 1TB)
- Multi-tenant applications
- Need to scale beyond single database limits

**Scaling Benefits**:
- Linear write scaling by adding shards
- Each shard handles subset of data
- Enables massive datasets (petabytes)

**Example**: Social media platform (shard by user_id), SaaS application (shard by tenant_id)

### Replication + Sharding
**Best Practice**: Combine both for maximum scalability
- **Shard** to distribute writes and large datasets
- **Replicate** each shard for read scaling and high availability

**Example**: Instagram
- Sharded by user_id for write scaling
- Each shard has replicas for read scaling and failover

---

## REST vs GraphQL vs gRPC

### REST (Representational State Transfer)
**Protocol**: HTTP/HTTPS with JSON (typically)

**Key Characteristics**:
- Resource-based URLs (`/users/123`)
- Standard HTTP methods (GET, POST, PUT, DELETE)
- Stateless communication
- Well-understood and widely adopted

**Advantages**:
- Simple and easy to understand
- Great tooling and ecosystem
- Cacheable (HTTP caching)
- Language and platform agnostic
- Browser-friendly

**Disadvantages**:
- Over-fetching (getting more data than needed)
- Under-fetching (multiple requests needed)
- Versioning challenges
- No built-in type system

**Best For**:
- Public APIs
- CRUD operations
- Simple client-server communication
- Browser-based applications
- Microservices communication (simple cases)

**Scaling Benefits**:
- HTTP caching reduces server load
- Stateless design enables horizontal scaling
- Load balancing is straightforward

**Example**: `GET /api/users/123/posts?limit=10`

### GraphQL
**Protocol**: HTTP/HTTPS with GraphQL query language

**Key Characteristics**:
- Single endpoint (`/graphql`)
- Client specifies exact data needed
- Strongly typed schema
- Introspection and documentation

**Advantages**:
- No over-fetching or under-fetching
- Single request for complex data
- Strong typing and validation
- Excellent developer experience
- Real-time subscriptions

**Disadvantages**:
- More complex than REST
- Caching is harder
- Query complexity can cause performance issues
- Learning curve for clients
- Potential for expensive queries

**Best For**:
- Mobile applications (reduce data transfer)
- Complex, nested data requirements
- Rapid frontend iteration
- Multiple client types (web, mobile, desktop)
- Real-time features

**Scaling Benefits**:
- Reduces number of requests
- Clients fetch only needed data
- Reduces bandwidth usage
- Backend can optimize specific queries

**Example**:
```graphql
query {
  user(id: 123) {
    name
    posts(limit: 10) {
      title
      comments { author }
    }
  }
}
```

### gRPC
**Protocol**: HTTP/2 with Protocol Buffers (binary)

**Key Characteristics**:
- Binary protocol (Protocol Buffers)
- Strongly typed contracts (.proto files)
- Bi-directional streaming
- Code generation for multiple languages
- HTTP/2 multiplexing

**Advantages**:
- Extremely fast (binary serialization)
- Efficient network usage
- Streaming support (client, server, bi-directional)
- Strong typing and code generation
- Built-in authentication and load balancing

**Disadvantages**:
- Not browser-friendly (requires gRPC-Web)
- Binary format harder to debug
- Less human-readable
- Smaller ecosystem than REST
- Requires HTTP/2

**Best For**:
- Microservices communication (internal)
- Real-time streaming
- Low-latency requirements
- Polyglot environments
- Mobile clients (efficient bandwidth)

**Scaling Benefits**:
- 7-10x faster than REST (binary serialization)
- HTTP/2 multiplexing reduces connections
- Efficient for high-throughput systems
- Streaming reduces overhead

**Example Use Cases**:
- Microservices mesh (service-to-service)
- Real-time chat applications
- IoT device communication
- Mobile apps with limited bandwidth

### Comparison Table

| Feature | REST | GraphQL | gRPC |
|---------|------|---------|------|
| **Protocol** | HTTP/1.1 | HTTP/1.1 | HTTP/2 |
| **Format** | JSON (typically) | JSON | Protocol Buffers (binary) |
| **Performance** | Moderate | Moderate | Very High |
| **Browser Support** | Excellent | Excellent | Limited (needs gRPC-Web) |
| **Caching** | Easy (HTTP caching) | Complex | Complex |
| **Streaming** | No (SSE separate) | Yes (subscriptions) | Yes (native) |
| **Typing** | No | Yes (schema) | Yes (.proto) |
| **Best For** | Public APIs, CRUD | Complex clients, mobile | Microservices, real-time |

### When to Use What

**Use REST when**:
- Building public APIs
- Simple CRUD operations
- Need HTTP caching
- Browser is primary client

**Use GraphQL when**:
- Complex, nested data requirements
- Multiple client types with different needs
- Mobile apps (reduce over-fetching)
- Rapid frontend development

**Use gRPC when**:
- Internal microservices communication
- Performance is critical
- Need streaming capabilities
- Polyglot environment with code generation

---

## Consistency Patterns (CAP Theorem)

### CAP Theorem
**Definition**: In a distributed system, you can only guarantee 2 out of 3: Consistency, Availability, Partition Tolerance.

- **Consistency (C)**: All nodes see the same data at the same time
- **Availability (A)**: Every request receives a response (success or failure)
- **Partition Tolerance (P)**: System continues to operate despite network partitions

**Reality**: Network partitions will happen, so you must choose between **CP** or **AP**.

### CP Systems (Consistency + Partition Tolerance)
**Behavior**: Sacrifice availability during partitions to maintain consistency.

**Examples**: 
- MongoDB (with majority write concern)
- HBase
- Redis (with wait command)
- Zookeeper
- Etcd

**Best For**:
- Financial transactions
- Inventory management
- Booking systems
- Any system where stale data is unacceptable

**Scaling Trade-off**:
- Strong consistency limits availability
- May reject writes during network issues
- Slower writes (must wait for acknowledgment)

**Example Use Case**: Bank account balance - can't show inconsistent balance across ATMs

### AP Systems (Availability + Partition Tolerance)
**Behavior**: Sacrifice consistency during partitions to maintain availability.

**Examples**:
- Cassandra
- DynamoDB
- Riak
- CouchDB
- DNS

**Best For**:
- Social media feeds
- Analytics and metrics
- Shopping carts
- User preferences
- Systems where eventual consistency is acceptable

**Scaling Benefits**:
- Always available for reads and writes
- Better performance (no coordination overhead)
- Easier to scale globally

**Example Use Case**: Twitter timeline - okay if you see a tweet a few seconds later

### Consistency Models

| Model | Description | Use Case |
|-------|-------------|----------|
| **Strong Consistency** | All reads see latest write | Banking, inventory |
| **Eventual Consistency** | All nodes converge eventually | Social media, DNS |
| **Causal Consistency** | Related operations seen in order | Chat messages |
| **Read-Your-Writes** | User sees their own writes immediately | User profile updates |
| **Monotonic Reads** | Successive reads don't go backward in time | Session data |

### PACELC Theorem (Extended CAP)
**If Partition (P)**: Choose between Availability (A) and Consistency (C)  
**Else (E)**: Choose between Latency (L) and Consistency (C)

Even without partitions, there's a trade-off between consistency and latency.

**Examples**:
- **PA/EL**: Cassandra, DynamoDB (prioritize availability and low latency)
- **PC/EC**: HBase, MongoDB (prioritize consistency, accept higher latency)
- **PA/EC**: Cosmos DB (configurable)

---

## Rate Limiting and Throttling

### Why Rate Limiting?
- Prevent abuse and DDoS attacks
- Ensure fair resource usage
- Protect backend services from overload
- Monetization (tier-based limits)
- Cost control (API usage)

### Rate Limiting Algorithms

#### 1. Token Bucket
**How it Works**:
- Bucket holds tokens (refilled at fixed rate)
- Each request consumes a token
- Request rejected if no tokens available

**Advantages**:
- Allows bursts (up to bucket size)
- Smooth traffic over time
- Memory efficient

**Best For**:
- APIs allowing occasional bursts
- General-purpose rate limiting

**Example**: AWS API Gateway, Stripe API

#### 2. Leaky Bucket
**How it Works**:
- Requests added to queue (bucket)
- Processed at fixed rate
- Overflow requests rejected

**Advantages**:
- Smooths out bursts
- Predictable output rate
- Prevents sudden spikes

**Best For**:
- Network traffic shaping
- Ensuring consistent processing rate

**Example**: Network routers, traffic shaping

#### 3. Fixed Window Counter
**How it Works**:
- Count requests in fixed time windows (e.g., per minute)
- Reset counter at window boundary
- Reject if limit exceeded

**Advantages**:
- Simple to implement
- Low memory usage

**Disadvantages**:
- Burst at window boundaries (2x limit possible)

**Best For**:
- Simple rate limiting
- Low-traffic APIs

#### 4. Sliding Window Log
**How it Works**:
- Store timestamp of each request
- Count requests in sliding time window
- Remove old timestamps

**Advantages**:
- Accurate rate limiting
- No boundary burst issues

**Disadvantages**:
- High memory usage (stores all timestamps)

**Best For**:
- Strict rate limiting requirements
- Premium APIs

#### 5. Sliding Window Counter
**How it Works**:
- Hybrid of fixed window and sliding window
- Weighted count from current and previous window

**Advantages**:
- More accurate than fixed window
- Less memory than sliding log
- Prevents boundary bursts

**Best For**:
- Production systems (good balance)
- High-traffic APIs

**Example**: Cloudflare, Redis rate limiting

### Rate Limiting Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **User-based** | Limit per user/API key | SaaS APIs, user quotas |
| **IP-based** | Limit per IP address | Public endpoints, DDoS prevention |
| **Global** | Limit across entire system | Protect backend capacity |
| **Endpoint-based** | Different limits per endpoint | Expensive operations get lower limits |
| **Tiered** | Different limits per subscription tier | Freemium models |

### Where to Implement

**1. API Gateway** (Recommended)
- Centralized rate limiting
- Before reaching backend services
- Examples: Kong, AWS API Gateway, Nginx

**2. Application Level**
- Fine-grained control
- Business logic aware
- Examples: Express middleware, Django decorators

**3. Infrastructure Level**
- DDoS protection
- Examples: Cloudflare, AWS WAF

### Scaling Considerations

**Distributed Rate Limiting**:
- Use Redis or Memcached for shared state
- Atomic operations (INCR, EXPIRE)
- Consider race conditions

**Example Redis Implementation**:
```
MULTI
INCR user:123:requests
EXPIRE user:123:requests 60
EXEC
```

---

## Authentication vs Authorization

### Authentication
**Definition**: Verifying **who you are** (identity verification).

**Methods**:

#### 1. Session-Based Authentication
**How it Works**:
1. User logs in with credentials
2. Server creates session, stores in database
3. Session ID sent to client (cookie)
4. Client sends session ID with each request

**Advantages**:
- Server has full control (can revoke sessions)
- Familiar pattern
- Secure (session data on server)

**Disadvantages**:
- Stateful (requires session storage)
- Harder to scale horizontally
- CSRF vulnerability (needs protection)

**Best For**:
- Traditional web applications
- Admin panels
- Applications requiring immediate session revocation

**Scaling Challenges**:
- Session storage becomes bottleneck
- Solutions: Redis for session store, sticky sessions

#### 2. Token-Based Authentication (JWT)
**How it Works**:
1. User logs in with credentials
2. Server generates signed token (JWT)
3. Client stores token (localStorage/cookie)
4. Client sends token in Authorization header

**Advantages**:
- Stateless (no server-side session storage)
- Easy to scale horizontally
- Works across domains (CORS-friendly)
- Mobile-friendly

**Disadvantages**:
- Can't revoke tokens easily (until expiration)
- Token size (larger than session ID)
- XSS vulnerability if stored in localStorage

**Best For**:
- Microservices architectures
- Mobile applications
- Single Page Applications (SPAs)
- APIs

**Scaling Benefits**:
- No shared session state needed
- Any server can validate token
- Perfect for distributed systems

#### 3. OAuth 2.0 / OpenID Connect
**How it Works**:
- Delegate authentication to third party
- User authorizes app to access their data
- App receives access token

**Advantages**:
- Don't store passwords
- Leverage existing accounts (Google, GitHub)
- Standardized protocol

**Best For**:
- Third-party integrations
- "Sign in with Google/Facebook"
- API access delegation

#### 4. API Keys
**How it Works**:
- Long-lived secret key for each client
- Sent with each request (header or query param)

**Advantages**:
- Simple to implement
- Good for server-to-server

**Disadvantages**:
- No user context
- Hard to rotate
- If leaked, full access until revoked

**Best For**:
- Server-to-server communication
- Public APIs with usage tracking
- Internal microservices

#### 5. Multi-Factor Authentication (MFA)
**Factors**:
- Something you know (password)
- Something you have (phone, hardware token)
- Something you are (biometrics)

**Best For**:
- High-security applications
- Financial services
- Admin access

### Authorization
**Definition**: Verifying **what you can do** (permission verification).

**Models**:

#### 1. Role-Based Access Control (RBAC)
**How it Works**:
- Users assigned to roles (Admin, Editor, Viewer)
- Roles have permissions
- Check user's role before allowing action

**Advantages**:
- Simple to understand and implement
- Easy to manage at scale
- Widely supported

**Disadvantages**:
- Role explosion (too many roles)
- Inflexible for complex scenarios

**Best For**:
- Most applications
- Clear organizational hierarchy
- SaaS applications

**Example**: 
- Admin: can delete users
- Editor: can edit content
- Viewer: can only read

#### 2. Attribute-Based Access Control (ABAC)
**How it Works**:
- Policies based on attributes (user, resource, environment)
- More fine-grained than RBAC
- Dynamic evaluation

**Advantages**:
- Very flexible
- Context-aware (time, location, device)
- Fewer policies than RBAC roles

**Disadvantages**:
- Complex to implement
- Harder to debug
- Performance overhead

**Best For**:
- Complex authorization requirements
- Multi-tenant systems
- Healthcare, government (compliance)

**Example**: 
- Allow if: user.department == document.department AND time.hour < 18

#### 3. Access Control Lists (ACL)
**How it Works**:
- Each resource has list of who can access it
- Direct mapping of users to permissions

**Advantages**:
- Fine-grained control
- Easy to understand for specific resources

**Disadvantages**:
- Doesn't scale (too many entries)
- Hard to manage

**Best For**:
- File systems
- Small applications
- Resource-specific permissions

**Example**: Google Docs sharing (specific users can edit/view)

### Comparison Table

| Aspect | Authentication | Authorization |
|--------|----------------|---------------|
| **Question** | Who are you? | What can you do? |
| **Process** | Login, verify identity | Check permissions |
| **Methods** | Password, JWT, OAuth | RBAC, ABAC, ACL |
| **When** | Once per session | Every request |
| **Example** | User logs in with email/password | User tries to delete a post |

### Scaling Best Practices

**Authentication**:
- Use JWT for stateless scaling
- Cache user data to reduce database hits
- Implement token refresh mechanism
- Use Redis for session storage if session-based

**Authorization**:
- Cache permissions (with TTL)
- Evaluate permissions at API Gateway
- Use policy engines (OPA, Casbin) for complex rules
- Denormalize permissions for faster lookups

---

## Object Storage vs Block Storage vs File Storage

### Object Storage
**Examples**: Amazon S3, Google Cloud Storage, Azure Blob Storage, MinIO

**How it Works**:
- Data stored as objects with metadata
- Flat namespace (no hierarchy)
- Accessed via HTTP/REST API
- Each object has unique ID

**Characteristics**:
- Highly scalable (petabytes+)
- Eventually consistent
- No file system overhead
- Rich metadata support

**Best For**:
- Static assets (images, videos, documents)
- Backups and archives
- Data lakes and analytics
- Content distribution
- Unstructured data

**Scaling Benefits**:
- Virtually unlimited scalability
- Built-in redundancy and durability (99.999999999%)
- Geographic replication
- Pay per GB stored and transferred

**Use Cases**:
- Netflix video storage
- Dropbox file storage
- Website static assets
- Machine learning datasets
- Log archival

**Limitations**:
- Can't modify objects (must replace)
- Higher latency than block storage
- Not suitable for databases

### Block Storage
**Examples**: Amazon EBS, Google Persistent Disk, Azure Disk Storage, SAN

**How it Works**:
- Data stored in fixed-size blocks
- Appears as raw disk to OS
- Requires file system on top
- Low-level access

**Characteristics**:
- Low latency, high IOPS
- Strongly consistent
- Can be formatted with any file system
- Attached to single instance (typically)

**Best For**:
- Databases (MySQL, PostgreSQL)
- Virtual machine boot disks
- High-performance applications
- Transactional workloads
- Applications requiring low latency

**Scaling Strategies**:
- Increase volume size (vertical)
- Use RAID for performance/redundancy
- Snapshot and restore
- Provision IOPS for guaranteed performance

**Use Cases**:
- Database storage
- Email servers
- Enterprise applications
- Boot volumes for VMs

**Limitations**:
- More expensive than object storage
- Typically single-instance attachment
- Manual scaling required

### File Storage (NFS/SMB)
**Examples**: Amazon EFS, Azure Files, Google Filestore, NFS servers

**How it Works**:
- Hierarchical file system (folders/files)
- Network-attached storage
- Multiple instances can mount simultaneously
- POSIX-compliant

**Characteristics**:
- Shared access across instances
- Familiar file system interface
- Supports file locking
- Moderate performance

**Best For**:
- Shared application data
- Content management systems
- Web serving (WordPress, etc.)
- Home directories
- Legacy applications requiring file system

**Scaling Benefits**:
- Automatic scaling (managed services)
- Shared across multiple servers
- No application changes needed

**Use Cases**:
- Shared web content
- Development environments
- Media processing workflows
- Container persistent storage

**Limitations**:
- More expensive than object storage
- Performance limited by network
- Scaling limits (compared to object storage)

### Comparison Table

| Feature | Object Storage | Block Storage | File Storage |
|---------|----------------|---------------|--------------|
| **Access Method** | HTTP/REST API | Block-level (iSCSI, FC) | File-level (NFS, SMB) |
| **Structure** | Flat (buckets/objects) | Raw blocks | Hierarchical (folders) |
| **Scalability** | Unlimited | Limited by volume size | Moderate |
| **Performance** | Moderate | Very High (low latency) | Moderate |
| **Cost** | Low | High | Moderate |
| **Shared Access** | Yes (concurrent reads) | No (single instance) | Yes (multi-instance) |
| **Use Case** | Static content, backups | Databases, VMs | Shared files, CMS |
| **Consistency** | Eventual | Strong | Strong |
| **Durability** | 11 nines | Depends on RAID/snapshots | Depends on implementation |

### Choosing the Right Storage

**Use Object Storage when**:
- Storing large amounts of unstructured data
- Need unlimited scalability
- Cost is a concern
- Data accessed via API

**Use Block Storage when**:
- Running databases
- Need low latency and high IOPS
- Require strong consistency
- Single-instance access

**Use File Storage when**:
- Multiple servers need shared access
- Legacy applications require file system
- Need hierarchical organization
- Collaborative workflows

---

## Push vs Pull Architecture

### Pull Architecture (Polling)
**How it Works**: Client periodically requests data from server.

**Characteristics**:
- Client initiates requests
- Server responds with current state
- Interval-based checking

**Advantages**:
- Simple to implement
- Works with standard HTTP
- Firewall-friendly
- Client controls polling rate

**Disadvantages**:
- Inefficient (many empty responses)
- Higher latency (up to polling interval)
- Wastes bandwidth and server resources
- Doesn't scale well

**Best For**:
- Low-frequency updates
- Simple applications
- When push infrastructure unavailable

**Scaling Challenges**:
- N clients = N requests per interval
- Server load increases linearly
- Most requests return "no new data"

**Example Use Cases**:
- Email client checking for new mail (every 5 min)
- Weather app updating forecast
- Checking job status

**Optimization**: Long Polling
- Client requests, server holds connection until data available
- Reduces empty responses
- Still client-initiated

### Push Architecture
**How it Works**: Server sends data to client when available.

**Technologies**:

#### 1. WebSockets
**Characteristics**:
- Bi-directional, full-duplex communication
- Persistent connection
- Low overhead after handshake

**Best For**:
- Real-time chat applications
- Live collaboration tools
- Gaming
- Financial trading platforms

**Scaling**:
- Requires maintaining open connections
- Use connection pooling
- Horizontal scaling with sticky sessions or message broker

#### 2. Server-Sent Events (SSE)
**Characteristics**:
- Unidirectional (server → client)
- Built on HTTP
- Automatic reconnection
- Simpler than WebSockets

**Best For**:
- Live feeds (news, sports scores)
- Real-time notifications
- Progress updates
- Monitoring dashboards

**Scaling**:
- Less resource-intensive than WebSockets
- Works with HTTP/2 multiplexing

#### 3. Push Notifications
**Characteristics**:
- Mobile/browser notifications
- Works when app is closed
- Platform-specific (APNs, FCM)

**Best For**:
- Mobile applications
- User engagement
- Critical alerts

#### 4. Webhooks
**Characteristics**:
- HTTP callbacks to client-provided URL
- Event-driven
- Asynchronous

**Best For**:
- Third-party integrations
- Payment confirmations (Stripe)
- CI/CD pipelines
- Event notifications

**Scaling Benefits**:
- Efficient (only send when data changes)
- Lower latency (immediate updates)
- Reduces server load (no polling)
- Better user experience

**Scaling Challenges**:
- Maintaining connections (WebSockets)
- Requires push infrastructure
- Harder to implement
- Firewall/NAT traversal

### Comparison Table

| Aspect | Pull (Polling) | Push |
|--------|----------------|------|
| **Initiator** | Client | Server |
| **Latency** | High (polling interval) | Low (immediate) |
| **Efficiency** | Low (many empty requests) | High (only when data changes) |
| **Complexity** | Simple | Moderate to High |
| **Scalability** | Poor (N clients = N requests) | Better (only on events) |
| **Use Case** | Infrequent updates | Real-time updates |

### Hybrid Approach
Many systems use **both**:
- **Push** for real-time updates (WebSockets)
- **Pull** as fallback (if WebSocket fails)
- **Pull** for initial data load

**Example**: Slack
- WebSocket for real-time messages
- Polling as fallback
- REST API for loading history

---

## Stateful vs Stateless Services

### Stateless Services
**Definition**: Service doesn't store client state between requests. Each request contains all necessary information.

**Characteristics**:
- No session data stored on server
- Each request is independent
- Any server can handle any request

**Advantages**:
- Easy to scale horizontally (add more servers)
- No session synchronization needed
- Simple load balancing (round-robin works)
- Fault-tolerant (server failure doesn't lose state)
- Easy to deploy and update

**Disadvantages**:
- May need to fetch user context repeatedly
- Larger request payloads (must include context)
- Requires external state storage (database, cache)

**Best For**:
- REST APIs
- Microservices
- Cloud-native applications
- High-scale web applications

**Scaling Benefits**:
- Linear horizontal scaling
- Auto-scaling works perfectly
- No sticky sessions needed
- Can use any load balancing algorithm

**Example**:
```
# Each request includes auth token
GET /api/users/123
Authorization: Bearer <token>

# Server validates token, fetches user data, responds
# No state stored on server
```

**Use Cases**:
- Public APIs (Stripe, Twilio)
- Serverless functions (AWS Lambda)
- Microservices
- RESTful web services

### Stateful Services
**Definition**: Service maintains client state between requests.

**Characteristics**:
- Session data stored on server
- Requests depend on previous interactions
- Client must connect to same server (sticky sessions)

**Advantages**:
- Faster (state already in memory)
- Smaller request payloads
- Can maintain complex state (game state, shopping cart)
- Better for real-time interactions

**Disadvantages**:
- Harder to scale (need sticky sessions)
- Server failure loses state
- Complex load balancing
- Difficult to deploy updates (drain connections)

**Best For**:
- WebSocket connections
- Online gaming
- Real-time collaboration
- Long-running transactions
- Streaming services

**Scaling Challenges**:
- Requires sticky sessions (session affinity)
- State synchronization across servers
- Graceful shutdown complexity
- Auto-scaling is harder

**Solutions**:
- Use external session store (Redis)
- Session replication across servers
- Consistent hashing for routing

**Example**:
```
# User logs in, server creates session
POST /login → Session ID: abc123

# Subsequent requests use session ID
GET /dashboard
Cookie: session_id=abc123

# Server looks up session, knows user context
```

**Use Cases**:
- Chat applications (WebSocket)
- Multiplayer games
- Video streaming
- Shopping carts (traditional)
- Admin dashboards with complex state

### Hybrid Approach: Stateless with External State

**Best Practice**: Design stateless services but store state externally.

**Architecture**:
- Application servers are stateless
- State stored in Redis, Memcached, or database
- Session ID in cookie/token
- Any server can retrieve state from external store

**Benefits**:
- Scalability of stateless
- State persistence of stateful
- Fault tolerance (state survives server failure)

**Example**: E-commerce
- Shopping cart stored in Redis (key: user_id)
- Any web server can retrieve cart
- Servers remain stateless

### Comparison Table

| Aspect | Stateless | Stateful |
|--------|-----------|----------|
| **State Storage** | None (or external) | On server |
| **Scalability** | Excellent | Challenging |
| **Load Balancing** | Simple (any algorithm) | Complex (sticky sessions) |
| **Fault Tolerance** | High | Low (state loss) |
| **Performance** | May need state lookup | Fast (state in memory) |
| **Deployment** | Easy (rolling updates) | Complex (drain connections) |
| **Use Case** | APIs, microservices | Real-time, gaming |

---

## Database Indexing Strategies

### What is an Index?
**Definition**: Data structure that improves query performance by allowing fast lookups.

**Trade-off**: Faster reads, slower writes (must update index).

### Index Types

#### 1. B-Tree Index (Default)
**How it Works**: Balanced tree structure, sorted keys.

**Best For**:
- Range queries (`WHERE age BETWEEN 18 AND 65`)
- Sorting (`ORDER BY created_at`)
- Equality searches (`WHERE user_id = 123`)
- Most general-purpose queries

**Characteristics**:
- O(log n) lookup time
- Supports prefix matching (`WHERE name LIKE 'John%'`)
- Most common index type

**Example**:
```sql
CREATE INDEX idx_users_email ON users(email);
```

#### 2. Hash Index
**How it Works**: Hash function maps keys to locations.

**Best For**:
- Exact equality searches (`WHERE id = 123`)
- Very fast lookups O(1)

**Limitations**:
- No range queries
- No sorting
- No prefix matching

**Use Case**: Primary key lookups, cache keys

#### 3. Full-Text Index
**How it Works**: Inverted index for text search.

**Best For**:
- Text search queries (`MATCH AGAINST`)
- Natural language search
- Keyword searching

**Example**:
```sql
CREATE FULLTEXT INDEX idx_posts_content ON posts(title, body);
SELECT * FROM posts WHERE MATCH(title, body) AGAINST('database scaling');
```

**Use Case**: Blog search, documentation search, product search

#### 4. Composite Index (Multi-Column)
**How it Works**: Index on multiple columns.

**Best For**:
- Queries filtering on multiple columns
- Follows left-prefix rule

**Example**:
```sql
CREATE INDEX idx_users_city_age ON users(city, age);

-- Uses index (left-prefix)
WHERE city = 'NYC' AND age = 25  ✓
WHERE city = 'NYC'                ✓

-- Doesn't use index
WHERE age = 25                    ✗
```

**Best Practice**: Order columns by selectivity (most selective first).

#### 5. Covering Index
**Definition**: Index includes all columns needed for query.

**Benefit**: Query can be satisfied entirely from index (no table lookup).

**Example**:
```sql
CREATE INDEX idx_users_email_name ON users(email, name);

-- Covering index (only needs email and name)
SELECT name FROM users WHERE email = 'user@example.com';
```

**Scaling Benefit**: Dramatically faster queries (index-only scan).

#### 6. Partial Index
**Definition**: Index on subset of rows.

**Best For**:
- Queries with common WHERE clause
- Reduces index size

**Example**:
```sql
-- Only index active users
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';
```

**Use Case**: Soft-deleted records, status-based queries

#### 7. Unique Index
**Definition**: Enforces uniqueness constraint.

**Example**:
```sql
CREATE UNIQUE INDEX idx_users_email ON users(email);
```

**Use Case**: Email addresses, usernames, SKUs

### Indexing Best Practices

#### When to Create Indexes

**Create indexes for**:
- Columns in WHERE clauses
- Columns in JOIN conditions
- Columns in ORDER BY
- Columns in GROUP BY
- Foreign keys

**Don't over-index**:
- Every index slows down writes
- Indexes consume disk space
- Too many indexes confuse query optimizer

#### Index Selectivity
**Selectivity**: Ratio of distinct values to total rows.

- **High selectivity** (good): email, user_id (many unique values)
- **Low selectivity** (bad): gender, boolean flags (few unique values)

**Rule**: Index high-selectivity columns.

#### Query Optimization

**Analyze queries**:
```sql
EXPLAIN SELECT * FROM users WHERE email = 'user@example.com';
```

**Look for**:
- Table scans (bad) → add index
- Index scans (good)
- Index-only scans (best)

### Scaling Strategies

#### 1. Index Maintenance
- Rebuild fragmented indexes periodically
- Update statistics for query optimizer
- Monitor index usage (drop unused indexes)

#### 2. Partitioning + Indexing
- Partition large tables
- Index each partition
- Reduces index size per partition

**Example**: Partition orders by year, index each partition.

#### 3. Database-Specific Optimizations

**PostgreSQL**:
- `BRIN` indexes for sequential data (timestamps)
- `GIN/GiST` for JSON, arrays, full-text

**MySQL**:
- InnoDB clustered index (primary key)
- Use covering indexes aggressively

**MongoDB**:
- Compound indexes
- Text indexes for search
- Geospatial indexes

### Common Pitfalls

| Pitfall | Impact | Solution |
|---------|--------|----------|
| **No indexes** | Slow queries, table scans | Add indexes on WHERE/JOIN columns |
| **Too many indexes** | Slow writes, wasted space | Drop unused indexes |
| **Wrong column order** | Index not used | Order by selectivity |
| **Function on indexed column** | Index not used | Use functional index or rewrite query |
| **Implicit type conversion** | Index not used | Match column types in query |

**Example of function pitfall**:
```sql
-- Doesn't use index on created_at
WHERE DATE(created_at) = '2024-01-01'

-- Uses index
WHERE created_at >= '2024-01-01' AND created_at < '2024-01-02'
```

---

## Service Discovery Patterns

### Why Service Discovery?
In microservices, services need to find each other dynamically (IP addresses change with auto-scaling).

### Client-Side Discovery
**How it Works**:
1. Service registers with service registry
2. Client queries registry for service location
3. Client makes direct request to service
4. Client handles load balancing

**Examples**: Netflix Eureka, Consul

**Advantages**:
- Client controls load balancing logic
- No additional network hop
- Flexible routing decisions

**Disadvantages**:
- Client complexity (must implement discovery)
- Language-specific clients needed
- Tight coupling to registry

**Best For**:
- Homogeneous environments (single language)
- Need custom load balancing logic
- Internal microservices

**Architecture**:
```
Client → Service Registry (get service locations)
Client → Service Instance (direct call)
```

### Server-Side Discovery
**How it Works**:
1. Service registers with service registry
2. Client makes request to load balancer
3. Load balancer queries registry
4. Load balancer forwards to service instance

**Examples**: AWS ELB + ECS, Kubernetes Services, Nginx + Consul

**Advantages**:
- Client simplicity (just call load balancer)
- Language-agnostic
- Centralized load balancing
- Loose coupling

**Disadvantages**:
- Additional network hop (latency)
- Load balancer is potential bottleneck
- Another component to manage

**Best For**:
- Polyglot environments (multiple languages)
- Kubernetes/container orchestration
- Public-facing services

**Architecture**:
```
Client → Load Balancer → Service Registry (query)
Load Balancer → Service Instance (forward)
```

### Service Registry Patterns

#### 1. Self-Registration
**How it Works**: Service instance registers itself on startup.

**Advantages**:
- Simple, direct control
- No external component needed

**Disadvantages**:
- Service must know about registry
- Coupling to infrastructure

**Example**:
```python
# Service startup
registry.register(service_name='user-service', host='10.0.1.5', port=8080)

# Service shutdown
registry.deregister(service_name='user-service', host='10.0.1.5')
```

#### 2. Third-Party Registration
**How it Works**: Separate registrar component registers services.

**Advantages**:
- Services decoupled from registry
- Centralized registration logic
- Works with legacy services

**Disadvantages**:
- Additional component to manage
- Registrar must detect service health

**Example**: Kubernetes (kubelet registers pods), Registrator

### Popular Service Discovery Tools

#### 1. Consul (HashiCorp)
**Features**:
- Service registry
- Health checking
- DNS and HTTP interface
- Multi-datacenter support
- Key-value store

**Best For**: Multi-cloud, hybrid environments

#### 2. Etcd
**Features**:
- Distributed key-value store
- Strong consistency (Raft)
- Watch API for changes

**Best For**: Kubernetes (native), configuration management

#### 3. Zookeeper
**Features**:
- Distributed coordination
- Hierarchical namespace
- Mature, battle-tested

**Best For**: Kafka, Hadoop ecosystems

#### 4. Kubernetes Services
**Features**:
- Built-in service discovery (DNS)
- Automatic load balancing
- Service mesh integration

**Best For**: Container orchestration

#### 5. AWS Cloud Map
**Features**:
- Managed service discovery
- Integrates with ECS, EKS
- Health checking

**Best For**: AWS-native applications

### Health Checking

**Why**: Ensure only healthy instances receive traffic.

**Types**:
- **Active**: Registry pings service periodically
- **Passive**: Service sends heartbeats
- **Application-level**: Custom health endpoint (`/health`)

**Example**:
```
GET /health
Response: 200 OK
{
  "status": "healthy",
  "database": "connected",
  "cache": "connected"
}
```

### Scaling Considerations

**Service Registry**:
- Must be highly available (critical component)
- Use clustered deployment (3-5 nodes)
- Eventual consistency acceptable (brief stale data okay)

**DNS-Based Discovery**:
- Simple, universal
- TTL caching issues (stale data)
- Limited load balancing options

**Comparison**:

| Pattern | Complexity | Latency | Coupling | Best For |
|---------|------------|---------|----------|----------|
| **Client-Side** | High | Low | Tight | Homogeneous services |
| **Server-Side** | Low | Medium | Loose | Polyglot, Kubernetes |
| **DNS** | Very Low | Low | Very Loose | Simple setups |

---

## Circuit Breaker vs Retry vs Timeout

### Why Resilience Patterns?
Prevent cascading failures in distributed systems. When a service fails, don't let it bring down the entire system.

### Timeout Pattern
**Definition**: Set maximum time to wait for response.

**How it Works**:
```python
response = requests.get(url, timeout=5)  # 5 second timeout
```

**Purpose**:
- Prevent indefinite waiting
- Free up resources quickly
- Fail fast

**Best Practices**:
- Set realistic timeouts (measure P99 latency)
- Different timeouts for different operations
- Connection timeout vs read timeout

**Example Timeouts**:
- Database query: 5-10 seconds
- External API: 30 seconds
- Internal microservice: 2-5 seconds

**Scaling Benefit**: Prevents thread pool exhaustion.

### Retry Pattern
**Definition**: Automatically retry failed requests.

**When to Retry**:
- Transient failures (network blip, temporary overload)
- Idempotent operations (safe to retry)
- Timeout errors

**When NOT to Retry**:
- Client errors (400, 401, 404) - won't succeed
- Non-idempotent operations (payment processing)
- Server explicitly says don't retry (429, 503 with Retry-After)

**Retry Strategies**:

#### 1. Fixed Delay
```python
for attempt in range(3):
    try:
        return make_request()
    except:
        time.sleep(1)  # Wait 1 second
```

**Problem**: Thundering herd (all clients retry simultaneously).

#### 2. Exponential Backoff
```python
for attempt in range(5):
    try:
        return make_request()
    except:
        time.sleep(2 ** attempt)  # 1s, 2s, 4s, 8s, 16s
```

**Benefit**: Gives service time to recover.

#### 3. Exponential Backoff + Jitter
```python
delay = (2 ** attempt) + random.uniform(0, 1)
```

**Benefit**: Prevents synchronized retries (best practice).

**Best Practices**:
- Limit retry attempts (3-5 max)
- Use exponential backoff with jitter
- Log retries for monitoring
- Set maximum backoff time

**Scaling Benefit**: Allows temporary failures to self-heal.

### Circuit Breaker Pattern
**Definition**: Stop making requests to failing service, fail fast instead.

**States**:

#### 1. Closed (Normal)
- Requests pass through
- Count failures
- If failures exceed threshold → Open

#### 2. Open (Failing)
- Requests immediately fail (don't call service)
- After timeout period → Half-Open

#### 3. Half-Open (Testing)
- Allow limited requests through
- If successful → Closed
- If failed → Open

**How it Works**:
```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.state = 'CLOSED'
        self.last_failure_time = None
    
    def call(self, func):
        if self.state == 'OPEN':
            if time.time() - self.last_failure_time > self.timeout:
                self.state = 'HALF_OPEN'
            else:
                raise CircuitBreakerOpen()
        
        try:
            result = func()
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise
    
    def on_success(self):
        self.failure_count = 0
        self.state = 'CLOSED'
    
    def on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = 'OPEN'
```

**Benefits**:
- Prevents cascading failures
- Gives failing service time to recover
- Fails fast (better UX than timeout)
- Reduces load on failing service

**Best For**:
- External service calls
- Database connections
- Microservices communication

**Configuration**:
- Failure threshold: 5-10 failures
- Timeout: 30-60 seconds
- Half-open requests: 1-3

**Scaling Benefit**: Prevents one failing service from taking down entire system.

### Bulkhead Pattern
**Definition**: Isolate resources to prevent total failure.

**Analogy**: Ship compartments - if one floods, others stay dry.

**Implementation**:
- Separate thread pools per service
- Separate connection pools
- Resource quotas

**Example**:
```python
# Separate thread pools
user_service_pool = ThreadPoolExecutor(max_workers=10)
payment_service_pool = ThreadPoolExecutor(max_workers=5)

# If payment service hangs, user service still works
```

**Scaling Benefit**: Fault isolation, prevents resource exhaustion.

### Combining Patterns

**Best Practice**: Use all together for maximum resilience.

```python
# 1. Timeout (fail fast)
# 2. Retry with exponential backoff (handle transient failures)
# 3. Circuit breaker (prevent cascading failures)
# 4. Bulkhead (isolate resources)

@circuit_breaker
@retry(max_attempts=3, backoff='exponential')
@timeout(seconds=5)
def call_external_service():
    return requests.get(url)
```

**Comparison Table**:

| Pattern | Purpose | When to Use | Scaling Benefit |
|---------|---------|-------------|-----------------|
| **Timeout** | Fail fast | Always | Prevents resource exhaustion |
| **Retry** | Handle transient failures | Idempotent operations | Self-healing |
| **Circuit Breaker** | Prevent cascading failures | External dependencies | System stability |
| **Bulkhead** | Isolate failures | Critical services | Fault isolation |

---

## Blue-Green vs Canary vs Rolling Deployment

### Why Deployment Strategies Matter
- Minimize downtime
- Reduce deployment risk
- Enable quick rollback
- Test in production safely

### Blue-Green Deployment
**How it Works**:
1. Two identical environments: Blue (current) and Green (new)
2. Deploy new version to Green
3. Test Green environment
4. Switch traffic from Blue to Green (instant cutover)
5. Keep Blue for quick rollback

**Advantages**:
- Zero downtime
- Instant rollback (switch back to Blue)
- Full testing before cutover
- Simple to understand

**Disadvantages**:
- Requires 2x infrastructure (expensive)
- Database migrations tricky (must be backward compatible)
- All-or-nothing switch (risky for major changes)

**Best For**:
- Critical applications requiring zero downtime
- When you can afford duplicate infrastructure
- Infrequent deployments

**Example**:
```
Blue (v1.0) ← 100% traffic
Green (v1.1) ← 0% traffic (testing)

[Switch]

Blue (v1.0) ← 0% traffic (standby for rollback)
Green (v1.1) ← 100% traffic
```

**Scaling Consideration**: Doubles infrastructure cost during deployment.

### Canary Deployment
**How it Works**:
1. Deploy new version to small subset of servers (canary)
2. Route small % of traffic to canary (e.g., 5%)
3. Monitor metrics (errors, latency, business KPIs)
4. Gradually increase traffic if healthy (10% → 25% → 50% → 100%)
5. Rollback if issues detected

**Advantages**:
- Low risk (limited blast radius)
- Real production testing
- Gradual rollout
- Can target specific users (beta testers)

**Disadvantages**:
- Slower deployment
- Complex traffic routing needed
- Requires good monitoring
- Two versions running simultaneously

**Best For**:
- High-risk changes
- User-facing applications
- Continuous deployment
- A/B testing

**Example**:
```
v1.0: 95% traffic
v1.1: 5% traffic (canary)

[Monitor metrics]

v1.0: 50% traffic
v1.1: 50% traffic

[Continue if healthy]

v1.0: 0% traffic
v1.1: 100% traffic
```

**Canary Metrics to Monitor**:
- Error rate
- Response time (P50, P95, P99)
- CPU/memory usage
- Business metrics (conversion rate, sign-ups)

**Scaling Benefit**: Catches issues before affecting all users.

### Rolling Deployment
**How it Works**:
1. Gradually replace instances with new version
2. Update small batch at a time (e.g., 2 out of 10 servers)
3. Wait for health checks
4. Continue to next batch
5. Repeat until all updated

**Advantages**:
- No extra infrastructure needed
- Gradual rollout
- Automatic rollback (stop deployment)
- Built into most orchestration tools

**Disadvantages**:
- Two versions running simultaneously (compatibility required)
- Slower than blue-green
- Rollback requires another rolling deployment
- Potential for inconsistent user experience

**Best For**:
- Stateless applications
- Backward-compatible changes
- Resource-constrained environments
- Kubernetes, ECS deployments

**Example**:
```
10 servers total, update 2 at a time:

[v1.0, v1.0, v1.0, v1.0, v1.0, v1.0, v1.0, v1.0, v1.0, v1.0]
[v1.1, v1.1, v1.0, v1.0, v1.0, v1.0, v1.0, v1.0, v1.0, v1.0]
[v1.1, v1.1, v1.1, v1.1, v1.0, v1.0, v1.0, v1.0, v1.0, v1.0]
...
[v1.1, v1.1, v1.1, v1.1, v1.1, v1.1, v1.1, v1.1, v1.1, v1.1]
```

**Kubernetes Example**:
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # Max new pods above desired count
    maxUnavailable: 1  # Max pods unavailable during update
```

**Scaling Benefit**: No additional infrastructure cost.

### Recreate Deployment
**How it Works**:
1. Stop all old version instances
2. Start all new version instances

**Advantages**:
- Simplest strategy
- No version compatibility issues

**Disadvantages**:
- Downtime during deployment
- All-or-nothing (high risk)

**Best For**:
- Development/staging environments
- Applications that can tolerate downtime
- Major breaking changes

### Feature Flags (Dark Launch)
**How it Works**:
- Deploy new code (disabled by default)
- Enable feature for specific users/percentage
- Gradually roll out feature
- Rollback = disable flag (no redeployment)

**Advantages**:
- Decouple deployment from release
- Instant rollback
- A/B testing
- Gradual feature rollout

**Best For**:
- Continuous deployment
- Experimentation
- Phased feature releases

**Example**:
```python
if feature_flag.is_enabled('new_checkout', user_id):
    return new_checkout_flow()
else:
    return old_checkout_flow()
```

### Comparison Table

| Strategy | Downtime | Risk | Cost | Rollback Speed | Best For |
|----------|----------|------|------|----------------|----------|
| **Blue-Green** | None | Medium | High (2x infra) | Instant | Critical apps, infrequent deploys |
| **Canary** | None | Low | Low | Fast | High-risk changes, continuous deployment |
| **Rolling** | None | Medium | Low | Slow | Stateless apps, Kubernetes |
| **Recreate** | Yes | High | Low | Slow | Dev/staging, breaking changes |
| **Feature Flags** | None | Very Low | Low | Instant | Continuous deployment, A/B testing |

### Choosing the Right Strategy

**Use Blue-Green when**:
- Zero downtime is critical
- You can afford duplicate infrastructure
- Need instant rollback capability

**Use Canary when**:
- Deploying high-risk changes
- Want to test in production with real users
- Have good monitoring in place

**Use Rolling when**:
- Using Kubernetes/container orchestration
- Changes are backward compatible
- Want to minimize infrastructure cost

**Use Feature Flags when**:
- Practicing continuous deployment
- Want to decouple deployment from release
- Need A/B testing capability

### Best Practices

1. **Automate deployments** (CI/CD pipelines)
2. **Monitor actively** during deployment
3. **Have rollback plan** ready
4. **Test backward compatibility**
5. **Use health checks** to verify deployment
6. **Gradual rollout** for high-risk changes
7. **Database migrations** should be backward compatible

---

## Summary: Choosing the Right Technology


### Decision Framework

1. **Understand Your Requirements**
   - Traffic patterns (read vs write heavy)
   - Consistency requirements (strong vs eventual)
   - Latency requirements
   - Data structure and relationships
   - Team expertise

2. **Start Simple, Scale Gradually**
   - Begin with proven technologies
   - Don't over-engineer for future scale
   - Measure before optimizing
   - Refactor when you hit limits

3. **Consider Trade-offs**
   - Every technology has trade-offs
   - No silver bullet exists
   - Context matters more than trends

### Common Architecture Patterns by Scale

**Small Scale (< 10K users)**:
- Monolithic application
- Single SQL database
- Simple caching (Redis)
- REST APIs
- Vertical scaling

**Medium Scale (10K - 1M users)**:
- Modular monolith or early microservices
- SQL with read replicas
- CDN for static assets
- Message queue for background jobs
- Horizontal scaling of app servers

**Large Scale (> 1M users)**:
- Microservices architecture
- Polyglot persistence (SQL + NoSQL)
- Multi-layer caching
- Event streaming (Kafka)
- Database sharding
- API Gateway
- Edge computing
- Auto-scaling infrastructure

---

## Additional Resources

- **Books**: "Designing Data-Intensive Applications" by Martin Kleppmann
- **Courses**: System Design Interview courses on platforms like Educative, Grokking
- **Practice**: Design systems like Twitter, Netflix, Uber to apply these concepts
- **Stay Updated**: Follow engineering blogs from companies like Netflix, Uber, Airbnb

---

**Remember**: The best architecture is the one that solves your specific problem with the resources and constraints you have. Start simple, measure, and evolve.
