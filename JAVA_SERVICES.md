# Java Services in Robot Shop

This document describes the Java-based microservices in the Robot Shop application.

## Services Overview

The Robot Shop now includes three Java microservices built with Spring Boot:

1. **Shipping Service** (existing)
2. **Inventory Service** (new)
3. **Recommendation Service** (new)

---

## 1. Shipping Service

**Location:** `shipping/`

**Description:** Handles shipping calculations and logistics for orders.

**Technology Stack:**
- Spring Boot 2.3.3
- Java 8
- MySQL database
- Maven

**Endpoints:**
- `GET /health` - Health check
- Additional shipping-related endpoints

---

## 2. Inventory Service

**Location:** `inventory/`

**Description:** Manages product inventory, stock levels, and availability.

**Technology Stack:**
- Spring Boot 2.3.3
- Java 8
- MySQL database
- Maven

**Database:** Uses MySQL to store product inventory data

**Endpoints:**
- `GET /health` - Health check
- `GET /inventory/{sku}` - Get inventory for a specific product SKU
- `GET /inventory` - Get all inventory items
- `POST /inventory/{sku}/reserve?quantity={qty}` - Reserve inventory for an order

**Environment Variables:**
- `DB_HOST` - MySQL database host (default: mysql)

**Features:**
- Real-time inventory tracking
- Stock reservation system
- Location-based inventory management
- Automatic retry logic for database connections

---

## 3. Recommendation Service

**Location:** `recommendation/`

**Description:** Provides product recommendations based on user behavior and product relationships.

**Technology Stack:**
- Spring Boot 2.3.3
- Java 8
- MongoDB database
- Maven

**Database:** Uses MongoDB to store recommendation data and algorithms

**Endpoints:**
- `GET /health` - Health check
- `GET /recommendations/{productId}` - Get recommendations for a specific product
- `GET /recommendations` - Get popular recommendations
- `GET /recommendations/category/{category}` - Get recommendations by category

**Environment Variables:**
- `MONGO_HOST` - MongoDB host (default: mongodb)

**Features:**
- Product-based recommendations
- Category-based recommendations
- Popular products fallback
- Score-based ranking system

---

## Building the Services

### Local Build with Maven

```bash
# Build Inventory Service
cd inventory
mvn clean package

# Build Recommendation Service
cd recommendation
mvn clean package
```

### Docker Build

```bash
# Build Inventory Service
cd inventory
docker build -t robotshop/rs-inventory:latest .

# Build Recommendation Service
cd recommendation
docker build -t robotshop/rs-recommendation:latest .
```

---

## Deploying to OpenShift

### Prerequisites
- OpenShift CLI (`oc`) installed
- Logged in to OpenShift cluster
- `robot-shop` namespace created

### Deployment Steps

1. **Login to OpenShift:**
```bash
oc login --token=<your-token> --server=<your-server>
```

2. **Create namespace (if not exists):**
```bash
oc create namespace robot-shop
oc project robot-shop
```

3. **Build images on OpenShift:**
```bash
# Create binary builds
oc new-build --binary --name=inventory --strategy=docker
oc new-build --binary --name=recommendation --strategy=docker

# Start builds from local directories
oc start-build inventory --from-dir=./inventory --follow
oc start-build recommendation --from-dir=./recommendation --follow
```

4. **Deploy services:**
```bash
# Deploy Inventory Service
oc apply -f K8s/inventory-deployment.yaml

# Deploy Recommendation Service
oc apply -f K8s/recommendation-deployment.yaml
```

5. **Verify deployment:**
```bash
oc get pods -n robot-shop
oc get services -n robot-shop
```

### Using the Deployment Script

Alternatively, use the provided deployment script:

```bash
cd OpenShift
./deploy-java-services.sh
```

---

## Monitoring and Troubleshooting

### Check Pod Status
```bash
oc get pods -n robot-shop | grep -E "inventory|recommendation"
```

### View Logs
```bash
# Inventory Service logs
oc logs -f deployment/inventory

# Recommendation Service logs
oc logs -f deployment/recommendation
```

### Check Build Status
```bash
oc get builds
oc logs -f build/inventory-1
oc logs -f build/recommendation-1
```

### Access Service Endpoints
```bash
# Port forward to test locally
oc port-forward service/inventory 8080:8080
oc port-forward service/recommendation 8081:8080

# Test endpoints
curl http://localhost:8080/health
curl http://localhost:8081/health
```

---

## Architecture

All three Java services follow similar architectural patterns:

- **Spring Boot** framework for rapid development
- **RESTful APIs** for service communication
- **Health checks** for Kubernetes/OpenShift monitoring
- **Instana integration** for observability
- **Retry logic** for resilient database connections
- **Actuator endpoints** for metrics and monitoring

### Database Dependencies

- **Shipping Service:** MySQL
- **Inventory Service:** MySQL
- **Recommendation Service:** MongoDB

Ensure the respective databases are deployed and accessible before deploying the Java services.

---

## Resource Requirements

Each service is configured with:

**Limits:**
- CPU: 500m
- Memory: 1Gi

**Requests:**
- CPU: 200m
- Memory: 512Mi

These can be adjusted in the deployment YAML files based on your cluster capacity and workload requirements.

---

## Future Enhancements

Potential improvements for the Java services:

1. Add caching layer (Redis) for frequently accessed data
2. Implement circuit breakers for external service calls
3. Add comprehensive unit and integration tests
4. Implement API versioning
5. Add Swagger/OpenAPI documentation
6. Implement rate limiting
7. Add authentication and authorization
8. Implement distributed tracing with Jaeger