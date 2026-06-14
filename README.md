# Real-time Analytics Dashboard

A full-stack analytics and order management platform built for FinTech and E-commerce clients. Handles high-throughput transaction data with real-time dashboard updates and a scalable microservices backend.

## Architecture Overview

```
┌──────────────────┐     ┌─────────────────────────────────────┐
│   React.js UI    │────▶│         API Gateway                 │
│  (TypeScript)    │◀────│                                     │
│  Live Dashboard  │     └──────────────┬──────────────────────┘
└──────────────────┘                    │
                           ┌────────────▼────────────┐
                           │   Spring Boot Services  │
                           │  ┌──────┐  ┌─────────┐  │
                           │  │Order │  │Analytics│  │
                           │  │ Svc  │  │  Svc    │  │
                           │  └──┬───┘  └────┬────┘  │
                           └─────┼───────────┼────────┘
                                 │           │
                    ┌────────────▼┐    ┌─────▼──────┐
                    │ PostgreSQL  │    │  MongoDB   │
                    │(transact.)  │    │(analytics) │
                    └─────────────┘    └────────────┘
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React.js, TypeScript, Recharts |
| Backend | Spring Boot, Java 17 |
| Databases | PostgreSQL (transactional), MongoDB (analytics) |
| Messaging | RabbitMQ |
| Cloud | AWS Lambda, AWS SNS |
| DevOps | Docker, Kubernetes, Jenkins |

## Key Features

- **Real-time dashboard** — live metrics updates for orders, revenue, and inventory KPIs
- **Order management system** — full lifecycle tracking from placement to delivery
- **Multi-model data layer** — PostgreSQL for transactional integrity, MongoDB for flexible analytics storage
- **Event-driven order processing** — RabbitMQ queues decouple order intake from downstream processing
- **Serverless compute** — AWS Lambda handles scheduled aggregations and report generation
- **Scalable deployment** — Kubernetes with HPA handles traffic spikes without downtime

## Microservices

### Order Service
- Manages order lifecycle: `PLACED → PROCESSING → SHIPPED → DELIVERED`
- Publishes order events to RabbitMQ exchange
- PostgreSQL for transactional order records

### Analytics Service
- Consumes order events from RabbitMQ
- Aggregates metrics into MongoDB collections
- Exposes REST endpoints for dashboard consumption

### Notification Service
- Subscribes to AWS SNS topics for pub/sub fan-out
- Sends order status updates to customers via email/SMS
- AWS Lambda triggers for scheduled digest reports

## API Endpoints

```
GET    /api/v1/orders                    # List orders with filters
POST   /api/v1/orders                    # Create new order
GET    /api/v1/orders/{id}               # Get order details
PATCH  /api/v1/orders/{id}/status        # Update order status

GET    /api/v1/analytics/revenue         # Revenue metrics
GET    /api/v1/analytics/orders/summary  # Order volume summary
GET    /api/v1/analytics/kpis            # Live KPI dashboard data
```

## Database Schema (PostgreSQL)

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id UUID NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'PLACED',
    total_amount DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE order_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID REFERENCES orders(id),
    product_id VARCHAR(100) NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL
);
```

## Event Flow (RabbitMQ)

```
Order Created → Exchange: orders.events
                    ├── Queue: analytics.order.created
                    ├── Queue: notification.order.created
                    └── Queue: inventory.order.created
```

## Frontend Component Library

Reusable TypeScript components built for consistency across product teams:

```tsx
// KPI Card Component
<KPICard
  title="Total Revenue"
  value={revenue}
  trend={+12.5}
  period="vs last month"
/>

// Order Status Timeline
<OrderTimeline
  orderId={orderId}
  events={orderEvents}
  currentStatus="SHIPPED"
/>
```

## Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: order-service
          image: order-service:latest
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          averageUtilization: 70
```

## Results

- **70% throughput improvement** after microservices migration from monolith
- **99.9% uptime** maintained across distributed production environment
- **40% reduction** in critical UI bugs via shared TypeScript component library
- **40% reduction** in system defects quarter-over-quarter through performance profiling

---

> **Note:** This repository contains architecture documentation. Source code is proprietary to client engagements.
