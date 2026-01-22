# Building Superset Image Only

This guide shows you how to build just the Superset Docker image without deploying anything.

### 1. Build the Image

Choose ONE of these build commands:

#### Option A: Build with a specific tag
```bash
docker build -t yourcompany/superset:v1.0.0 .
```

#### Option B: Build with version and latest tags
```bash
docker build -t yourcompany/superset:v1.0.0 -t yourcompany/superset:latest .
```

### 5. Verify the Build

```bash
# List your images
docker images | grep superset

# Expected output:
# yourcompany/superset   v1.0.0    abc123def456   2 minutes ago   2.5GB
# yourcompany/superset   latest    abc123def456   2 minutes ago   2.5GB
```

### 6. Test the Image (Optional)

```bash
# Quick test - start Superset container
docker run --rm -p 8088:8088 \
  -e SECRET_KEY=test \
  -e DATABASE_DIALECT=sqlite \
  yourcompany/superset:v1.0.0

# Access http://localhost:8088
# Press Ctrl+C to stop
```

### 7. Push to Docker Hub

```bash
# Login to Docker Hub
docker login

# Push specific version
docker push yourcompany/superset:v1.0.0

# Push latest tag
docker push yourcompany/superset:latest
```