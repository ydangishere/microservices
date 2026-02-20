# Microservices Architecture Project

A **fundamental yet complete** microservices system with Auth Service, People Service, and Case Service integrated with Kafka, Redis, Elasticsearch, and JWT authentication.

---

## 🧪 How to test & use (TL;DR)

### Option A – Chỉ cần test UI (services đã chạy sẵn)

1. Mở link: **http://127.0.0.1:3000/admin-ui/index.html**
2. **Register** (email + password) → **Login**
3. Thử: **Create Person** → **Create Case** → **Search Cases**

### Option B – Chạy full từ đầu (local)

1. **Clone & install**
   ```bash
   git clone https://github.com/ydangishere/microservices.git
   cd microservices
   npm install
   npm run build --workspace=shared
   ```
2. **Infrastructure** (Docker): `npm run docker:up`
3. **3 services** (mở 3 terminal):
   - `cd services/auth-service && npx ts-node src/index.ts`
   - `cd services/people-service && npx ts-node src/index.ts`
   - `cd services/case-service && npx ts-node src/index.ts`
4. **Mở UI**: http://127.0.0.1:3000/admin-ui/index.html (hoặc double-click `admin-ui/index.html`)
5. **Register** → **Login** → tạo People, Cases, Search.

### Link để mở UI

| Môi trường | Link |
|------------|------|
| **Local** | http://127.0.0.1:3000/admin-ui/index.html |
| **Render (sau khi deploy)** | https://&lt;your-admin-ui-service&gt;.onrender.com |

Chi tiết từng bước nằm ở phần [Getting Started](#-getting-started) và [Demo Validation](#-demo-validation-ui--microservices) bên dưới.

---

## 📋 System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         API Gateway (optional)                   │
└────────────┬──────────────┬──────────────┬─────────────────────┘
             │              │              │
    ┌────────▼──────┐  ┌───▼──────────┐  ┌▼───────────────┐
    │ Auth Service  │  │People Service│  │ Case Service   │
    │  Port: 3101   │  │ Port: 3002   │  │  Port: 3003    │
    │               │  │              │  │                │
    │ - JWT Auth    │  │ - CRUD       │  │ - CRUD         │
    │ - Register    │  │ - Redis Cache│  │ - ES Search    │
    │ - Login       │  │ - Kafka Pub  │  │ - Kafka Sub    │
    └───────┬───────┘  └──────┬───────┘  └────┬───────────┘
            │                 │                │
            │                 │                │
    ┌───────▼─────────────────▼────────────────▼───────────┐
    │              PostgreSQL (3 databases)                 │
    │   - auth_db    - people_db    - case_db              │
    └───────────────────────────────────────────────────────┘

    ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐
    │    Redis     │  │    Kafka     │  │ Elasticsearch   │
    │  (Cache)     │  │  (Events)    │  │   (Search)      │
    │  Port: 6379  │  │  Port: 9092  │  │   Port: 9200    │
    └──────────────┘  └──────────────┘  └─────────────────┘
```

## 🎯 Main Components

### 1. **Auth Service** (Port 3101)
- **Context**: User authentication and authorization
- **Action**: JWT authentication, register, login, profile management
- **Result**: Stateless authentication, easy to scale
- **Metrics**: 401 rate, token refresh success rate

### 2. **People Service** (Port 3002)
- **Context**: People management (CRUD operations)
- **Action**: 
  - CRUD operations
  - Redis caching (reduce DB load)
  - Publish Kafka events (people.created, people.updated, people.deleted)
- **Result**: Reduced latency, async communication
- **Metrics**: Cache hit rate, Kafka publish success, P95 latency

### 3. **Case Service** (Port 3003)
- **Context**: Case/ticket management
- **Action**:
  - CRUD operations
  - Elasticsearch full-text search
  - Consume Kafka events from People Service
- **Result**: Fast search, service integration via events
- **Metrics**: Search latency, ES index lag, consumer lag

### 4. **Infrastructure**

#### PostgreSQL (Port 5432)
- **Purpose**: Main database, each service has its own DB
- **Databases**: `auth_db`, `people_db`, `case_db`

#### Redis (Port 6379)
- **Purpose**: Cache frequently read data
- **Use case**: Cache people list, individual person records
- **Metrics**: Cache hit rate, eviction rate

#### Kafka (Port 9092)
- **Purpose**: Event streaming between services
- **Topics**: `people.created`, `people.updated`, `people.deleted`
- **Metrics**: Consumer lag, processing time, retry count

#### Elasticsearch (Port 9200)
- **Purpose**: Full-text search, complex filtering
- **Use case**: Search cases by title/description
- **Metrics**: Search latency, index lag

## 🌐 Demo Validation (UI + Microservices)

### Link để test / dùng

- **Local**: [http://127.0.0.1:3000/admin-ui/index.html](http://127.0.0.1:3000/admin-ui/index.html)
- **Render (sau khi deploy)**: `https://<your-admin-ui-service>.onrender.com`

### Cách dùng UI (từng bước)

1. Mở link UI ở trên.
2. **Register**: nhập email, password, tên → bấm **REGISTER**.
3. **Login**: cùng email/password → bấm **LOGIN**.
4. **People**: Create New Person → Refresh List (xem cache: mở F12 Console).
5. **Cases**: Create New Case → Search Cases (test Elasticsearch).
6. **Health**: bấm **Check All Services** để xem Auth / People / Case có sống không.

Dùng link UI để test giao diện **hoặc** kiểm chứng end-to-end kiến trúc microservice (Auth, People, Case, Redis, Kafka, Elasticsearch).

### Render URL mapping (required before go-live)

Update `admin-ui/config.js` with your Render service URLs:

```javascript
window.MICROSERVICES_CONFIG = {
  authBaseUrl: 'https://<your-auth-service>.onrender.com',
  peopleBaseUrl: 'https://<your-people-service>.onrender.com',
  caseBaseUrl: 'https://<your-case-service>.onrender.com',
};
```

This is what allows users who open the Render UI link to call live backend services instead of localhost.

If you need full details, see `admin-ui/README.md`.

## 🚀 Getting Started

### Step 1: Clone and Install

```bash
cd d:\microservices
npm install
```

### Step 2: Start Infrastructure (Docker)

```bash
# Folder: d:\microservices
npm run docker:up
```

This will start:
- PostgreSQL with 3 databases
- Redis
- Kafka + Zookeeper
- Elasticsearch + Kibana

### Step 3: Setup Environment Variables

Copy `.env.example` to `.env` for each service:

```bash
# Auth service
cp services/auth-service/.env.example services/auth-service/.env

# People service
cp services/people-service/.env.example services/people-service/.env

# Case service
cp services/case-service/.env.example services/case-service/.env
```

### Step 4: Build Shared Package

```bash
# Folder: d:\microservices
npm run build --workspace=shared
```

### Step 5: Start Services (recommended local demo mode)

Open 3 separate terminals:

**Terminal 1 - Auth Service (Port 3101):**
```bash
# Folder: d:\microservices\services\auth-service
npx ts-node src/index.ts
```

**Terminal 2 - People Service (Port 3002):**
```bash
# Folder: d:\microservices\services\people-service
npx ts-node src/index.ts
```

**Terminal 3 - Case Service (Port 3003):**
```bash
# Folder: d:\microservices\services\case-service
npx ts-node src/index.ts
```

## 📡 API Endpoints

### Auth Service (http://localhost:3101)

#### Register
```bash
POST /api/auth/register
Content-Type: application/json

{
  "email": "test@example.com",
  "password": "password123",
  "full_name": "Test User"
}
```

#### Login
```bash
POST /api/auth/login
Content-Type: application/json

{
  "email": "test@example.com",
  "password": "password123"
}

# Response: { "success": true, "data": { "token": "jwt-token...", "user": {...} } }
```

#### Get Profile
```bash
GET /api/auth/profile
Authorization: Bearer <jwt-token>
```

### People Service (http://localhost:3002)

#### Create Person
```bash
POST /api/people
Authorization: Bearer <jwt-token>
Content-Type: application/json

{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "phone": "0123456789",
  "address": "123 Street"
}
```

#### Get Person (with Redis cache)
```bash
GET /api/people/:id
Authorization: Bearer <jwt-token>

# First call: cache miss → query DB → cache result
# Second call: cache hit → return from Redis instantly
```

#### List People (paginated + cached)
```bash
GET /api/people?page=1&limit=10
Authorization: Bearer <jwt-token>
```

#### Update Person
```bash
PUT /api/people/:id
Authorization: Bearer <jwt-token>
Content-Type: application/json

{
  "phone": "0987654321"
}

# → Invalidates cache
# → Publishes Kafka event: people.updated
```

### Case Service (http://localhost:3003)

#### Create Case
```bash
POST /api/cases
Authorization: Bearer <jwt-token>
Content-Type: application/json

{
  "title": "Bug in login page",
  "description": "Users cannot login with email",
  "status": "open",
  "priority": "high",
  "person_id": 1
}

# → Auto-indexed to Elasticsearch
```

#### Search Cases (Elasticsearch)
```bash
GET /api/cases/search?q=login&status=open&priority=high
Authorization: Bearer <jwt-token>

# Full-text search with filters
```

#### List Cases
```bash
GET /api/cases?page=1&limit=10
Authorization: Bearer <jwt-token>
```

## 🧪 Testing

### Run Unit Tests
```bash
# Folder: d:\microservices
npm test
```

### Run Tests Per Service
```bash
npm run test --workspace=services/auth-service
npm run test --workspace=services/people-service
npm run test --workspace=services/case-service
```

### Check Coverage
```bash
npm test -- --coverage
```

## 📊 Metrics to Monitor

### Authentication/Authorization
- **401 rate**: Number of unauthorized requests
- **Token refresh success rate**: % of successful token refreshes

### People Service
- **Cache hit rate**: Redis cache hit percentage
- **DB QPS**: Queries per second to PostgreSQL
- **Kafka publish success**: % of successfully published events
- **P95 latency**: 95th percentile latency

### Case Service
- **Search latency**: Elasticsearch search time
- **Index lag**: Synchronization delay to ES
- **Consumer lag**: Unprocessed Kafka events
- **Error rate 5xx**: Server error percentage

### Infrastructure
- **Redis**: Eviction rate, memory usage
- **Kafka**: Topic lag, partition rebalance
- **Elasticsearch**: Cluster health, JVM heap
- **PostgreSQL**: Connection pool, slow queries

## 🔄 Event Flow Example

**Scenario**: User creates a new person

1. **Client** → POST `/api/people` (People Service)
2. **People Service**:
   - Insert into `people_db`
   - Invalidate cache
   - **Publish event** `people.created` → Kafka
3. **Kafka** → Broadcast event
4. **Case Service** (consumer):
   - Receive event `people.created`
   - Log or sync data if needed

## 🐛 Common Roadblocks & Solutions

### Roadblock 1: Kafka timeout/lag
- **Cause**: Slow consumer processing, Kafka down
- **Metric**: Consumer lag > 1000
- **Fix**: Scale consumers, optimize processing, implement retry mechanism

### Roadblock 2: High Redis cache miss rate
- **Cause**: TTL too short, high eviction rate
- **Metric**: Cache hit rate < 70%
- **Fix**: Increase TTL, increase memory, review access patterns

### Roadblock 3: Slow Elasticsearch search
- **Cause**: Large index, complex queries
- **Metric**: Search latency P95 > 500ms
- **Fix**: Optimize mapping, filter before search, use pagination

### Roadblock 4: Cross-service calls → timeout
- **Cause**: Network latency, service overload
- **Metric**: High P95 latency, high timeout rate
- **Fix**: Use async (Kafka), implement caching, add circuit breaker

## 📦 CI/CD Pipeline

### Workflow
1. **PR** → Lint + Unit tests
2. **Merge to main** → Integration tests + Build Docker images
3. **Deploy to staging** → Smoke tests
4. **Deploy to production** → Blue-green deployment + Monitor

### Tracked Metrics
- **Build time**: CI pipeline duration
- **Deploy frequency**: Deployments per day
- **Change failure rate**: % of deployments requiring rollback
- **MTTR**: Mean Time To Recovery

## 🛠 Tech Stack

- **Language**: Node.js + TypeScript
- **Framework**: Express.js
- **Database**: PostgreSQL
- **Cache**: Redis
- **Message Queue**: Kafka
- **Search**: Elasticsearch
- **Authentication**: JWT
- **Testing**: Jest
- **CI/CD**: GitHub Actions
- **Containerization**: Docker

## 📁 Project Structure

```
d:\microservices/
├── services/
│   ├── auth-service/        # JWT authentication
│   ├── people-service/      # CRUD + Redis + Kafka
│   └── case-service/        # CRUD + Elasticsearch + Kafka consumer
├── shared/                  # Common utilities, types
├── admin-ui/                # Web UI for testing (NEW!)
│   ├── index.html
│   ├── styles.css
│   ├── app.js
│   └── README.md
├── infrastructure/
│   ├── docker-compose.yml   # Infrastructure setup
│   └── init-db.sql          # Database initialization
├── tests/
│   └── integration/         # Integration tests
├── .github/
│   └── workflows/           # CI/CD pipelines
├── package.json             # Root workspace
├── README.md                # This file
├── ARCHITECTURE.md          # Detailed architecture documentation
├── QUICKSTART.md            # Quick start guide
└── postman_collection.json  # Postman API collection
```

## 🔐 Security Notes

⚠️ **IMPORTANT - CHANGE BEFORE PRODUCTION DEPLOYMENT:**

1. Change `JWT_SECRET` in all services
2. Update database credentials
3. Enable Elasticsearch security (xpack)
4. Configure Kafka SASL authentication
5. Implement API rate limiting
6. Configure CORS properly
7. Never commit environment variables to git

## 📚 Resources

- **Infrastructure Guide**: `infrastructure/README.md`
- **Testing Guide**: `tests/integration/README.md`
- **CI/CD Guide**: `.github/workflows/README.md`
- **Architecture Deep Dive**: `ARCHITECTURE.md`
- **Quick Start Guide**: `QUICKSTART.md`

## 🤝 Contributing

1. Create a new branch: `git checkout -b feature/your-feature`
2. Commit changes: `git commit -m "Add feature"`
3. Push: `git push origin feature/your-feature`
4. Create PR → CI pipeline will run automatically

## 📝 License

MIT

---

## 📋 Quick Commands Reference

### Infrastructure
```bash
npm run docker:up       # Start all infrastructure
npm run docker:down     # Stop all infrastructure
npm run docker:logs     # View logs
```

### Development
```bash
# Folder: d:\microservices\services\auth-service
npx ts-node src/index.ts

# Folder: d:\microservices\services\people-service
npx ts-node src/index.ts

# Folder: d:\microservices\services\case-service
npx ts-node src/index.ts
```

### Build & Test
```bash
npm run build           # Build all services
npm test                # Run all tests
npm test -- --coverage  # Run with coverage
```

---

## ✅ Summary

**3 Services**: Auth, People, Case  
**JWT Authentication**: Stateless, scalable  
**Redis Cache**: Reduce DB load  
**Kafka Events**: Async communication  
**Elasticsearch**: Full-text search  
**PostgreSQL**: Database-per-service pattern  
**Testing**: Unit + integration tests  
**CI/CD**: GitHub Actions, Docker  
**Metrics**: Latency, cache hit rate, consumer lag, error rate
