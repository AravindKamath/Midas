# Midas Core – Event-Driven Financial Processing System

## Overview

Midas Core is a Spring Boot–based financial processing system built incrementally across five tasks. The application integrates:

- Apache Kafka for transaction ingestion
- H2 in-memory database for persistence
- External Incentives API for transaction enrichment
- REST API for balance querying

The system simulates a real-world microservice that processes transactions, applies incentives, updates user balances, and exposes data through HTTP endpoints.

---

## Architecture

```
Kafka Producer
      ↓
Kafka Listener
      ↓
Transaction Service
      ↓
Validation + Incentive API Call
      ↓
H2 Database (UserRecord + TransactionRecord)
      ↓
REST API (/balance)
```

---

## Technologies Used

- Java 17
- Spring Boot 3.2.5
- Spring Data JPA
- Spring Kafka
- H2 Database
- RestTemplate
- Maven
- Embedded Kafka (for tests)

---

## Task 1 – Project Setup & Environment Configuration

### Objective
Set up the development environment and configure dependencies.

### Key Actions
- Forked and cloned repository
- Installed Java 17
- Configured Maven
- Added required dependencies:
    - spring-boot-starter-web
    - spring-boot-starter-data-jpa
    - spring-kafka
    - h2
    - spring-boot-starter-test
    - spring-kafka-test
    - testcontainers kafka
- Configured `application.yml`
- Verified build and test execution

### Build Commands

```bash
mvn clean install
mvn test
```

---

## Task 2 – Kafka Integration

### Objective
Integrate Kafka message consumption into Midas Core.

### Implementation
- Implemented `@KafkaListener`
- Configured topic via `application.yml`
- Deserialized incoming messages into `Transaction` objects
- Ensured compatibility with embedded Kafka used in tests

### Result
Midas Core now consumes transactions from a Kafka topic dynamically.

---

## Task 3 – Database Integration & Transaction Validation

### Objective
Persist valid transactions and update user balances.

### Key Features Implemented
- Created `TransactionRecord` entity
- Established `@ManyToOne` relationship with `UserRecord`
- Implemented validation rules:
    - Sender must exist
    - Recipient must exist
    - Sender must have sufficient balance
- Updated balances accordingly
- Used `@Transactional` to ensure atomic operations

### Database Behavior
If transaction is valid:
- Sender balance is deducted
- Recipient balance is credited
- Transaction is recorded

If invalid:
- Transaction is discarded
- No database changes occur

---

## Task 4 – Incentive API Integration

### Objective
Enrich transactions with external incentive amounts.

### Integration Steps
- Created `Incentive` DTO
- Configured `RestTemplate` bean
- Posted validated transaction to:
  ```
  http://localhost:8080/incentive
  ```
- Received incentive amount
- Updated balance logic:
    - Sender: deduct transaction amount only
    - Recipient: add transaction amount + incentive
- Extended `TransactionRecord` to store incentive

### Running Incentive API

```bash
java -jar services/incentives-api.jar
```

### Result
Transactions now include external enrichment before persistence.

---

## Task 5 – REST API for Balance Query

### Objective
Expose a REST endpoint for querying user balances.

### Endpoint

```
GET /balance?userId={id}
```

### Behavior
- Returns JSON representation of `Balance`
- If user exists → return current balance
- If user does not exist → return 0
- Runs on port **33400**

### Configuration

```yaml
server:
  port: 33400
```

---

## Running the Application

### 1. Start Incentive API

```bash
java -jar services/incentives-api.jar
```

### 2. Run Midas Core

```bash
mvn spring-boot:run
```

### 3. Test Balance Endpoint

```
GET http://localhost:33400/balance?userId=1
```

---

## Testing

Run all tests:

```bash
mvn test
```

Run specific task:

```bash
mvn -Dtest=TaskFiveTests test
```

> Note: Incentive API must be running for Task 4 and Task 5 tests.

---

## Key Engineering Concepts Demonstrated

- Event-driven architecture
- Message queue consumption (Kafka)
- JPA entity relationships
- ACID transactional processing
- External service communication
- REST API design
- Spring Boot auto-configuration
- Integration testing with embedded infrastructure

---

## Final System Capabilities

Midas Core now:

- Consumes financial transactions via Kafka
- Validates transaction rules
- Applies incentive enrichment
- Persists transaction history
- Maintains user balances
- Exposes a REST API for querying balances

This represents a complete microservice pipeline from ingestion to exposure.

---

## Author

Aravind Kamath  
Backend Systems Simulation – Midas Core