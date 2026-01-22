# Apache Superset - Modular Deployment Guide

Deploy only what you need: just Superset, or with optional PostgreSQL, Redis, and Workers.

## 🎯 Deployment Scenarios

This setup supports multiple deployment configurations:

| Scenario | Components | Use Case |
|----------|------------|----------|
| **Minimal** | Superset only | Use external managed database and Redis |
| **With Database** | Superset + PostgreSQL | Self-contained but use external Redis |
| **With Cache** | Superset + Redis | Use external database, bundled cache |
| **With Workers** | Superset + Workers | Async queries with external DB/Redis |
| **Full Stack** | Everything | Complete self-contained deployment |

## 📋 Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- Your custom Superset image on Docker Hub
- External database (if not using bundled PostgreSQL)

## 🚀 Quick Start

### 1. Minimal Setup (Superset Only)

Use this when you have external PostgreSQL and Redis servers.

**Configuration (.env):**
```bash
SUPERSET_IMAGE=yourcompany/superset:latest
SECRET_KEY=$(openssl rand -base64 42)
SUPERSET_PORT=8088

# Point to your external database
DATABASE_HOST=your-postgres.example.com
DATABASE_PORT=5432
DATABASE_USER=superset
DATABASE_PASSWORD=your_db_password
DATABASE_DB=superset

# Point to your external Redis
REDIS_HOST=your-redis.example.com
REDIS_PORT_INTERNAL=6379

# Don't start bundled services
COMPOSE_PROFILES=
```

**Deploy:**
```bash
docker-compose up -d superset
```

This starts **only** the Superset container, connecting to your external services.

### 2. Full Stack Setup

Use this for a complete self-contained deployment.

**Configuration (.env):**
```bash
SUPERSET_IMAGE=yourcompany/superset:latest
SECRET_KEY=$(openssl rand -base64 42)
SUPERSET_PORT=8088

# Use bundled services
DATABASE_HOST=postgres
DATABASE_PASSWORD=secure_password_here
REDIS_HOST=redis

# Enable all services
COMPOSE_PROFILES=full
```

**Deploy:**
```bash
docker-compose up -d
```

This starts everything: Superset, PostgreSQL, Redis, and Workers.

## 🔧 Deployment Profiles

Use Docker Compose profiles to control which services start:

### Profile 1: Minimal (Superset Only)

```bash
# .env
COMPOSE_PROFILES=

# Deploy
docker-compose up -d superset
```

**Starts:** Superset only  
**Requires:** External PostgreSQL and Redis

### Profile 2: With Database

```bash
# .env
COMPOSE_PROFILES=database

# Deploy
docker-compose up -d
```

**Starts:** Superset + PostgreSQL  
**Requires:** External Redis (or disable caching)

### Profile 3: With Cache

```bash
# .env
COMPOSE_PROFILES=cache

# Deploy
docker-compose up -d
```

**Starts:** Superset + Redis  
**Requires:** External PostgreSQL

### Profile 4: With Workers

```bash
# .env
COMPOSE_PROFILES=workers

# Deploy
docker-compose up -d
```

**Starts:** Superset + Celery Workers + Beat  
**Requires:** External PostgreSQL and Redis

### Profile 5: Full Stack

```bash
# .env
COMPOSE_PROFILES=full

# Deploy
docker-compose up -d
```

**Starts:** Everything (Superset + PostgreSQL + Redis + Workers + Beat)

## 📝 Configuration Examples

### Example 1: AWS RDS + ElastiCache

Using managed AWS services:

```bash
# .env
SUPERSET_IMAGE=mycompany/superset:v1.0.0
SECRET_KEY=your-secret-key-here
SUPERSET_PORT=8088

# AWS RDS PostgreSQL
DATABASE_HOST=superset-db.abc123.us-east-1.rds.amazonaws.com
DATABASE_PORT=5432
DATABASE_DIALECT=postgresql
DATABASE_USER=superset_admin
DATABASE_PASSWORD=rds_password
DATABASE_DB=superset_prod

# AWS ElastiCache Redis
REDIS_HOST=superset-cache.abc123.0001.use1.cache.amazonaws.com
REDIS_PORT_INTERNAL=6379

# Only Superset container needed
COMPOSE_PROFILES=
```

```bash
# Initialize database (first time only)
docker-compose run --rm superset superset db upgrade
docker-compose run --rm superset superset init

# Deploy
docker-compose up -d superset
```

### Example 2: Cloud SQL + Memorystore

Using Google Cloud services:

```bash
# .env
SUPERSET_IMAGE=mycompany/superset:latest
SECRET_KEY=your-secret-key-here

# Cloud SQL (PostgreSQL)
DATABASE_HOST=10.0.0.3
DATABASE_PORT=5432
DATABASE_DIALECT=postgresql
DATABASE_USER=superset
DATABASE_PASSWORD=cloudsql_password
DATABASE_DB=superset

# Memorystore (Redis)
REDIS_HOST=10.0.0.5
REDIS_PORT_INTERNAL=6379

COMPOSE_PROFILES=
```

### Example 3: Self-hosted with Workers

Local database but managed Redis:

```bash
# .env
SUPERSET_IMAGE=mycompany/superset:latest
SECRET_KEY=your-secret-key-here

# Bundled PostgreSQL
DATABASE_HOST=postgres
DATABASE_PASSWORD=secure_local_password

# External Redis
REDIS_HOST=redis.internal.company.com
REDIS_PORT_INTERNAL=6379

# Start database and workers
COMPOSE_PROFILES=database,workers
```

```bash
docker-compose up -d
```

### Example 4: Development Setup

Full stack for local development:

```bash
# .env
SUPERSET_IMAGE=mycompany/superset:dev
SECRET_KEY=dev-secret-key
SUPERSET_PORT=8088
SUPERSET_LOAD_EXAMPLES=yes

DATABASE_HOST=postgres
DATABASE_PASSWORD=dev_password
REDIS_HOST=redis

COMPOSE_PROFILES=full
```

## 🛠️ Common Operations

### Initialize Superset (First Time)

```bash
# Run database migrations
docker-compose run --rm superset superset db upgrade

# Create admin user
docker-compose run --rm superset superset fab create-admin \
  --username admin \
  --firstname Admin \
  --lastname User \
  --email admin@example.com \
  --password admin

# Initialize Superset
docker-compose run --rm superset superset init

# Optional: Load examples
docker-compose run --rm superset superset load-examples
```

### Start/Stop Services

```bash
# Start all configured services
docker-compose up -d

# Start only specific service
docker-compose up -d superset

# Stop all
docker-compose stop

# Remove containers (keeps data)
docker-compose down

# Remove everything including data
docker-compose down -v
```

### Update Superset Image

```bash
# Pull new image
docker pull yourcompany/superset:v2.0.0

# Update .env
# SUPERSET_IMAGE=yourcompany/superset:v2.0.0

# Recreate containers
docker-compose up -d

# Run migrations
docker-compose run --rm superset superset db upgrade
```

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f superset
docker-compose logs -f postgres
docker-compose logs -f superset-worker
```

### Check Service Status

```bash
# View running services
docker-compose ps

# Health check
curl http://localhost:8088/health
```

## 🔌 External Service Configuration

### Using External PostgreSQL

Requirements:
- PostgreSQL 12+
- Empty database created
- User with full permissions on the database

**Setup:**
```sql
-- On your PostgreSQL server
CREATE DATABASE superset;
CREATE USER superset_user WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE superset TO superset_user;
```

**Configure in .env:**
```bash
DATABASE_HOST=your-postgres-server.example.com
DATABASE_PORT=5432
DATABASE_USER=superset_user
DATABASE_PASSWORD=secure_password
DATABASE_DB=superset
```

### Using External Redis

Requirements:
- Redis 5+
- Network accessible from Superset

**Configure in .env:**
```bash
REDIS_HOST=your-redis-server.example.com
REDIS_PORT_INTERNAL=6379
```

### Using MySQL Instead of PostgreSQL

**Configure in .env:**
```bash
DATABASE_DIALECT=mysql
DATABASE_HOST=your-mysql-server.example.com
DATABASE_PORT=3306
DATABASE_USER=superset_user
DATABASE_PASSWORD=mysql_password
DATABASE_DB=superset
```

**Note:** Ensure your Superset image includes `mysqlclient` driver.

## 🏗️ Architecture Patterns

### Pattern 1: Kubernetes + Managed Services

```
┌─────────────────────┐
│   Kubernetes Pod    │
│   ┌─────────────┐   │     ┌──────────────────┐
│   │  Superset   │───┼────▶│  AWS RDS         │
│   └─────────────┘   │     │  (PostgreSQL)    │
│                     │     └──────────────────┘
└─────────────────────┘
          │
          ▼
    ┌──────────────────┐
    │  ElastiCache     │
    │  (Redis)         │
    └──────────────────┘
```

Deploy: `COMPOSE_PROFILES=` (Superset only)

### Pattern 2: Docker Compose Full Stack

```
┌─────────────────────────────────────┐
│        Docker Compose Host          │
│  ┌──────────┐  ┌──────────────────┐ │
│  │Superset  │  │ Celery Workers   │ │
│  └──────────┘  └──────────────────┘ │
│       │               │              │
│       ▼               ▼              │
│  ┌──────────┐  ┌──────────────────┐ │
│  │PostgreSQL│  │ Redis            │ │
│  └──────────┘  └──────────────────┘ │
└─────────────────────────────────────┘
```

Deploy: `COMPOSE_PROFILES=full`

### Pattern 3: Hybrid Cloud

```
┌──────────────┐
│ Docker Host  │     ┌──────────────────┐
│ ┌──────────┐ │────▶│ Cloud SQL        │
│ │Superset  │ │     │ (Managed DB)     │
│ └──────────┘ │     └──────────────────┘
└──────────────┘
      │
      ▼
┌──────────────────┐
│ On-prem Redis    │
│ (or Bundled)     │
└──────────────────┘
```

Deploy: `COMPOSE_PROFILES=` or `COMPOSE_PROFILES=cache`

## 🔒 Security Considerations

1. **Network Isolation**: Use Docker networks or cloud VPCs
2. **Secrets Management**: Consider using Docker secrets or cloud secret managers
3. **Database Security**: Use SSL/TLS for database connections
4. **Redis Security**: Enable authentication if Redis is exposed
5. **Firewall Rules**: Restrict access to necessary ports only
6. **Image Security**: Regularly scan and update images

## 📊 Monitoring

### Health Endpoints

```bash
# Superset health
curl http://localhost:8088/health

# PostgreSQL (if using bundled)
docker-compose exec postgres pg_isready

# Redis (if using bundled)
docker-compose exec redis redis-cli ping
```

### Resource Monitoring

```bash
# Container stats
docker stats

# Logs monitoring
docker-compose logs -f --tail=100
```

## 🐛 Troubleshooting

### Superset Can't Connect to Database

```bash
# Check database connectivity
docker-compose exec superset ping -c 3 postgres

# Test database connection
docker-compose exec superset python -c "
from sqlalchemy import create_engine
import os
uri = f\"postgresql://{os.environ['DATABASE_USER']}:{os.environ['DATABASE_PASSWORD']}@{os.environ['DATABASE_HOST']}:{os.environ['DATABASE_PORT']}/{os.environ['DATABASE_DB']}\"
engine = create_engine(uri)
print('Connected:', engine.connect())
"
```

### Workers Not Processing Tasks

```bash
# Check worker logs
docker-compose logs superset-worker

# Verify Redis connection
docker-compose exec superset-worker redis-cli -h $REDIS_HOST ping

# Check Celery status
docker-compose exec superset-worker celery -A superset.tasks.celery_app:app inspect active
```

### Service Won't Start

```bash
# Check which profile is active
docker-compose config --profiles

# Verify environment variables
docker-compose config

# View service dependencies
docker-compose ps
```

## 📚 Additional Resources

- [Superset Documentation](https://superset.apache.org/docs/intro)
- [Docker Compose Profiles](https://docs.docker.com/compose/profiles/)
- [Superset Configuration](https://superset.apache.org/docs/configuration/configuring-superset)

## 🎓 Next Steps

1. **Build Your Image**: Use the Dockerfile from previous artifacts
2. **Push to Registry**: `docker push yourcompany/superset:latest`
3. **Choose Your Profile**: Select deployment scenario
4. **Configure .env**: Set database and Redis settings
5. **Deploy**: Run `docker-compose up -d`
6. **Initialize**: Run database migrations and create admin
7. **Access**: Open http://localhost:8088

Enjoy your flexible Superset deployment! 🚀