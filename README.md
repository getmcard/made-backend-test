# Backend Developer Candidate Test

Welcome to the backend developer candidate test! This repository contains the necessary materials for you to implement and test your solution locally before submitting it for evaluation.

## 📋 Project Overview

Your task is to implement a backend solution for the Made Backend Test. You'll build a RESTful API that manages client transactions and statements, with specific business rules and performance requirements.

## 🎯 Objective

Develop a backend system that handles:
- Client transaction processing (credits and debits)
- Client statement generation
- Business rule validation
- High-performance concurrent operations

## 📁 Project Structure

- **README.md**: This document with instructions and requirements
- **simulations/**: Contains Gatling performance test scenarios
  - `pom.xml`: Maven configuration for running Gatling tests
  - `src/main/scala/MadeBackendTestSimulation.scala`: Scala-based load testing simulation

## 🚀 Getting Started

### Prerequisites

- Choose your preferred programming language and framework
- Set up a database (PostgreSQL, MySQL, or similar)
- Install Java 11 or higher (required for Maven and Gatling)
- Install Maven for running the performance tests
- Install Gatling Community Edition: [Download here](https://www.gatling.io/download-gatling-community-edition)
  - Extract the downloaded archive (optional - used for reference and advanced testing)

### Running the Performance Tests

1. **Run the simulation**:
   ```bash
   cd simulations
   mvn gatling:test -Dgatling.simulationClass=MadeBackendTestSimulation
   ```

   The simulation will test your API at `http://localhost:9999` with various load patterns and business rule validations.

   **Note**: Make sure your API is running on `http://localhost:9999` before starting the simulation. The test will fail if it cannot connect to your API.

### Understanding Test Results

After running the simulation, you'll see detailed results including:

- **Response times** for each endpoint
- **Success/failure rates** for different scenarios
- **Error messages** for failed requests
- **Throughput metrics** (requests per second)

**Common issues to watch for:**
- **HTTP 500 errors**: Check your server logs for exceptions
- **HTTP 422 errors**: Verify business rule validation (balance limits, input validation)
- **HTTP 404 errors**: Ensure client IDs 1-5 are properly initialized
- **Timeout errors**: Check if your API can handle the expected load
- **Data consistency errors**: Verify concurrent transaction handling

**Sample successful output:**
```
Simulation made-backend-test.MadeBackendTestSimulation completed in 4 minutes
================================================================================
---- Global Information --------------------------------------------------------
> request count                                         1000 (OK=1000   KO=0     )
> min response time                                       10 (OK=10      KO=-     )
> max response time                                      500 (OK=500     KO=-     )
> mean response time                                     150 (OK=150     KO=-     )
> std deviation                                          100 (OK=100     KO=-     )
> response time 50th percentile                          120 (OK=120     KO=-     )
> response time 75th percentile                          200 (OK=200     KO=-     )
> response time 95th percentile                          400 (OK=400     KO=-     )
> response time 99th percentile                          450 (OK=450     KO=-     )
> mean requests/sec                                     250 (OK=250     KO=-     )
================================================================================
```

## 📚 API Requirements

### Base URL
Your API should run on `http://localhost:9999`

### Endpoints

#### 1. Create Transaction
**POST** `/clients/{id}/transactions`

Creates a new transaction for a client.

**Request Body:**
```json
{
  "value": 1000,
  "type": "c",
  "description": "description"
}
```

**Parameters:**
- `id` (path): Client ID (integer, 1-5 for valid clients, 6 for testing 404 responses)
- `value` (body): Transaction amount in cents (positive integer)
- `type` (body): Transaction type - "c" for credit, "d" for debit
- `description` (body): Transaction description (string, 1-10 characters, non-empty)

**Response (200 OK):**
```json
{
  "limit": 100000,
  "balance": 1000
}
```

**Response (422 Unprocessable Entity):**
- Invalid transaction data
- Business rule violations

#### 2. Get Statement
**GET** `/clients/{id}/statement`

Retrieves the client's statement with recent transactions.

**Parameters:**
- `id` (path): Client ID (integer, 1-5 for valid clients, 6 for testing 404 responses)

**Response (200 OK):**
```json
{
  "balance": {
    "total": 1000,
    "limit": 100000,
    "date": "2024-01-01T00:00:00Z"
  },
  "latest_transactions": [
    {
      "value": 1000,
      "type": "c",
      "description": "description",
      "executed_at": "2024-01-01T00:00:00Z"
    }
  ]
}
```

**Response Fields:**
- `balance.total`: Current account balance (sum of all transactions)
- `balance.limit`: Client's credit limit
- `balance.date`: Statement generation timestamp
- `latest_transactions`: **Array of up to 10 most recent transactions**, ordered by execution date (most recent first)

**Response (404 Not Found):**
- Client ID doesn't exist

## 🔧 Business Rules

### Client Configuration
- **Client 1**: Limit = $1,000.00 (100,000 cents)
- **Client 2**: Limit = $800.00 (80,000 cents)
- **Client 3**: Limit = $10,000.00 (1,000,000 cents)
- **Client 4**: Limit = $100,000.00 (10,000,000 cents)
- **Client 5**: Limit = $5,000.00 (500,000 cents)
- **Client 6**: **Intentionally empty** - Used for testing 404 responses

**Important**: Client 6 is deliberately not initialized and should return HTTP 404 for all requests. This is part of the simulation test scenarios to verify proper error handling.

### Transaction Rules
1. **Credit transactions**: Always allowed, increase balance
2. **Debit transactions**: Only allowed if resulting balance ≥ -limit
3. **Value validation**: Must be positive integer
4. **Type validation**: Must be exactly "c" or "d"
5. **Description validation**: 1-10 characters, non-empty, not null

### Statement Rules
1. **Latest transactions**: Show up to 10 most recent transactions
2. **Ordering**: Most recent first (descending by execution time)
3. **Balance calculation**: Sum of all transactions (credits positive, debits negative)

## 🧪 Testing Your Implementation

### Manual Testing
```bash
# Test transaction creation
curl -X POST http://localhost:9999/clients/1/transactions \
  -H "Content-Type: application/json" \
  -d '{"value": 1000, "type": "c", "description": "test"}'

# Test statement retrieval
curl http://localhost:9999/clients/1/statement

# Test invalid transaction (should return 422)
curl -X POST http://localhost:9999/clients/1/transactions \
  -H "Content-Type: application/json" \
  -d '{"value": 1.5, "type": "d", "description": "invalid"}'

# Test non-existent client (should return 404)
curl http://localhost:9999/clients/6/statement
```

### Performance Testing
The Gatling simulation will:
- Test concurrent transaction processing
- Validate business rule consistency
- Measure response times under load
- Verify data integrity across concurrent operations

### Troubleshooting Common Issues

**1. Connection Refused (Connection refused)**
- Ensure your API is running on `http://localhost:9999`
- Check if the port is available and not blocked by firewall

**2. High Error Rate (KO > 0)**
- Check your API logs for detailed error messages
- Verify all business rules are correctly implemented
- Ensure proper error handling and HTTP status codes
- **Note**: Some 404 errors for Client 6 are expected and correct (testing error handling)

**3. Slow Response Times**
- Optimize database queries
- Consider connection pooling
- Check for memory leaks or resource bottlenecks
- Verify concurrent request handling

**4. Data Consistency Errors**
- Ensure proper transaction handling in your database
- Check for race conditions in concurrent operations
- Verify atomic operations for balance updates

**5. Memory Issues**
- Monitor memory usage during the test
- Check for memory leaks in your application
- Consider garbage collection tuning if using JVM-based languages

## 📊 Performance Expectations

Your implementation should demonstrate:
- **Response time**: Sub-second for most operations
- **Data consistency**: Maintain correct balances under concurrent load
- **Error handling**: Proper HTTP status codes and error responses
- **Scalability**: Ability to handle the load patterns defined in the simulation

The Gatling simulation will generate detailed performance reports showing your system's capabilities under various load conditions.

### Why Multiple Backend Instances?

The Gatling simulation is designed to test **production-like scenarios** where:
- **Load distribution** across multiple backend instances
- **Horizontal scaling** capabilities under high load
- **Load balancer** behavior and failover scenarios
- **Database concurrency** with multiple application instances
- **Real-world architecture** patterns that mirror production systems

This setup ensures your solution can handle the expected traffic patterns and demonstrates your understanding of scalable backend architecture.

### Why Resource Constraints Matter

The strict CPU and memory limits (1.5 CPU, 550MB total) are designed to:

- **Test efficiency**: Force optimization of algorithms and data structures
- **Simulate real-world constraints**: Production environments often have resource limitations
- **Evaluate resource management**: Assess ability to work within budget constraints
- **Demonstrate optimization skills**: Show understanding of performance vs. resource trade-offs
- **Ensure fair comparison**: All candidates work under identical resource constraints

**Performance under constraints is a key differentiator** - the best solutions will handle the full load efficiently within these tight limits.

## 📝 Submission Guidelines

### What to Include
1. **Source code** of your implementation
2. **README.md** with:
   - Setup and run instructions
   - Technology stack used
   - Any assumptions or design decisions
3. **Database schema** (if applicable)
4. **Docker configuration** - **REQUIRED**:
   - Docker Compose setup with **2 backend server instances**
   - **Load balancer** (Nginx recommended) to distribute traffic
   - Database container (PostgreSQL/MySQL)
   - Proper networking between containers
   - Health checks and proper startup order

### Production Architecture Requirements

Your Docker setup must include:

- **2 Backend Instances**: Scale horizontally to handle concurrent load
- **Load Balancer**: Distribute requests across backend instances
- **Database**: Persistent storage for transactions and client data
- **Proper Networking**: Containers must communicate correctly
- **Health Checks**: Ensure services are ready before accepting traffic

### ⚠️ Resource Constraints (CRITICAL)

To ensure fair evaluation and demonstrate efficient resource usage, your Docker setup must respect these **strict limits**:

- **Total CPU Limit**: Maximum **1.5 CPU units** across all services
- **Total Memory Limit**: Maximum **550MB** across all services

**Example Docker Compose with resource constraints:**
```yaml
version: '3.8'
services:
  nginx:
    image: nginx:latest
    ports:
      - "9999:9999"
    deploy:
      resources:
        limits:
          cpus: "0.17"
          memory: "10MB"
    depends_on:
      - backend1
      - backend2

  backend1:
    build: .
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: "200MB"
    depends_on:
      - database

  backend2:
    build: .
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: "200MB"
    depends_on:
      - database

  database:
    image: postgres:15
    environment:
      POSTGRES_DB: made_backend_test
    deploy:
      resources:
        limits:
          cpus: "0.33"
          memory: "140MB"
```

**Resource Allocation Strategy:**
- **Nginx**: 0.17 CPU, 10MB (lightweight load balancer)
- **Backend 1**: 0.5 CPU, 200MB (main processing)
- **Backend 2**: 0.5 CPU, 200MB (main processing)
- **Database**: 0.33 CPU, 140MB (data persistence)
- **Total**: 1.5 CPU, 550MB ✅

### How to Submit
1. Compress your project into a `.zip` file
2. Include your name in the filename: `backend-test-[YourName].zip`
3. Send to the hiring team with subject: "Backend Test Submission - [Your Name]"

## 🎯 Evaluation Criteria

Your submission will be evaluated on:

- **✅ Functionality**: Correct implementation of all requirements
- **🏗️ Architecture**: Clean, maintainable code structure
- **🐳 Docker Configuration**: **Critical** - Proper multi-container setup with load balancer
- **⚡ Performance**: Efficient handling of concurrent operations across multiple instances
- **🧪 Testing**: Proper validation and error handling
- **📖 Documentation**: Clear setup and usage instructions
- **🔒 Data Integrity**: Consistent behavior under load with horizontal scaling
- **🚀 Production Readiness**: Ability to handle real-world traffic patterns
- **💾 Resource Efficiency**: **Critical** - Must operate within 1.5 CPU and 550MB memory limits

## 🆘 Support

If you have questions about the requirements or need clarification, please reach out to the hiring team.

---

**Good luck! We look forward to seeing your solution! 🚀**