# NeroChat - Medical AI Assistant Platform

An intelligent medical chatbot platform powered by AI agents, designed to help users find doctors, book appointments, and get medical information through natural language conversations.

## 🏗️ Architecture

This is a microservices-based application with the following components:

### Backend Services

- **Agent Service** (Port 8001) - FastAPI-based agentic AI service for medical conversations and appointment booking
- **Auth Service** (Port 5004) - FastAPI authentication service supporting multiple OAuth providers (Google, Twitter, Microsoft) and email authentication
- **Record Service** (Port 7001) - Node.js service for managing medical records (optional)

### Infrastructure Services

- **Redis** (Port 6379) - Caching and session management
- **ChromaDB** (Port 8000) - Vector database for medical knowledge retrieval (RAG)
- **Nginx** - Reverse proxy and load balancer (optional)

### Frontend

- **React Interface** - Modern React-based chat interface with authentication

## 🚀 Quick Start

### Prerequisites

- Docker & Docker Compose
- Node.js 16+ (for local development)
- Python 3.9+ (for local development)

### Using Docker Compose (Recommended)

1. **Clone the repository**
   ```bash
   git clone https://github.com/amritendunath/nerochat.co.in.git
   cd nerochat.co.in
   ```

2. **Set up environment variables**
   
   Create `.env` files in each service directory:
   
   - `agent/.env` - Agent service configuration
   - `auth/.env` - Authentication service configuration
   
   See the [Environment Variables](#environment-variables) section below for required variables.

3. **Start all services**
   ```bash
   docker-compose up -d
   ```

4. **Access the services**
   - Agent Service API Docs: http://localhost:8001/
   - Auth Service: http://localhost:5004
   - ChromaDB: http://localhost:8000
   - Redis: localhost:6379

### Local Development

#### Agent Service

```bash
cd agent
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
python src/main.py
```

#### Auth Service

```bash
cd auth
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
python src/main.py
```

#### Frontend Interface

```bash
cd ../interface
npm install
npm start
```

## 🔧 Environment Variables

### Agent Service (`agent/.env`)

```env
# LLM Configuration
ANTHROPIC_API_KEY=your_anthropic_key
OPENAI_API_KEY=your_openai_key

# Database
MONGODB_URI=your_mongodb_connection_string

# ChromaDB
CHROMA_HOST=http://chroma:8000

# Redis
REDIS_HOST=redis
REDIS_PORT=6379

# CORS
LOCAL_URL=http://localhost:3000
PROD_URL=https://your-production-domain.com
```

### Auth Service (`auth/.env`)

```env
# JWT
JWT_SECRET=your_jwt_secret_key

# Google OAuth
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret

# Twitter OAuth
TWITTER_CLIENT_ID=your_twitter_client_id
TWITTER_CLIENT_SECRET=your_twitter_client_secret

# Microsoft OAuth
MICROSOFT_CLIENT_ID=your_microsoft_client_id
MICROSOFT_CLIENT_SECRET=your_microsoft_client_secret

# AWS SES (Email Authentication)
AWS_SES=ses
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
AWS_REGION=your_aws_region
SES_SENDER_EMAIL=noreply@nerochat.co.in

# Database
MONGODB_URI=your_mongodb_connection_string
```

## 📦 Service Details

### Agent Service

**Technology Stack:** FastAPI, Python 3.9+, LangChain/LiteLLM

**Key Features:**
- Agentic AI workflow for medical conversations
- Doctor search and appointment booking
- Medical knowledge retrieval using RAG (ChromaDB)
- Multi-provider LLM support (Anthropic, OpenAI)
- Redis-based caching

**API Endpoints:**
- `GET /` - Interactive API documentation (Swagger UI)
- `GET /health` - Health check endpoint
- `POST /api/chat` - Chat endpoint for conversations

### Auth Service

**Technology Stack:** FastAPI, Python 3.9+, Authlib

**Key Features:**
- Multi-provider OAuth (Google, Twitter, Microsoft)
- Email-based authentication with AWS SES
- JWT token generation and validation
- Session management
- User profile management

**API Endpoints:**
- OAuth routes for each provider
- Email authentication endpoints
- Token verification endpoints

### Frontend Interface

**Technology Stack:** React 19, React Router, TailwindCSS

**Key Features:**
- Modern chat interface with markdown support
- Syntax highlighting for code blocks
- Multiple authentication options
- Responsive design
- Real-time chat updates

## 🐳 Docker Configuration

The `docker-compose.yml` file orchestrates all services:

```yaml
services:
  - agent_service (Port 8001)
  - auth_service (Port 5004)
  - redis (Port 6379)
  - chroma (Port 8000)
```

### Building Individual Services

```bash
# Build agent service
docker build -t nerochat-agent ./agent

# Build auth service
docker build -t nerochat-auth ./auth
```

## 🔒 Security Considerations

1. **Environment Variables:** Never commit `.env` files to version control
2. **HTTPS:** Use SSL certificates in production (see commented SSL configuration in `main.py` files)
3. **CORS:** Update `allow_origins` in production to specific domains
4. **JWT Secret:** Use a strong, randomly generated secret key
5. **OAuth Callbacks:** Configure proper redirect URIs in OAuth provider settings

## 📊 Database Schema

The application uses MongoDB for data persistence:

- **Users Collection:** User profiles and authentication data
- **Conversations Collection:** Chat history and context
- **Appointments Collection:** Booking information
- **Doctors Collection:** Medical professional database

## 🧪 Testing

```bash
# Run agent service tests
cd agent
pytest

# Run auth service tests
cd auth
pytest

# Run frontend tests
cd interface
npm test
```

## 📝 API Documentation

Once the services are running, access the interactive API documentation:

- **Agent Service:** http://localhost:8001/
- **Auth Service:** http://localhost:5004/docs

## 🚢 Deployment

### Cloud Deployment Options

1. **Docker-based platforms:** AWS ECS, Google Cloud Run, Azure Container Instances
2. **Kubernetes:** Use the provided Dockerfiles to create K8s deployments
3. **Traditional VPS:** Use docker-compose for deployment

### Environment-specific Configuration

Update the following for production:
- Set proper CORS origins in both services
- Enable SSL/TLS (uncomment SSL configuration in `main.py`)
- Use managed services for Redis and MongoDB
- Configure proper logging and monitoring
- Set up health checks and auto-scaling

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Built with FastAPI, React, and modern AI technologies
- Uses ChromaDB for vector storage and retrieval
- Powered by Anthropic Claude and OpenAI models

## 📧 Support

For issues and questions:
- GitHub Issues: https://github.com/amritendunath/nerochat.co.in/issues
- Email: support@nerochat.co.in

---

**Note:** This is a medical information platform. Always consult with qualified healthcare professionals for medical advice.
