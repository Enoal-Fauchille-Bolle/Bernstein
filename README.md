# Bernstein

A Kubernetes-based voting application with PostgreSQL as persistent storage and Redis for message queuing.

## Table of Contents

- [Architecture](#architecture)
- [PostgreSQL Configuration](#postgresql-configuration)
  - [Components](#components)
  - [Deployment](#deployment)
  - [Security Features](#security-features)
  - [Environment Variables](#environment-variables)
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

### Components

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

### Deployment

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

### Security Features

- Credentials stored securely in Kubernetes Secret with base64 encoding
- Database only accessible via internal ClusterIP service
- No external exposure through Traefik ingress
- Persistent data storage with proper volume management

### Environment Variables

The PostgreSQL container uses the following environment variables:

- `POSTGRES_USER`: From postgres-secret
- `POSTGRES_PASSWORD`: From postgres-secret
- `POSTGRES_DB`: From postgres-config

These variables are automatically referenced by other services in the cluster using:

- Host: `postgres-service` (from postgres-config)
- Port: `5432` (from postgres-config)

## Project Requirements

This configuration follows the Bernstein project specifications:

- ✅ Uses `postgres:12` image
- ✅ Runs in `default` namespace
- ✅ Always restart policy
- ✅ Single instance (not replicated)
- ✅ Exposes port 5432
- ✅ Persistent volume mounted at `/var/lib/postgresql/data`
- ✅ Credentials in Secret, other config in ConfigMap
- ✅ Internal ClusterIP service only
- ✅ Compatible with automated testing
