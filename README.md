# Complex Kubernetes Multi-Container Application

## 📋 Project Overview

This is a production-grade multi-container application deployed on Kubernetes that demonstrates microservices architecture, container orchestration, and modern DevOps practices. The application calculates Fibonacci numbers using a distributed architecture with multiple services working together.

## 🏗️ Architecture Overview

The application consists of five main components working in a microservices architecture:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Browser   │────▶│   NGINX     │────▶│   React     │
└─────────────┘     │   Ingress   │     │   Client    │
                    └─────────────┘     └─────────────┘
                            │                    │
                            ▼                    ▼
                    ┌─────────────┐     ┌─────────────┐
                    │   Express   │◀────│    Redis    │
                    │   Server    │     │    Cache    │
                    └─────────────┘     └─────────────┘
                            │                    ▲
                            ▼                    │
                    ┌─────────────┐     ┌─────────────┐
                    │  PostgreSQL │     │   Worker    │
                    │   Database  │     │   Service   │
                    └─────────────┘     └─────────────┘
```

### Component Details

#### 1. **Client (React Frontend)**
- **Location**: `/client`
- **Technology**: React 16.4.2 with React Router
- **Purpose**: Provides the user interface for inputting Fibonacci indices and displaying results
- **Key Features**:
  - Single-page application with routing
  - Real-time updates of calculated values
  - Display of previously calculated indices
- **Port**: 3000
- **Build Process**: Multi-stage Docker build with nginx serving static files

#### 2. **Server (Express API)**
- **Location**: `/server`
- **Technology**: Node.js with Express 4.16.3
- **Purpose**: API backend that handles client requests and coordinates data flow
- **Key Features**:
  - RESTful API endpoints
  - Database connection management (PostgreSQL)
  - Cache management (Redis)
  - Message publishing for worker tasks
- **Port**: 5000
- **Endpoints**:
  - `GET /`: Health check
  - `GET /values/all`: Retrieve all calculated indices from PostgreSQL
  - `GET /values/current`: Get current values from Redis cache
  - `POST /values`: Submit new index for calculation

#### 3. **Worker (Background Processor)**
- **Location**: `/worker`
- **Technology**: Node.js
- **Purpose**: Asynchronous calculation of Fibonacci values
- **Key Features**:
  - Subscribes to Redis pub/sub channel
  - Performs recursive Fibonacci calculations
  - Stores results back to Redis cache
- **Algorithm**: Recursive implementation (intentionally slow for demonstration)

#### 4. **Redis**
- **Purpose**: 
  - In-memory data cache for fast access
  - Pub/Sub messaging between server and worker
- **Configuration**:
  - Single replica deployment
  - ClusterIP service for internal access
  - Port: 6379

#### 5. **PostgreSQL**
- **Purpose**: Persistent storage for all submitted indices
- **Configuration**:
  - Single replica with persistent volume claim (2Gi)
  - ClusterIP service for internal access
  - Port: 5432
  - Password managed via Kubernetes Secret

## 🚀 Kubernetes Configuration

### Deployments

| Component | Replicas | Image | Service Type |
|-----------|----------|-------|--------------|
| Client | 3 | vitorspk/multi-client | ClusterIP |
| Server | 3 | vitorspk/multi-server | ClusterIP |
| Worker | 1 | vitorspk/multi-worker | None |
| Redis | 1 | redis | ClusterIP |
| PostgreSQL | 1 | postgres | ClusterIP |

### Services

- **client-cluster-ip-service**: Internal service for React client (port 3000)
- **server-cluster-ip-service**: Internal service for Express server (port 5000)
- **redis-cluster-ip-service**: Internal service for Redis (port 6379)
- **postgres-cluster-ip-service**: Internal service for PostgreSQL (port 5432)

### Ingress Configuration

The application uses NGINX Ingress Controller to route external traffic:
- `/` → Client service (React app)
- `/api/` → Server service (Express API)

### Persistent Storage

- **PostgreSQL**: Uses PersistentVolumeClaim with 2Gi storage
- **Mount Path**: `/var/lib/postgresql/data`
- **Access Mode**: ReadWriteOnce

## 🔄 Data Flow

1. **User Input**: User enters a Fibonacci index in the React frontend
2. **API Request**: Frontend sends POST request to `/api/values`
3. **Validation**: Server validates the index (must be ≤ 40)
4. **Storage**: Server stores index in PostgreSQL for persistence
5. **Cache**: Server sets placeholder in Redis cache
6. **Message**: Server publishes index to Redis pub/sub channel
7. **Calculation**: Worker receives message and calculates Fibonacci value
8. **Update**: Worker updates Redis cache with calculated value
9. **Display**: Frontend fetches and displays results from both cache and database

## 🛠️ Technology Stack

### Frontend
- React 16.4.2
- React Router DOM 4.3.1
- Axios 0.18.0 (HTTP client)
- Nginx (production server)

### Backend
- Node.js
- Express 4.16.3
- PostgreSQL client (pg 7.4.3)
- Redis client 2.8.0
- Body-parser
- CORS middleware

### Infrastructure
- Kubernetes
- Docker (multi-stage builds)
- NGINX Ingress Controller
- GitHub Actions (CI/CD)

## 📦 Docker Configuration

### Multi-Stage Build (Client)
```dockerfile
# Build stage: Node Alpine for building React app
# Production stage: Nginx for serving static files
```

### Standard Build (Server & Worker)
```dockerfile
# Node Alpine base image
# npm install and start scripts
```

## 🔐 Security Considerations

1. **Secrets Management**: PostgreSQL password stored as Kubernetes Secret
2. **Network Policies**: Services use ClusterIP for internal-only access
3. **Input Validation**: Server validates index range (≤ 40) to prevent abuse
4. **CORS Configuration**: Enabled for cross-origin requests

## 🚦 CI/CD Pipeline

### GitHub Actions Workflow
- **Trigger**: Comments with `@claude` mention
- **Automated Code Reviews**: Using Claude AI assistant
- **Permissions**: 
  - Read access to contents, pull requests, issues
  - Action read permissions for CI results

## 🏃 Running the Application

### Prerequisites
- Kubernetes cluster (local or cloud)
- kubectl configured
- Docker images available in registry

### Deployment Steps

1. **Create Secret for PostgreSQL**:
```bash
kubectl create secret generic pgpassword --from-literal PGPASSWORD=your_password
```

2. **Apply Kubernetes Configurations**:
```bash
kubectl apply -f k8s/
```

3. **Install NGINX Ingress Controller** (if not already installed):
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
```

4. **Verify Deployments**:
```bash
kubectl get deployments
kubectl get services
kubectl get pods
```

## 📊 Monitoring & Debugging

### Useful Commands
```bash
# Check pod logs
kubectl logs <pod-name>

# Describe deployment
kubectl describe deployment <deployment-name>

# Access PostgreSQL
kubectl exec -it <postgres-pod> -- psql -U postgres

# Access Redis CLI
kubectl exec -it <redis-pod> -- redis-cli
```

## 🔍 Application Features

### Implemented
- ✅ Fibonacci calculation service
- ✅ Persistent storage of all submitted indices
- ✅ In-memory caching for fast retrieval
- ✅ Horizontal scaling for client and server
- ✅ Load balancing via Kubernetes services
- ✅ Ingress routing for external access

### Architecture Patterns
- **Microservices**: Separated concerns across multiple containers
- **Event-Driven**: Pub/Sub pattern for async processing
- **Cache-Aside**: Redis cache with PostgreSQL persistence
- **Multi-Tier**: Presentation, Application, and Data layers

## 📈 Performance Considerations

1. **Client Scaling**: 3 replicas for high availability
2. **Server Scaling**: 3 replicas for API load distribution
3. **Worker Limitation**: Single replica (stateful calculations)
4. **Cache Strategy**: Redis for immediate reads, PostgreSQL for persistence
5. **Index Limitation**: Max index of 40 to prevent long calculations

## 🔧 Development Notes

### Local Development
Each service can be developed independently:
- Client: `npm start` (port 3000)
- Server: `npm run dev` (port 5000)
- Worker: `npm run dev`

### Environment Variables
Server and Worker require:
- `REDIS_HOST`, `REDIS_PORT`
- `PGUSER`, `PGHOST`, `PGPORT`, `PGDATABASE`, `PGPASSWORD`

## 📝 Known Limitations

1. **Fibonacci Algorithm**: Uses recursive approach (intentionally inefficient for demonstration)
2. **Index Limit**: Maximum value of 40 to prevent server overload
3. **Worker Scaling**: Limited to single instance due to calculation state
4. **Database Backup**: No automated backup strategy implemented

## 🤝 Contributing

This project uses GitHub Actions with Claude AI for automated code reviews. To contribute:
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request
5. Tag `@claude` in comments for AI-assisted code review

## 📄 License

This project is part of a Kubernetes learning exercise and demonstration of microservices architecture.

---

**Generated with Claude AI Assistant** - This comprehensive analysis provides a complete understanding of the application architecture, components, and deployment strategy.