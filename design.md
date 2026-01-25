# Valora AI - Technical Design Document

## System Architecture Overview

Valora AI is built as a cloud-native, microservices-based platform leveraging AWS services for scalability and reliability. The architecture follows a three-tier design pattern with clear separation between presentation, business logic, and data layers.

```
┌─────────────────────────────────────────────────────────────────┐
│                        Frontend Layer                           │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   React SPA     │  │  Mobile PWA     │  │   Admin Panel   │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                │
                        ┌───────────────┐
                        │  API Gateway  │
                        │   (AWS ALB)   │
                        └───────────────┘
                                │
┌─────────────────────────────────────────────────────────────────┐
│                      Backend Services                           │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │  FastAPI Core   │  │   ML Pipeline   │  │  Integration    │ │
│  │    Service      │  │   (SageMaker)   │  │    Service      │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────────┐
│                        Data Layer                               │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   PostgreSQL    │  │      Redis      │  │      S3         │ │
│  │   (RDS/Aurora)  │  │   (ElastiCache) │  │  (Data Lake)    │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Major Components

### 1. Frontend Layer

#### React Single Page Application (SPA)
- **Technology**: React 18 with TypeScript, Vite build system
- **State Management**: Redux Toolkit with RTK Query
- **UI Framework**: Material-UI (MUI) with custom theme
- **Visualization**: D3.js for interactive charts and Recharts for standard graphs
- **Authentication**: Auth0 integration with JWT tokens

**Key Features:**
- Feature prioritization dashboard
- ROI prediction visualization
- Interactive ranking interface
- Real-time updates via WebSocket
- Responsive design for desktop and tablet

#### Progressive Web App (PWA)
- **Technology**: React Native Web with Capacitor
- **Purpose**: Mobile access for on-the-go prioritization decisions
- **Features**: Offline capability, push notifications, native mobile feel

### 2. Backend Services

#### FastAPI Core Service
- **Technology**: FastAPI with Python 3.11, Pydantic for validation
- **Architecture**: Hexagonal architecture with dependency injection
- **Database ORM**: SQLAlchemy with Alembic migrations
- **Authentication**: JWT-based with role-based access control (RBAC)
- **API Documentation**: Auto-generated OpenAPI/Swagger docs

**Responsibilities:**
- User management and authentication
- Feature CRUD operations
- Ranking algorithm orchestration
- Real-time WebSocket connections
- Integration coordination

#### ML Pipeline Service (AWS SageMaker)
- **Technology**: Python with scikit-learn, XGBoost, and TensorFlow
- **Infrastructure**: SageMaker Training Jobs and Endpoints
- **Model Storage**: S3 with versioning and MLflow tracking
- **Feature Store**: SageMaker Feature Store for ML features

**ML Models:**
- **ROI Prediction Model**: Gradient boosting ensemble (XGBoost + LightGBM)
- **Effort Estimation Model**: Neural network with historical development data
- **Customer Signal Analysis**: NLP model for sentiment and priority extraction
- **Feature Similarity Model**: Embedding-based clustering for related features

#### Integration Service
- **Technology**: FastAPI with async/await for concurrent API calls
- **Rate Limiting**: Token bucket algorithm with Redis backing
- **Retry Logic**: Exponential backoff with circuit breaker pattern
- **Webhook Handling**: Event-driven updates from external systems

**Supported Integrations:**
- Jira REST API v3 with OAuth 2.0
- GitHub REST API v4 and GraphQL API
- Slack for notifications
- Linear, Asana (future roadmap)

### 3. Data Layer

#### Primary Database (PostgreSQL on AWS RDS)
```sql
-- Core tables structure
users (id, email, role, team_id, created_at)
teams (id, name, settings, subscription_tier)
features (id, title, description, status, team_id, jira_key)
predictions (id, feature_id, roi_score, confidence, model_version)
rankings (id, feature_id, score, rank, criteria_weights)
integrations (id, team_id, type, config, status)
```

#### Cache Layer (Redis on ElastiCache)
- Session storage and JWT blacklisting
- API response caching (5-minute TTL)
- Real-time ranking calculations
- Rate limiting counters
- WebSocket connection management

#### Data Lake (S3)
- Raw integration data (JSON format, partitioned by date)
- ML training datasets and feature engineering pipelines
- Model artifacts and experiment tracking
- Audit logs and analytics data
- Backup and disaster recovery storage

## Data Flow Architecture

### 1. Data Ingestion Flow
```
External APIs → Integration Service → Data Validation → S3 Raw Storage
                      ↓
              Feature Engineering → Feature Store → ML Training Pipeline
                      ↓
                PostgreSQL → Cache Layer → Frontend Updates
```

### 2. Prediction Generation Flow
```
Feature Request → Core Service → ML Service (SageMaker Endpoint)
                                      ↓
                 Prediction Results → Database Storage → Cache Update
                                      ↓
                 WebSocket Broadcast → Frontend Real-time Update
```

### 3. Ranking Calculation Flow
```
User Criteria → Ranking Algorithm → ML Predictions + Manual Weights
                      ↓
              Calculated Scores → Database + Cache → Dashboard Update
```

## User Interaction Flow

### 1. Initial Setup Flow
```
User Registration → Team Creation → Integration Setup → Data Sync
                                         ↓
                    Historical Analysis → Initial Predictions → Dashboard Ready
```

### 2. Feature Prioritization Flow
```
Feature Input → Automatic Analysis → ROI Prediction → Ranking Calculation
                      ↓
              User Review → Manual Adjustments → Final Ranking → Epic Creation
```

### 3. Continuous Learning Flow
```
Feature Delivery → Outcome Tracking → Performance Measurement → Model Retraining
                                         ↓
                    Updated Predictions → Improved Accuracy → User Notification
```

## API Design (High Level)

### Core API Endpoints

#### Authentication & Users
```
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
GET    /api/v1/users/profile
PUT    /api/v1/users/profile
GET    /api/v1/teams/{team_id}/members
```

#### Features Management
```
GET    /api/v1/features                    # List with filtering/pagination
POST   /api/v1/features                    # Create new feature
GET    /api/v1/features/{feature_id}       # Get feature details
PUT    /api/v1/features/{feature_id}       # Update feature
DELETE /api/v1/features/{feature_id}       # Delete feature
```

#### Predictions & Rankings
```
POST   /api/v1/features/{feature_id}/predict     # Generate ROI prediction
GET    /api/v1/rankings                          # Get current rankings
POST   /api/v1/rankings/calculate                # Recalculate rankings
PUT    /api/v1/rankings/{feature_id}             # Manual ranking adjustment
```

#### Integrations
```
GET    /api/v1/integrations                      # List active integrations
POST   /api/v1/integrations/jira                 # Setup Jira integration
POST   /api/v1/integrations/github               # Setup GitHub integration
POST   /api/v1/integrations/{id}/sync            # Trigger manual sync
```

#### ML & Analytics
```
GET    /api/v1/models/performance                 # Model accuracy metrics
POST   /api/v1/models/retrain                    # Trigger model retraining
GET    /api/v1/analytics/team                    # Team performance analytics
GET    /api/v1/analytics/features                # Feature outcome analytics
```

### WebSocket Events
```
feature.created         # New feature added
feature.updated         # Feature modified
prediction.completed    # ROI prediction ready
ranking.updated         # Rankings recalculated
integration.synced      # External data synchronized
model.retrained         # ML model updated
```

### API Response Format
```json
{
  "success": true,
  "data": {
    "features": [...],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150
    }
  },
  "meta": {
    "timestamp": "2026-01-25T10:30:00Z",
    "version": "v1.2.0"
  }
}
```

## Deployment Architecture (AWS)

### Infrastructure Components

#### Compute Layer
```
┌─────────────────────────────────────────────────────────────┐
│                    AWS ECS Fargate                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐ │
│  │  FastAPI Core   │  │  Integration    │  │   Worker    │ │
│  │   (3 tasks)     │  │    Service      │  │   Queue     │ │
│  │                 │  │   (2 tasks)     │  │ (Celery)    │ │
│  └─────────────────┘  └─────────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

#### ML Infrastructure
```
┌─────────────────────────────────────────────────────────────┐
│                   AWS SageMaker                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐ │
│  │  Training Jobs  │  │   Endpoints     │  │ Feature     │ │
│  │  (On-demand)    │  │ (Real-time ML)  │  │   Store     │ │
│  └─────────────────┘  └─────────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

#### Data Infrastructure
```
┌─────────────────────────────────────────────────────────────┐
│                     Data Services                           │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐ │
│  │   RDS Aurora    │  │  ElastiCache    │  │     S3      │ │
│  │  (Multi-AZ)     │  │    (Redis)      │  │ (Data Lake) │ │
│  └─────────────────┘  └─────────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Deployment Pipeline

#### CI/CD with GitHub Actions
```yaml
# .github/workflows/deploy.yml
name: Deploy to AWS
on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: |
          pytest backend/tests/
          npm test frontend/
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to ECS
        run: |
          aws ecs update-service --cluster valora-cluster
          aws sagemaker update-endpoint --endpoint-name valora-ml
```

#### Infrastructure as Code (Terraform)
```hcl
# infrastructure/main.tf
resource "aws_ecs_cluster" "valora" {
  name = "valora-cluster"
  
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}

resource "aws_sagemaker_endpoint" "valora_ml" {
  name                 = "valora-ml-endpoint"
  endpoint_config_name = aws_sagemaker_endpoint_configuration.valora.name
}
```

### Environment Configuration

#### Production Environment
- **Compute**: ECS Fargate with auto-scaling (2-10 tasks)
- **Database**: RDS Aurora PostgreSQL (Multi-AZ)
- **Cache**: ElastiCache Redis (Cluster mode)
- **ML**: SageMaker multi-model endpoints
- **CDN**: CloudFront for static assets
- **Monitoring**: CloudWatch + DataDog

#### Staging Environment
- **Compute**: ECS Fargate (1-2 tasks)
- **Database**: RDS PostgreSQL (Single-AZ)
- **Cache**: ElastiCache Redis (Single node)
- **ML**: SageMaker single-model endpoints

## Scalability Considerations

### Horizontal Scaling Strategies

#### Application Layer Scaling
```
Load Balancer → Auto Scaling Group (ECS Tasks)
                      ↓
              Stateless Services → Shared Cache Layer
                      ↓
              Database Connection Pooling → Read Replicas
```

#### Database Scaling
- **Read Scaling**: Aurora read replicas (up to 15 replicas)
- **Write Scaling**: Database sharding by team_id
- **Cache Scaling**: Redis cluster mode with automatic failover
- **Query Optimization**: Materialized views for complex analytics

#### ML Scaling
- **Real-time Predictions**: SageMaker multi-model endpoints with auto-scaling
- **Batch Processing**: SageMaker Processing Jobs with Spot instances
- **Model Serving**: A/B testing with traffic splitting
- **Feature Engineering**: EMR Serverless for large-scale data processing

### Performance Optimization

#### Caching Strategy
```
Browser Cache (Static Assets) → CDN (CloudFront) → API Gateway Cache
                                      ↓
                              Application Cache (Redis) → Database
```

#### Database Optimization
- Indexed queries on frequently accessed columns
- Partitioning large tables by date/team
- Connection pooling with PgBouncer
- Query result caching for expensive operations

#### API Performance
- Response compression (gzip)
- Pagination for large datasets
- Async processing for heavy operations
- Rate limiting to prevent abuse

### Monitoring & Observability

#### Metrics Collection
```
Application Metrics → CloudWatch → DataDog Dashboard
                           ↓
                    Alerts → PagerDuty → Slack Notifications
```

#### Key Performance Indicators
- API response time (p95 < 500ms)
- ML prediction latency (< 2 seconds)
- Database query performance
- Integration sync success rate
- User session duration and engagement

#### Logging Strategy
- Structured logging with correlation IDs
- Centralized log aggregation (CloudWatch Logs)
- Error tracking with Sentry
- Audit trail for compliance

### Security & Compliance

#### Security Measures
- WAF protection against common attacks
- VPC with private subnets for databases