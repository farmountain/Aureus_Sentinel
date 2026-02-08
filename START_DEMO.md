# Start Complete Aureus-Sentinel Demo

## Environment Support
✅ Docker  
✅ Podman  
✅ Kubernetes

## Current Status ✅
- Bridge: Running on port 3000 (permission fixes applied)
- PostgreSQL: Ready
- Redis: Ready

## Option 1: Docker Environment

### Build and Start Everything (15-20 min)
```powershell
docker-compose -f docker-compose-full.yml up --build -d
```

## Option 2: Podman Environment

### Prerequisites
```powershell
# Ensure Podman machine is running
podman machine list

# If not running, start it
podman machine start

# Verify Podman is working
podman ps
```

### Build Bridge Service
```powershell
# Build Bridge image with Podman
cd Aureus-Sentinel
podman build -t aureus-bridge:latest -f Dockerfile .
```

### Start Bridge with Podman
```powershell
# Create network
podman network create aureus-network

# Start PostgreSQL
podman run -d --name aureus-postgres \
  --network aureus-network \
  -e POSTGRES_USER=aureus \
  -e POSTGRES_PASSWORD=dev_password \
  -e POSTGRES_DB=aureus \
  -p 5432:5432 \
  postgres:15-alpine

# Start Redis
podman run -d --name aureus-redis \
  --network aureus-network \
  -p 6379:6379 \
  redis:7-alpine

# Start Bridge
podman run -d --name aureus-bridge \
  --network aureus-network \
  -e PORT=3000 \
  -e NODE_ENV=development \
  -e EVENT_STORE_PATH=/app/events \
  -e AUDIT_LOG_PATH=/app/audit \
  -e POSTGRES_HOST=aureus-postgres \
  -e REDIS_HOST=aureus-redis \
  -p 3000:3000 \
  aureus-bridge:latest

# Check status
podman ps
```

### Option 3: Manual Build Services Individually
```powershell
# Build Aureus OS (5-10 minutes)
docker-compose -f docker-compose-full.yml build aureus-os

# Build OpenClaw (2-5 minutes)
docker-compose -f docker-compose-full.yml build openclaw

# Start all services
docker-compose -f docker-compose-full.yml up -d
```

## Verify All Services Running
```powershell
docker-compose -f docker-compose-full.yml ps
```

Expected output:
- ✅ bridge: Up (healthy)
- ✅ aureus-console: Up (healthy)
- ✅ openclaw: Up (healthy)
- ✅ postgres: Up (healthy)
- ✅ redis: Up (healthy)
- ✅ vector-db: Up
- ✅ prometheus: Up
- ✅ grafana: Up

## Test Endpoints

### Bridge Health
```powershell
curl http://localhost:3000/health
# Expected: {"status":"ok"}
```

### Aureus Console
```powershell
curl http://localhost:3002/health
```

### OpenClaw Gateway
```powershell
curl http://localhost:8080/health
```

## Test End-to-End Integration

### 1. Test Bridge Signing
```powershell
$body = @{
    plan = @{
        action = "test_action"
        parameters = @{}
    }
    context = @{}
} | ConvertTo-Json

curl -X POST http://localhost:3000/sign `
  -H "Content-Type: application/json" `
  -H "x-api-key: dev_bridge_key_change_in_prod" `
  -d $body
```

Expected: Signed approval with signature

### 2. Send Intent via OpenClaw
```powershell
$intent = @{
    intent = "Query weather in San Francisco"
    channel = "test"
} | ConvertTo-Json

curl -X POST http://localhost:8080/api/intent `
  -H "Content-Type: application/json" `
  -d $intent
```

## View Dashboards

- **Grafana**: http://localhost:3001 (admin/admin)
- **Prometheus**: http://localhost:9090
- **PGAdmin**: http://localhost:5050 (admin@aureus.local/admin)

## Logs

View logs for any service:
```powershell
# Bridge
docker-compose -f docker-compose-full.yml logs -f bridge

# Aureus OS
docker-compose -f docker-compose-full.yml logs -f aureus-os

# OpenClaw
docker-compose -f docker-compose-full.yml logs -f openclaw

# All services
docker-compose -f docker-compose-full.yml logs -f
```

## Troubleshooting

### If services fail to start
```powershell
# Check logs for errors
docker-compose -f docker-compose-full.yml logs --tail=50 <service-name>

# Restart specific service
docker-compose -f docker-compose-full.yml restart <service-name>

# Rebuild and restart
docker-compose -f docker-compose-full.yml up --build -d <service-name>
```

### Clean restart
```powershell
docker-compose -f docker-compose-full.yml down
docker-compose -f docker-compose-full.yml up --build -d
```

## Stop Demo
```powershell
docker-compose -f docker-compose-full.yml down
```

## Week 15 Complete! 🎉

All three core systems integrated:
- ✅ OpenClaw Gateway (multi-channel communication)
- ✅ Bridge Signing Service (cryptographic verification)
- ✅ Aureus Agentic OS (policy & planning engine)
