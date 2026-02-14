# Port Migration Guide: Enterprise Architecture

## Overview

This guide explains the migration from development ports to an enterprise-grade port architecture using Nginx as a reverse proxy.

## Architecture Changes

### Before (Development)
```
External Access:
- Agent Service:  localhost:8001
- Auth Service:   localhost:5004
- Record Service: localhost:7001
- ChromaDB:       localhost:8000
- Redis:          localhost:6379
```

### After (Enterprise)
```
External Access:
- HTTP:  localhost:80  (or nerochat.co.in:80)
- HTTPS: localhost:443 (or nerochat.co.in:443)

Internal Routing (via Nginx):
- /api/agent/    → agent_service:8080
- /api/auth/     → auth_service:8080
- /api/records/  → record_service:8080

Internal Services (not exposed):
- ChromaDB: chroma:8000
- Redis:    redis:6379
```

## Benefits of Enterprise Architecture

### 🔒 Security
- **Single Entry Point**: Only ports 80/443 exposed to the internet
- **Service Isolation**: Backend services not directly accessible
- **SSL Termination**: Centralized HTTPS handling
- **Rate Limiting**: Built-in DDoS protection

### 🚀 Performance
- **Load Balancing**: Distribute traffic across multiple instances
- **Gzip Compression**: Reduce bandwidth usage
- **Connection Pooling**: Efficient resource utilization
- **Caching**: Static content caching at proxy level

### 📊 Monitoring & Operations
- **Centralized Logging**: All requests logged at nginx
- **Health Checks**: Automatic failover for unhealthy services
- **Graceful Degradation**: Service-level fault tolerance
- **Easy Scaling**: Add/remove service instances without client changes

### 🔧 Development & Deployment
- **Environment Parity**: Same architecture for dev/staging/prod
- **Zero Downtime Deploys**: Rolling updates per service
- **Path-based Routing**: Easy API versioning (/api/v1, /api/v2)
- **Standard Ports**: No firewall configuration needed

## Migration Steps

### Step 1: Update Service Code

Each service needs to support the `PORT` environment variable.

#### Agent Service (FastAPI)
File: `agent/src/main.py`

```python
import os

if __name__ == "__main__":
    port = int(os.environ.get("PORT", 8001))  # Default to 8001 for backward compatibility
    uvicorn.run("main:app", host="0.0.0.0", port=port, reload=True)
```

#### Auth Service (FastAPI)
File: `auth/src/main.py`

```python
import os

if __name__ == "__main__":
    port = int(os.environ.get("PORT", 5004))  # Default to 5004 for backward compatibility
    uvicorn.run(app, host="0.0.0.0", port=port)
```

#### Record Service (Node.js)
File: `record/src/server.js`

```javascript
const PORT = process.env.PORT || 7001;
app.listen(PORT, '0.0.0.0', () => {
    console.log(`Server running on port ${PORT}`);
});
```

### Step 2: Update Frontend API Endpoints

Update your frontend to use the new API paths.

#### Before:
```javascript
const AGENT_API = 'http://localhost:8001';
const AUTH_API = 'http://localhost:5004';
```

#### After:
```javascript
const API_BASE = process.env.REACT_APP_API_URL || 'http://localhost';
const AGENT_API = `${API_BASE}/api/agent`;
const AUTH_API = `${API_BASE}/api/auth`;
```

### Step 3: Choose Your Deployment Strategy

#### Option A: Full Enterprise (Recommended for Production)

```bash
# Use the enterprise docker-compose
docker-compose -f docker-compose.enterprise.yml up -d

# Access services
curl http://localhost/api/agent/health
curl http://localhost/api/auth/health
```

**Pros:**
- Production-ready
- Best security
- Scalable
- Standard ports

**Cons:**
- Slightly more complex
- Requires nginx configuration

#### Option B: Hybrid (Good for Development)

Keep current setup but use standardized ports:

```yaml
# docker-compose.yml
services:
  agent_service:
    ports:
      - "8080:8080"  # Standardized
  auth_service:
    ports:
      - "8081:8080"  # Different host port, same container port
  record_service:
    ports:
      - "8082:8080"
```

**Pros:**
- Easy to debug
- Direct service access
- Familiar workflow

**Cons:**
- Multiple ports to manage
- Not production-ready
- Security concerns

#### Option C: Keep Current (Not Recommended)

No changes needed, but you miss out on enterprise benefits.

### Step 4: Update Environment Variables

Create `.env` files with PORT configuration:

```bash
# agent/.env
PORT=8080

# auth/.env
PORT=8080

# record/.env
PORT=8080
```

### Step 5: Test the Migration

```bash
# Start services
docker-compose -f docker-compose.enterprise.yml up -d

# Test health endpoints
curl http://localhost/health
curl http://localhost/api/agent/
curl http://localhost/api/auth/health

# Check logs
docker-compose -f docker-compose.enterprise.yml logs -f nginx
docker-compose -f docker-compose.enterprise.yml logs -f agent_service
```

## Rollback Plan

If issues occur, you can quickly rollback:

```bash
# Stop enterprise setup
docker-compose -f docker-compose.enterprise.yml down

# Start original setup
docker-compose up -d
```

## Production Checklist

Before deploying to production:

- [ ] Configure SSL certificates in `nginx/ssl/`
- [ ] Uncomment HTTPS server block in `nginx.conf`
- [ ] Update `server_name` to your actual domain
- [ ] Enable HTTP to HTTPS redirect
- [ ] Configure rate limiting based on your traffic
- [ ] Set up monitoring and alerting
- [ ] Test failover scenarios
- [ ] Configure log rotation
- [ ] Set up automated backups for volumes
- [ ] Review and adjust timeout values
- [ ] Enable security headers
- [ ] Configure CORS properly for your domain

## Port Reference Table

| Service | Dev Port | Enterprise Internal | External Access |
|---------|----------|---------------------|-----------------|
| Nginx | - | - | 80, 443 |
| Agent Service | 8001 | 8080 | /api/agent/ |
| Auth Service | 5004 | 8080 | /api/auth/ |
| Record Service | 7001 | 8080 | /api/records/ |
| ChromaDB | 8000 | 8000 | Internal only |
| Redis | 6379 | 6379 | Internal only |

## API Endpoint Mapping

| Old Endpoint | New Endpoint |
|--------------|--------------|
| `http://localhost:8001/` | `http://localhost/api/agent/` |
| `http://localhost:8001/health` | `http://localhost/api/agent/health` |
| `http://localhost:5004/login/google` | `http://localhost/api/auth/login/google` |
| `http://localhost:7001/records` | `http://localhost/api/records/records` |

## Troubleshooting

### Services can't connect to each other
- Ensure all services are on the same Docker network (`app_network`)
- Use service names (e.g., `redis`, `chroma`) instead of `localhost`

### 502 Bad Gateway
- Check if backend service is running: `docker-compose ps`
- Check backend service logs: `docker-compose logs agent_service`
- Verify PORT environment variable is set correctly

### CORS errors
- Update CORS configuration in each service to allow nginx proxy
- Add `X-Forwarded-*` headers handling

### SSL certificate errors
- Ensure certificates are in `nginx/ssl/` directory
- Verify certificate paths in nginx.conf
- Check certificate validity: `openssl x509 -in cert.pem -text -noout`

## Next Steps

1. **Test locally** with `docker-compose.enterprise.yml`
2. **Update frontend** to use new API paths
3. **Deploy to staging** environment
4. **Load test** the new architecture
5. **Deploy to production** with SSL enabled

## Questions?

- Review the nginx.conf for detailed routing rules
- Check docker-compose.enterprise.yml for service configuration
- Refer to nginx documentation: https://nginx.org/en/docs/
