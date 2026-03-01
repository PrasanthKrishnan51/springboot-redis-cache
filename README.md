#  Spring Boot + MongoDB + Redis

A production-ready REST API built with **Spring Boot 3**, **Java 21**, **MongoDB** (persistence), and **Redis** (caching). Ships with Swagger UI, Docker Compose, Lombok, and JUnit 5 tests.

---

## 🗂 Project Structure

```
src/
├── main/java/com/example/app/
│   ├── Application.java               # Entry point (@EnableCaching)
│   ├── config/
│   │   ├── RedisConfig.java           # CacheManager + RedisTemplate (JSON)
│   │   └── OpenApiConfig.java         # Swagger metadata
│   ├── controller/
│   │   └── ProductController.java        # REST endpoints + Swagger annotations
│   ├── dto/
│   │   └── ProductDto.java               # CreateRequest, UpdateRequest, Response (records)
│   ├── exception/
│   │   ├── ProductException.java         # Domain exceptions
│   │   └── GlobalExceptionHandler.java # RFC 7807 ProblemDetail responses
│   ├── model/
│   │   └── Product.java                  # MongoDB @Document + Serializable
│   ├── repository/
│   │   └── ProductRepository.java        # MongoRepository + custom @Query methods
│   └── service/
│       └── ProductService.java           # Business logic + @Cacheable / @CacheEvict
└── test/java/com/example/app/
    ├── controller/ProductControllerTest.java  # @WebMvcTest + MockMvc
    └── service/ProductServiceTest.java        # Mockito unit tests
```

---

## 🚀 Quick Start

### Option A — Docker Compose (recommended)

```bash
docker-compose up -d
# App starts at http://localhost:8080
```

### Option B — Run locally

```bash
# 1. Start infra
docker run -d -p 27017:27017 mongo:7.0
docker run -d -p 6379:6379 redis:7.2-alpine

# 2. Build & run
mvn spring-boot:run
```

### Run tests

```bash
mvn test
```

---

## 🌐 API Endpoints

| Method | Endpoint                         | Description                    | Cache behaviour                         |
|--------|----------------------------------|--------------------------------|-----------------------------------------|
| GET    | `/api/v1/products`                  | List all products                 | `products::all` — 10 min TTL              |
| GET    | `/api/v1/products/{id}`             | Get product by ID                 | `product::{id}` — 30 min TTL             |
| GET    | `/api/v1/products/genre/{genre}`    | Filter by genre                | `products::genre:{genre}` — 10 min TTL   |
| GET    | `/api/v1/products/in-stock`         | Products with stock > 0           | `products::in-stock` — 10 min TTL        |
| GET    | `/api/v1/products/search?author=`   | Search by author               | ❌ Not cached (dynamic)                 |
| POST   | `/api/v1/products`                  | Create a product                  | Evicts `products::*`                       |
| PUT    | `/api/v1/products/{id}`             | Update a product                  | Updates `product::{id}`, evicts `products::*` |
| DELETE | `/api/v1/products/{id}`             | Delete a product                  | Evicts `product::{id}` + `products::*`       |

### Swagger UI

```
http://localhost:8080/swagger-ui.html
```

### Actuator

```
http://localhost:8080/actuator/health
http://localhost:8080/actuator/caches
http://localhost:8080/actuator/metrics
```

---

## 📦 Example cURL Requests

```bash
# Create a product
curl -X POST http://localhost:8080/api/v1/products \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "isbn": "9780132350884",
    "price": 39.99,
    "genre": "Programming",
    "stock": 10
  }'

# Get all products
curl http://localhost:8080/api/v1/products

# Get by genre
curl http://localhost:8080/api/v1/products/genre/programming

# Search by author
curl "http://localhost:8080/api/v1/products/search?author=Martin"

# Update
curl -X PUT http://localhost:8080/api/v1/products/{id} \
  -H "Content-Type: application/json" \
  -d '{"title":"Clean Code","author":"Robert C. Martin","price":34.99,"genre":"Programming","stock":5}'

# Delete
curl -X DELETE http://localhost:8080/api/v1/products/{id}
```

---

## 🏛 Architecture

```
Client
  │
  ▼
ProductController        ← validates input, maps HTTP ↔ DTOs
  │
  ▼
ProductService           ← business logic
  ├─ Redis HIT?  ──── return cached value immediately
  └─ Redis MISS? ──── query MongoDB → cache result → return
                               │
                               ▼
                          ProductRepository   ← MongoRepository
```

---

## ⚙ Configuration

| Property | Default | Env override |
|---|---|---|
| MongoDB URI | `mongodb://localhost:27017/productstoredb` | `MONGO_URI` |
| Redis host | `localhost` | `REDIS_HOST` |
| Redis port | `6379` | `REDIS_PORT` |
| Server port | `8080` | — |
| Cache TTL (products list) | 10 min | — |
| Cache TTL (single product) | 30 min | — |

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| Framework | Spring Boot 3.2, Java 21 |
| Persistence | MongoDB 7, Spring Data MongoDB |
| Caching | Redis 7, Spring Cache |
| API Docs | SpringDoc OpenAPI 3 (Swagger UI) |
| Boilerplate | Lombok |
| Testing | JUnit 5, Mockito, MockMvc, Flapdoodle Embedded Mongo |
| Packaging | Maven |
| Containers | Docker, Docker Compose |
