# neroxchat.com - Medical AI Assistant Platform

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/Docker-Ready-blue.svg)](https://www.docker.com/)
[![Production](https://img.shields.io/badge/Production-Ready-green.svg)](https://nerochat.co.in)

> An enterprise-grade intelligent medical chatbot platform powered by AI agents, designed to help users find doctors, book appointments, and get medical information through natural language conversations.

## 📑 Table of Contents

- [Architecture](#-architecture)
- [Quick Start](#-quick-start)
- [Deployment Options](#-deployment-options)
- [Environment Configuration](#-environment-configuration)
- [API Documentation](#-api-documentation)
- [Security](#-security)
- [Monitoring & Operations](#-monitoring--operations)
- [Contributing](#-contributing)

---

## 🏗️ Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Nginx Reverse Proxy                   │
│              (Port 80/HTTP, 443/HTTPS)                   │
│         SSL Termination | Load Balancing | Rate Limiting │
└─────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Agent Service│    │ Auth Service │    │Record Service│
│   (Port 8080)│    │  (Port 8080) │    │  (Port 8080) │
│   FastAPI    │    │   FastAPI    │    │   Node.js    │
└──────────────┘    └──────────────┘    └──────────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   ChromaDB   │    │    Redis     │    │   MongoDB    │
│  (Port 8000) │    │  (Port 6379) │    │ (Port 27017) │
│ Vector Store │    │    Cache     │    │   Database   │
└──────────────┘    └──────────────┘    └──────────────┘
```

### Microservices

| Service | Technology | Port (Internal) | External Access | Purpose |
|---------|-----------|-----------------|-----------------|---------|
| **Nginx** | Nginx 1.29 | - | 80, 443 | Reverse proxy, SSL termination, load balancing |
| **Agent Service** | FastAPI + Python 3.9 | 8080 | `/api/agent/*` | AI-powered medical conversations, appointment booking |
| **Auth Service** | FastAPI + Python 3.9 | 8080 | `/api/auth/*` | Multi-provider OAuth, JWT authentication |
| **Record Service** | Node.js + Express | 8080 | `/api/records/*` | Medical records management |
| **ChromaDB** | ChromaDB | 8000 | Internal only | Vector database for RAG (Retrieval-Augmented Generation) |
| **Redis** | Redis 7 | 6379 | Internal only | Session management, caching |
| **MongoDB** | MongoDB 6 | 27017 | Internal only | Primary data store |

### Frontend

- **Technology:** React 19, React Router, TailwindCSS
- **Deployment:** Vercel / Netlify / Static hosting
- **Repository:** Separate frontend repository

---

## 🚀 Quick Start

### Prerequisites

- **Docker** 20.10+ and **Docker Compose** 2.0+
- **Git** for version control
- **SSL Certificates** (for production HTTPS)

### Production Deployment (Recommended)

```bash
# 1. Clone the repository
git clone https://github.com/amritendunath/nerochat.co.in.git
cd nerochat.co.in/services

# 2. Configure environment variables
cp agent/.env.example agent/.env
cp auth/.env.example auth/.env
# Edit .env files with your credentials

# 3. Set up SSL certificates (production)
mkdir -p nginx/ssl
# Place your cert.pem and key.pem in nginx/ssl/

# 4. Deploy with enterprise configuration
docker-compose -f docker-compose.enterprise.yml up -d

# 5. Verify deployment
curl https://nerochat.co.in/health
curl https://nerochat.co.in/api/agent/health
curl https://nerochat.co.in/api/auth/health
```

### Development Setup

```bash
# 1. Clone and navigate
git clone https://github.com/amritendunath/nerochat.co.in.git
cd nerochat.co.in/services

# 2. Configure environment
cp agent/.env.example agent/.env
cp auth/.env.example auth/.env

# 3. Start development environment
docker-compose up -d

# 4. Access services
# Agent Service: http://localhost:8001
# Auth Service:  http://localhost:5004
```

### Local Development (Without Docker)

#### Agent Service

```bash
cd agent
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
python src/main.py
```

#### Auth Service

```bash
cd auth
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
python src/main.py
```

---

## 🌐 Deployment Options

### Option 1: Enterprise Production (Recommended)

**Use Case:** Production deployment with SSL, load balancing, and security

```bash
docker-compose -f docker-compose.enterprise.yml up -d
```

**Features:**
- ✅ Nginx reverse proxy with SSL termination
- ✅ Path-based routing (`/api/agent`, `/api/auth`, `/api/records`)
- ✅ Rate limiting and DDoS protection
- ✅ Automatic health checks and failover
- ✅ Centralized logging
- ✅ Gzip compression
- ✅ Production-ready security headers

**Access:**
```
https://nerochat.co.in/api/agent/health
https://nerochat.co.in/api/auth/login/google
https://nerochat.co.in/api/records/
```

### Option 2: Development Mode

**Use Case:** Local development and testing

```bash
docker-compose up -d
```

**Features:**
- Direct service access on individual ports
- Hot-reload enabled
- Debug logging
- No SSL required

**Access:**
```
http://localhost:8001  # Agent Service
http://localhost:5004  # Auth Service
http://localhost:7001  # Record Service
```

### Option 3: Cloud Platforms

#### AWS ECS / Fargate

```bash
# Use docker-compose.enterprise.yml as base
# Configure ECS task definitions
# Set up Application Load Balancer
# Configure Route53 for DNS
```

#### Google Cloud Run

```bash
# Build and push images
docker build -t gcr.io/PROJECT_ID/nerochat-agent ./agent
docker push gcr.io/PROJECT_ID/nerochat-agent

# Deploy
gcloud run deploy nerochat-agent \
  --image gcr.io/PROJECT_ID/nerochat-agent \
  --platform managed \
  --region us-central1
```

#### Azure Container Instances

```bash
# Create resource group
az group create --name nerochat-rg --location eastus

# Deploy container
az container create \
  --resource-group nerochat-rg \
  --name nerochat-agent \
  --image nerochat-agent:latest \
  --dns-name-label nerochat \
  --ports 80 443
```

#### Kubernetes (K8s)

```bash
# Apply configurations
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmaps.yaml
kubectl apply -f k8s/secrets.yaml
kubectl apply -f k8s/deployments.yaml
kubectl apply -f k8s/services.yaml
kubectl apply -f k8s/ingress.yaml
```

---

## 🔧 Environment Configuration

### Agent Service (`agent/.env`)

```env
# ============================================
# LLM Provider Configuration
# ============================================
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxx
OPENAI_API_KEY=sk-xxxxxxxxxxxxx
GOOGLE_API_KEY=xxxxxxxxxxxxx

# Default LLM provider (anthropic, openai, google)
DEFAULT_LLM_PROVIDER=anthropic
DEFAULT_MODEL=claude-3-5-sonnet-20241022

# ============================================
# Database Configuration
# ============================================
MONGODB_URI=mongodb://mongo:27017/nerochat
# For MongoDB Atlas:
# MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/nerochat

# ============================================
# Vector Database (ChromaDB)
# ============================================
CHROMA_HOST=http://chroma:8000
CHROMA_COLLECTION=medical_knowledge

# ============================================
# Cache Configuration (Redis)
# ============================================
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_DB=0
REDIS_PASSWORD=  # Leave empty for no password

# ============================================
# Application Configuration
# ============================================
PORT=8080  # Internal port (8001 for dev mode)
ENVIRONMENT=production  # development, staging, production

# ============================================
# CORS Configuration
# ============================================
LOCAL_URL=http://localhost:3000
PROD_URL=https://nerochat.co.in
ALLOWED_ORIGINS=https://nerochat.co.in,https://www.nerochat.co.in

# ============================================
# Logging
# ============================================
LOG_LEVEL=INFO  # DEBUG, INFO, WARNING, ERROR, CRITICAL
LOG_FORMAT=json  # json or text
```

### Auth Service (`auth/.env`)

```env
# ============================================
# JWT Configuration
# ============================================
JWT_SECRET=your-super-secret-jwt-key-min-32-chars
JWT_ALGORITHM=HS256
JWT_EXPIRATION_HOURS=24

# ============================================
# Google OAuth
# ============================================
GOOGLE_CLIENT_ID=xxxxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=GOCSPX-xxxxxxxxxxxxx
GOOGLE_REDIRECT_URI=https://nerochat.co.in/api/auth/callback/google

# ============================================
# Twitter OAuth
# ============================================
TWITTER_CLIENT_ID=xxxxxxxxxxxxx
TWITTER_CLIENT_SECRET=xxxxxxxxxxxxx
TWITTER_REDIRECT_URI=https://nerochat.co.in/api/auth/callback/twitter

# ============================================
# Microsoft OAuth
# ============================================
MICROSOFT_CLIENT_ID=xxxxxxxxxxxxx
MICROSOFT_CLIENT_SECRET=xxxxxxxxxxxxx
MICROSOFT_REDIRECT_URI=https://nerochat.co.in/api/auth/callback/microsoft

# ============================================
# AWS SES (Email Authentication)
# ============================================
AWS_SES=ses
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxx
AWS_REGION=us-east-1
SES_SENDER_EMAIL=noreply@nerochat.co.in
SES_SENDER_NAME=NeroChat

# ============================================
# Database
# ============================================
MONGODB_URI=mongodb://mongo:27017/nerochat

# ============================================
# Application Configuration
# ============================================
PORT=8080  # Internal port (5004 for dev mode)
ENVIRONMENT=production
```

### Nginx Configuration

SSL certificates should be placed in `nginx/ssl/`:
- `cert.pem` - SSL certificate
- `key.pem` - Private key

For Let's Encrypt:
```bash
certbot certonly --standalone -d nerochat.co.in -d www.nerochat.co.in
cp /etc/letsencrypt/live/nerochat.co.in/fullchain.pem nginx/ssl/cert.pem
cp /etc/letsencrypt/live/nerochat.co.in/privkey.pem nginx/ssl/key.pem
```

---

## 📝 API Documentation

### Interactive Documentation

Once deployed, access Swagger UI documentation:

- **Production:** https://nerochat.co.in/api/agent/docs
- **Development:** http://localhost:8001/docs

### API Endpoints

#### Agent Service (`/api/agent/`)

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/health` | Health check | No |
| POST | `/chat` | Send chat message | Yes |
| GET | `/doctors` | Search doctors | Yes |
| POST | `/appointments` | Book appointment | Yes |
| GET | `/appointments/{id}` | Get appointment details | Yes |

#### Auth Service (`/api/auth/`)

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/health` | Health check | No |
| GET | `/login/google` | Google OAuth login | No |
| GET | `/login/twitter` | Twitter OAuth login | No |
| GET | `/login/microsoft` | Microsoft OAuth login | No |
| POST | `/login/email` | Email login | No |
| POST | `/verify-token` | Verify JWT token | Yes |
| POST | `/refresh` | Refresh access token | Yes |
| POST | `/logout` | Logout user | Yes |

#### Record Service (`/api/records/`)

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/health` | Health check | No |
| GET | `/records` | Get user records | Yes |
| POST | `/records` | Create record | Yes |
| PUT | `/records/{id}` | Update record | Yes |
| DELETE | `/records/{id}` | Delete record | Yes |

---

## 🔒 Security

### Production Security Checklist

- [x] **SSL/TLS Encryption:** All traffic encrypted via HTTPS
- [x] **Environment Variables:** Sensitive data in `.env` files (not committed)
- [x] **CORS Configuration:** Restricted to specific domains
- [x] **Rate Limiting:** 10 req/s for API, 5 req/s for auth
- [x] **JWT Authentication:** Secure token-based auth with expiration
- [x] **OAuth 2.0:** Industry-standard OAuth providers
- [x] **Security Headers:** HSTS, X-Frame-Options, CSP, etc.
- [x] **Input Validation:** All inputs sanitized and validated
- [x] **SQL Injection Protection:** Using ORMs and parameterized queries
- [x] **DDoS Protection:** Nginx rate limiting and connection limits
- [x] **Health Checks:** Automatic failover for unhealthy services
- [x] **Secrets Management:** Use AWS Secrets Manager / HashiCorp Vault in production

### Security Headers (Nginx)

```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Content-Security-Policy "default-src 'self'" always;
```

### HIPAA Compliance Considerations

⚠️ **Important:** For handling Protected Health Information (PHI):

1. **Encryption at Rest:** Enable MongoDB encryption
2. **Audit Logging:** Log all data access
3. **Access Controls:** Implement role-based access control (RBAC)
4. **Data Retention:** Configure automatic data purging
5. **Business Associate Agreement (BAA):** Required with cloud providers

---

## 📊 Monitoring & Operations

### Health Checks

```bash
# Overall system health
curl https://nerochat.co.in/health

# Individual services
curl https://nerochat.co.in/api/agent/health
curl https://nerochat.co.in/api/auth/health
curl https://nerochat.co.in/api/records/health
```

### Logging

```bash
# View all logs
docker-compose -f docker-compose.enterprise.yml logs -f

# Service-specific logs
docker-compose -f docker-compose.enterprise.yml logs -f agent_service
docker-compose -f docker-compose.enterprise.yml logs -f nginx

# Nginx access logs
docker-compose -f docker-compose.enterprise.yml exec nginx tail -f /var/log/nginx/access.log
```

### Metrics & Monitoring

Recommended monitoring stack:

- **Prometheus:** Metrics collection
- **Grafana:** Visualization dashboards
- **ELK Stack:** Centralized logging (Elasticsearch, Logstash, Kibana)
- **Sentry:** Error tracking
- **DataDog / New Relic:** APM (Application Performance Monitoring)

### Backup & Disaster Recovery

```bash
# MongoDB backup
docker-compose exec mongo mongodump --out /backup

# Redis backup
docker-compose exec redis redis-cli BGSAVE

# ChromaDB backup
docker-compose exec chroma tar -czf /backup/chroma.tar.gz /chroma/chroma
```

---

## 🧪 Testing

### Unit Tests

```bash
# Agent service
cd agent
pytest tests/ -v --cov=src

# Auth service
cd auth
pytest tests/ -v --cov=src
```

### Integration Tests

```bash
# Start test environment
docker-compose -f docker-compose.test.yml up -d

# Run integration tests
pytest tests/integration/ -v
```

### Load Testing

```bash
# Using Apache Bench
ab -n 1000 -c 10 https://nerochat.co.in/api/agent/health

# Using k6
k6 run tests/load/chat-scenario.js
```

---

## 🚢 CI/CD Pipeline

### GitHub Actions Example

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build and push Docker images
        run: |
          docker build -t nerochat-agent:latest ./agent
          docker push nerochat-agent:latest
      - name: Deploy to production
        run: |
          ssh production "cd /app && docker-compose pull && docker-compose up -d"
```

---

## 📈 Performance Optimization

### Caching Strategy

- **Redis:** Session data, frequently accessed data (TTL: 1 hour)
- **Nginx:** Static content caching
- **CDN:** Frontend assets via CloudFlare/CloudFront

### Database Optimization

- **Indexes:** Created on frequently queried fields
- **Connection Pooling:** Configured in MongoDB driver
- **Query Optimization:** Analyzed with MongoDB profiler

### Load Balancing

- **Nginx:** Round-robin load balancing
- **Horizontal Scaling:** Multiple instances of each service
- **Auto-scaling:** Based on CPU/memory metrics

---

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md).

### Development Workflow

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Make your changes
4. Run tests: `pytest tests/`
5. Commit: `git commit -m 'Add amazing feature'`
6. Push: `git push origin feature/amazing-feature`
7. Open a Pull Request

### Code Standards

- **Python:** Follow PEP 8, use Black formatter
- **JavaScript:** Follow Airbnb style guide, use Prettier
- **Commits:** Use conventional commits (feat, fix, docs, etc.)
- **Tests:** Maintain >80% code coverage

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- **FastAPI:** Modern, fast web framework for building APIs
- **React:** UI library for building user interfaces
- **ChromaDB:** Open-source vector database
- **Anthropic Claude:** Advanced AI language model
- **OpenAI:** GPT models for natural language processing

---

## 📧 Support & Contact

- **Documentation:** https://docs.nerochat.co.in
- **GitHub Issues:** https://github.com/amritendunath/nerochat.co.in/issues
- **Email:** support@nerochat.co.in
- **Discord Community:** https://discord.gg/nerochat

---

## ⚠️ Medical Disclaimer

**This is a medical information platform for educational and informational purposes only.** 

- Not a substitute for professional medical advice
- Always consult qualified healthcare professionals
- Do not use for medical emergencies
- Call your local emergency number (911, 112, etc.) for emergencies

---

## 🗺️ Roadmap

- [x] Multi-provider OAuth authentication
- [x] AI-powered medical conversations
- [x] Doctor search and appointment booking
- [x] Enterprise-grade deployment
- [ ] Multi-language support (Spanish, Hindi, Mandarin)
- [ ] Voice-based interactions
- [ ] Mobile applications (iOS, Android)
- [ ] Telemedicine video consultations
- [ ] Electronic Health Records (EHR) integration
- [ ] FHIR API compliance
- [ ] AI-powered symptom checker
- [ ] Prescription management
- [ ] Insurance integration

---

**Built with ❤️ by the NeroChat Team**

**Version:** 6.0.0 | **Last Updated:** February 2026
