# Bernstein

A Kubernetes-based voting application with PostgreSQL as persistent storage and Redis for message queuing.

## Table of Contents

- [Architecture](#architecture)
- [PostgreSQL Configuration](#postgresql-configuration)
  - [PostgreSQL Components](#postgresql-components)
  - [PostgreSQL Deployment](#postgresql-deployment)
  - [PostgreSQL Security Features](#postgresql-security-features)
  - [PostgreSQL Environment Variables](#postgresql-environment-variables)
- [Redis Configuration](#redis-configuration)
  - [Redis Components](#redis-components)
  - [Redis Deployment](#redis-deployment)
  - [Redis Security Features](#redis-security-features)
  - [Redis Environment Variables](#redis-environment-variables)
- [Project Requirements](#project-requirements)

## Architecture

The Bernstein application consists of:

- **PostgreSQL**: Persistent database for storing votes
- **Redis**: Message queue for communication between services
- **Poll**: Frontend service for voting interface
- **Worker**: Backend service for processing votes
- **Result**: Service for displaying voting results

![Architecture Schema](./assets/application-schema.png)

## PostgreSQL Configuration

### PostgreSQL Components

The PostgreSQL deployment consists of five main Kubernetes resources:

1. **postgres.secret.yaml**: Stores sensitive database credentials
   - `POSTGRES_USER`: Database username (base64 encoded)
   - `POSTGRES_PASSWORD`: Database password (base64 encoded)

2. **postgres.configmap.yaml**: Contains non-sensitive configuration
   - `POSTGRES_HOST`: Service hostname (postgres-service)
   - `POSTGRES_PORT`: Database port (5432)
   - `POSTGRES_DB`: Database name (postgres)

3. **postgres.volume.yaml**: Defines persistent storage
   - PersistentVolume: 5Gi storage capacity
   - PersistentVolumeClaim: Claims storage for the deployment
   - Mount path: `/var/lib/postgresql/data`

4. **postgres.deployment.yaml**: Main deployment configuration
   - Image: `postgres:12`
   - Namespace: `default`
   - Restart policy: `Always`
   - Replicas: 1 (single instance)
   - Port: 5432
   - Environment variables from Secret and ConfigMap
   - Persistent volume mounted

5. **postgres.service.yaml**: Internal service exposure
   - Type: `ClusterIP` (internal access only)
   - Port: 5432
   - Not exposed via Traefik (secure internal access)

### PostgreSQL Deployment

To deploy the PostgreSQL components:

```bash
# Apply all PostgreSQL resources
kubectl apply -f postgres.secret.yaml
kubectl apply -f postgres.configmap.yaml
kubectl apply -f postgres.volume.yaml
kubectl apply -f postgres.deployment.yaml
kubectl apply -f postgres.service.yaml

# Verify deployment
kubectl get pods -l app=postgres
kubectl get svc postgres-service
kubectl get pvc postgres-pvc
```

### PostgreSQL Security Features

- Credentials stored securely in Kubernetes Secret with base64 encoding
- Database only accessible via internal ClusterIP service
- No external exposure through Traefik ingress
- Persistent data storage with proper volume management

### PostgreSQL Environment Variables

The PostgreSQL container uses the following environment variables:

- `POSTGRES_USER`: From postgres-secret
- `POSTGRES_PASSWORD`: From postgres-secret
- `POSTGRES_DB`: From postgres-config

These variables are automatically referenced by other services in the cluster using:

- Host: `postgres-service` (from postgres-config)
- Port: `5432` (from postgres-config)

## Redis Configuration

### Redis Components

The Redis deployment consists of three main Kubernetes resources:

1. **redis.configmap.yaml**: Contains configuration data
   - `REDIS_HOST`: Service hostname (redis-service)

2. **redis.deployment.yaml**: Main deployment configuration
   - Image: `redis:5.0`
   - Namespace: `default`
   - Restart policy: `Always`
   - Replicas: 1 (single instance)
   - Port: 6379
   - No persistent storage (in-memory cache)

3. **redis.service.yaml**: Internal service exposure
   - Type: `ClusterIP` (internal access only)
   - Port: 6379
   - Not exposed via Traefik (secure internal access)

### Redis Deployment

To deploy the Redis components:

```bash
# Apply all Redis resources
kubectl apply -f redis.configmap.yaml
kubectl apply -f redis.deployment.yaml
kubectl apply -f redis.service.yaml

# Verify deployment
kubectl get pods -l app=redis
kubectl get svc redis-service

# Optional: Test Redis connectivity
kubectl apply -f redis.test.job.yaml
kubectl logs job/redis-test-job
```

### Redis Security Features

- Redis only accessible via internal ClusterIP service
- No external exposure through Traefik ingress
- Runs in default namespace with appropriate labels
- No authentication required for internal cluster communication

### Redis Environment Variables

The Redis service uses the following configuration:

- `REDIS_HOST`: From redis-config (value: `redis-service`)

Other services can connect to Redis using:

- Host: `redis-service` (from redis-config)
- Port: `6379` (default Redis port)

## Project Requirements

This configuration follows the Bernstein project specifications:

### PostgreSQL Requirements

- ✅ Uses `postgres:12` image
- ✅ Runs in `default` namespace
- ✅ Always restart policy
- ✅ Single instance (not replicated)
- ✅ Exposes port 5432
- ✅ Persistent volume mounted at `/var/lib/postgresql/data`
- ✅ Credentials in Secret, other config in ConfigMap
- ✅ Internal ClusterIP service only
- ✅ Compatible with automated testing

### Redis Requirements

- ✅ Uses `redis:5.0` image
- ✅ Runs in `default` namespace
- ✅ Always restart policy
- ✅ Single instance (not replicated)
- ✅ Exposes port 6379
- ✅ Configuration in ConfigMap
- ✅ Internal ClusterIP service only
- ✅ Not exposed via Traefik
- ✅ Compatible with automated testing
