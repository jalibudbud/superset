# Superset Deployment Quick Reference

Copy-paste examples for common deployment scenarios.

## 🎯 Scenario 1: Superset Only (External Services)

**When to use:** You have AWS RDS, Cloud SQL, or other managed database/cache services.

**.env file:**
```bash
SUPERSET_IMAGE=yourcompany/superset:latest
SECRET_KEY=your-42-character-secret-key-here
SUPERSET_PORT=8088

# Your external PostgreSQL
DATABASE_HOST=my-db.abc123.us-east-1.rds.amazonaws.com
DATABASE_PORT=5432
DATABASE_DIALECT=postgresql
DATABASE_USER=superset
DATABASE_PASSWORD=my_db_password
DATABASE_DB=superset_prod

# Your external Redis
REDIS_HOST=my-redis.abc123.0001.use1.cache.amazonaws.com
REDIS_PORT_INTERNAL=6379

# No bundled services
COMPOSE_PROFILES=
```

**Commands:**
```bash
# First time setup
docker-compose run --rm superset superset db upgrade
docker-compose run --rm superset superset fab create-admin
docker-compose run --rm superset superset init

# Start Superset
docker-compose up -d superset

# Check status
curl http://localhost:8088/health
```

**What runs:** Only Superset container

---

## 🎯 Scenario 2: Superset + PostgreSQL (External Redis)

**When to use:** You have external Redis but want bundled database.

**.env file:**
```bash
SUPERSET_IMAGE=yourcompany/superset:latest
SECRET_KEY=your-42-character-secret-key-here
SUPERSET_PORT=8088

# Bundled PostgreSQL
DATABASE_HOST=postgres
DATABASE_PORT=5432
DATABASE_DIALECT=postgresql
DATABASE_USER=superset
DATABASE_PASSWORD=secure_postgres_password
DATABASE_DB=superset
POSTGRES_PORT=5432

# External Redis
REDIS_HOST=redis.mycompany.internal
REDIS_PORT_INTERNAL=6379

# Enable database service
COMPOSE_PROFILES=database
```

**Commands:**
```bash
# Start services
docker-compose up -d

# Initialize (first time)
docker-compose run --rm superset superset db upgrade
docker-compose run --rm superset superset fab create-admin
docker-compose run --rm superset superset init

# Check status
docker-compose ps
```

**What runs:** Superset + PostgreSQL containers

---

## 🎯 Scenario 3: Superset + Redis (External Database)

**When to use:** You have external database but want bundled Redis for caching.

**.env file:**
```bash
SUPERSET_IMAGE=yourcompany/superset:latest
SECRET_KEY=your-42-character-secret-key-here
SUPERSET_PORT=8088

# External PostgreSQL
DATABASE_HOST=postgres.mycompany.internal
DATABASE_PORT=5432
DATABASE_DIALECT=postgresql
DATABASE_USER=superset
DATABASE_PASSWORD=external_db_password
DATABASE_DB=superset_prod

# Bundled Redis
REDIS_HOST=redis
REDIS_PORT_INTERNAL=6379
REDIS_PORT=6379

# Enable cache service
COMPOSE_PROFILES=cache
```

**Commands:**
```bash
# Start services
docker-compose up -d

# Initialize (first time)
docker-compose run --rm superset superset db upgrade
docker-compose run --rm superset superset fab create-admin
docker-compose run --rm superset superset init
```

**What runs:** Superset + Redis containers

---

## 🎯 Scenario 4: Superset + Workers (External DB & Redis)

**When to use:** You need async queries/alerts but have external database and Redis.

**.env file:**
```bash
SUPERSET_IMAGE=yourcompany/superset:latest
SECRET_KEY=your-42-character-secret-key-here
SUPERSET_PORT=8088

# External PostgreSQL
DATABASE_HOST=my-postgres.example.com
DATABASE_PORT=5432
DATABASE_DIALECT=postgresql
DATABASE_USER=superset
DATABASE_PASSWORD=db_password
DATABASE_DB=superset

# External Redis
REDIS_HOST=my-redis.example.com
REDIS_PORT_INTERNAL=6379

# Celery configuration
CELERY_WORKERS=4

# Enable workers
COMPOSE_PROFILES=workers
```

**Commands:**
```bash
# Start all services
docker-compose up -d

# Check workers
docker-compose logs -f superset-worker
docker-compose exec superset-worker celery -A superset.tasks.celery_app:app inspect active
```

**What runs:** Superset + Celery Worker + Celery Beat containers

---

## 🎯 Scenario 5: Full Stack (Everything Bundled)

**When to use:** Development, testing, or simple production on single host.

**.env file:**
```bash
SUPERSET_IMAGE=yourcompany/superset:latest
SECRET_KEY=your-42-character-secret-key-here
SUPERSET_PORT=8088
SUPERSET_LOAD_EXAMPLES=no

# Bundled PostgreSQL
DATABASE_HOST=postgres
DATABASE_PORT=5432
DATABASE_DIALECT=postgresql
DATABASE_USER=superset
DATABASE_PASSWORD=secure_postgres_password
DATABASE_DB=superset
POSTGRES_PORT=5432

# Bundled Redis
REDIS_HOST=redis
REDIS_PORT_INTERNAL=6379
REDIS_PORT=6379

# Celery configuration
CELERY_WORKERS=4

# Enable all services
COMPOSE_PROFILES=full
```

**Commands:**
```bash
# Start everything
docker-compose up -d

# Initialize (first time)
docker-compose run --rm superset superset db upgrade
docker-compose run --rm superset superset fab create-admin
docker-compose run --rm superset superset init

# View all logs
docker-compose logs -f

# Check status
docker-compose ps
```

**What runs:** Superset + PostgreSQL + Redis + Celery Worker + Celery Beat

---

## 🎯 Scenario 6: Multiple Environments on Same Host

**When to use:** Running dev, staging, and prod on same server.

**Directory structure:**
```
/opt/superset/
├── dev/
│   ├── docker-compose.yml
│   └── .env
├── staging/
│   ├── docker-compose.yml
│   └── .env
└── prod/
    ├── docker-compose.yml
    └── .env
```

**dev/.env:**
```bash
SUPERSET_IMAGE=yourcompany/superset:dev
SECRET_KEY=dev-secret-key-unique-42-chars
SUPERSET_PORT=8088
DATABASE_HOST=postgres
DATABASE_PASSWORD=dev_db_password
REDIS_HOST=redis
COMPOSE_PROFILES=full
SUPERSET_LOAD_EXAMPLES=yes
```

**staging/.env:**
```bash
SUPERSET_IMAGE=yourcompany/superset:staging
SECRET_KEY=staging-secret-key-unique-42-chars
SUPERSET_PORT=8089
DATABASE_HOST=postgres
DATABASE_PASSWORD=staging_db_password
REDIS_HOST=redis
COMPOSE_PROFILES=full
SUPERSET_LOAD_EXAMPLES=no
```

**prod/.env:**
```bash
SUPERSET_IMAGE=yourcompany/superset:v1.0.0
SECRET_KEY=prod-secret-key-unique-42-chars
SUPERSET_PORT=8090
DATABASE_HOST=postgres
DATABASE_PASSWORD=prod_db_password
REDIS_HOST=redis
COMPOSE_PROFILES=full
SUPERSET_LOAD_EXAMPLES=no
CELERY_WORKERS=8
```

**Commands:**
```bash
# Deploy all environments
cd /opt/superset/dev && docker-compose up -d
cd /opt/superset/staging && docker-compose up -d
cd /opt/superset/prod && docker-compose up -d

# Access each environment
# Dev:     http://localhost:8088
# Staging: http://localhost:8089
# Prod:    http://localhost:8090
```

---

## 🎯 Scenario 7: Using MySQL Instead of PostgreSQL

**.env file:**
```bash
SUPERSET_IMAGE=yourcompany/superset:latest
SECRET_KEY=your-42-character-secret-key-here
SUPERSET_PORT=8088

# MySQL configuration
DATABASE_DIALECT=mysql
DATABASE_HOST=mysql.mycompany.internal
DATABASE_PORT=3306
DATABASE_USER=superset
DATABASE_PASSWORD=mysql_password
DATABASE_DB=superset

# Bundled Redis
REDIS_HOST=redis
REDIS_PORT_INTERNAL=6379

COMPOSE_PROFILES=cache
```

**Note:** Ensure your Superset image includes `mysqlclient` driver!

---

## 🔧 Quick Commands Reference

### First Time Setup
```bash
docker-compose run --rm superset superset db upgrade
docker-compose run --rm superset superset fab create-admin
docker-compose run --rm superset superset init
```

### Start/Stop
```bash
docker-compose up -d          # Start all configured services
docker-compose up -d superset # Start only Superset
docker-compose stop           # Stop all
docker-compose down           # Stop and remove containers
```

### Monitoring
```bash
docker-compose ps             # Service status
docker-compose logs -f        # All logs
docker-compose logs -f superset  # Superset logs only
curl http://localhost:8088/health  # Health check
```

### Updates
```bash
docker pull yourcompany/superset:latest
docker-compose up -d
docker-compose run --rm superset superset db upgrade
```

### Backup
```bash
# PostgreSQL backup (if using bundled)
docker-compose exec postgres pg_dump -U superset superset > backup.sql

# Restore
docker-compose exec -T postgres psql -U superset superset < backup.sql
```

---

## 🛠️ Troubleshooting Commands

### Check connectivity
```bash
# Test database connection
docker-compose exec superset python -c "
from sqlalchemy import create_engine
import os
uri = f\"postgresql://{os.environ['DATABASE_USER']}:{os.environ['DATABASE_PASSWORD']}@{os.environ['DATABASE_HOST']}:{os.environ['DATABASE_PORT']}/{os.environ['DATABASE_DB']}\"
engine = create_engine(uri)
conn = engine.connect()
print('Database connected!')
"

# Test Redis connection
docker-compose exec superset redis-cli -h $REDIS_HOST ping
```

### View configuration
```bash
# See what will be deployed
docker-compose config

# See active profiles
docker-compose config --profiles

# View environment variables
docker-compose exec superset env | grep -E 'DATABASE|REDIS|SECRET'
```

### Reset everything
```bash
docker-compose down -v  # ⚠️ Deletes all data!
docker-compose up -d
# Run initialization again
```

---

## 📝 Notes

- Always use unique `SECRET_KEY` for each environment
- Use different passwords for dev/staging/prod
- External services need to be network accessible
- Workers require Redis to be configured
- PostgreSQL and MySQL databases need to be created before first run