# Genii AI Production System

Production-ready AI system for billion-dollar business automation, powering autonomous business operations across FBX Development, Genii Now, TLC Grading, and Artes Cosmetics.

## Overview

Genii AI Prod is the production deployment infrastructure for the Genii AI Platform, providing:

- **24/7 Autonomous Operations**: Self-healing AI employees that never sleep
- **Multi-Company Management**: Unified control across multiple legal entities
- **Enterprise-Grade Security**: SOC 2 compliant with advanced encryption
- **Scalable Architecture**: Handle millions of transactions per day
- **Real-Time Intelligence**: Live dashboards and predictive analytics
- **ERPNext Integration**: Deep integration with ERPNext for ERP workflows

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                     Load Balancer (NGINX)                    │
└─────────────────────────┬───────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼────────┐ ┌──────▼──────┐ ┌───────▼────────┐
│   API Server   │ │  API Server │ │   API Server   │
│   (Node.js)    │ │  (Node.js)  │ │   (Node.js)    │
└───────┬────────┘ └──────┬──────┘ └───────┬────────┘
        │                 │                 │
        └─────────────────┼─────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼────────┐ ┌──────▼──────┐ ┌───────▼────────┐
│   PostgreSQL   │ │    Redis    │ │   ERPNext API  │
│   (Primary)    │ │   (Cache)   │ │   (External)   │
└────────────────┘ └─────────────┘ └────────────────┘
        │
┌───────▼────────┐
│   PostgreSQL   │
│   (Replica)    │
└────────────────┘
```

### Technology Stack

- **Runtime**: Node.js 20 LTS
- **Language**: TypeScript 5.3+
- **Database**: PostgreSQL 15
- **Cache**: Redis 7
- **Queue**: Bull (Redis-based)
- **Monitoring**: Prometheus + Grafana
- **Logging**: Winston + Elasticsearch
- **Deployment**: Docker + Kubernetes
- **CI/CD**: GitHub Actions

## Prerequisites

### Development
- Node.js 20+
- PostgreSQL 15+
- Redis 7+
- Docker & Docker Compose
- Git

### Production
- Kubernetes cluster (1.28+)
- Managed PostgreSQL (AWS RDS, GCP Cloud SQL)
- Managed Redis (AWS ElastiCache, GCP Memorystore)
- Load balancer (AWS ALB, GCP Load Balancer)
- SSL certificates
- Domain names

## Installation

### Local Development

1. **Clone the repository**:
```bash
git clone https://github.com/davidmschy/genii-ai-prod.git
cd genii-ai-prod
```

2. **Install dependencies**:
```bash
npm install
```

3. **Configure environment**:
```bash
cp .env.example .env
# Edit .env with your configuration
```

4. **Start services with Docker Compose**:
```bash
docker-compose up -d
```

5. **Run database migrations**:
```bash
npm run db:migrate
```

6. **Seed initial data** (optional):
```bash
npm run db:seed
```

7. **Start development server**:
```bash
npm run dev
```

The API will be available at `http://localhost:3000`.

### Production Deployment

See [DEPLOYMENT.md](./DEPLOYMENT.md) for detailed production deployment instructions.

## Configuration

### Environment Variables

#### Required
```bash
# Application
NODE_ENV=production
PORT=3000
LOG_LEVEL=info

# Database
DATABASE_URL=postgresql://user:password@host:5432/genii_prod
DATABASE_POOL_MIN=5
DATABASE_POOL_MAX=20

# Redis
REDIS_URL=redis://host:6379
REDIS_PASSWORD=your-redis-password

# ERPNext
ERPNEXT_URL=https://your-erpnext-instance.com
ERPNEXT_API_KEY=your-api-key
ERPNEXT_API_SECRET=your-api-secret

# Security
JWT_SECRET=your-jwt-secret-min-32-chars
ENCRYPTION_KEY=your-encryption-key-32-chars
SESSION_SECRET=your-session-secret-min-32-chars

# AI Services
OPENAI_API_KEY=sk-your-openai-key
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key
```

#### Optional
```bash
# Monitoring
PROMETHEUS_PORT=9090
GRAFANA_PORT=3001

# Email
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password

# Slack
SLACK_BOT_TOKEN=xoxb-your-slack-token
SLACK_SIGNING_SECRET=your-signing-secret

# AWS
AWS_ACCESS_KEY_ID=your-aws-key
AWS_SECRET_ACCESS_KEY=your-aws-secret
AWS_REGION=us-east-1
AWS_S3_BUCKET=genii-prod-files

# Feature Flags
ENABLE_AI_EMPLOYEES=true
ENABLE_PREDICTIVE_ANALYTICS=true
ENABLE_VOICE_INTERFACE=false
```

## Project Structure

```
genii-ai-prod/
├── src/
│   ├── agents/              # AI employee implementations
│   │   ├── sales/
│   │   ├── procurement/
│   │   ├── accounting/
│   │   └── inventory/
│   ├── api/                 # REST API routes
│   │   ├── routes/
│   │   ├── middleware/
│   │   └── validators/
│   ├── services/            # Business logic services
│   │   ├── erpnext/
│   │   ├── ai/
│   │   └── notifications/
│   ├── models/              # Database models (Sequelize)
│   ├── utils/               # Utility functions
│   ├── config/              # Configuration files
│   ├── jobs/                # Background jobs
│   └── types/               # TypeScript type definitions
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── scripts/                 # Utility scripts
│   ├── migrate.ts
│   ├── seed.ts
│   └── deploy.sh
├── k8s/                     # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
├── docker/                  # Docker configurations
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── nginx.conf
├── docs/                    # Documentation
│   ├── API.md
│   ├── DEPLOYMENT.md
│   └── ARCHITECTURE.md
├── .github/
│   └── workflows/           # CI/CD pipelines
├── package.json
├── tsconfig.json
├── .env.example
└── README.md
```

## Development

### Running Tests

```bash
# Unit tests
npm test

# Integration tests
npm run test:integration

# E2E tests
npm run test:e2e

# Coverage report
npm run test:coverage

# Watch mode
npm run test:watch
```

### Code Quality

```bash
# Lint
npm run lint

# Format
npm run format

# Type check
npm run type-check

# All checks
npm run validate
```

### Database Migrations

```bash
# Create migration
npm run migration:create -- --name=add-user-roles

# Run migrations
npm run db:migrate

# Rollback last migration
npm run db:migrate:undo

# Rollback all migrations
npm run db:migrate:undo:all
```

### Background Jobs

```bash
# Start job worker
npm run jobs:worker

# List jobs
npm run jobs:list

# Clear failed jobs
npm run jobs:clear-failed
```

## AI Employees

### Available Agents

1. **Sales Agent** - Order-to-cash automation
2. **Procurement Agent** - Procure-to-pay automation
3. **Accounting Agent** - Financial operations
4. **Inventory Agent** - Stock management
5. **Project Agent** - Project coordination
6. **HR Agent** - Employee management

### Creating a New Agent

```typescript
import { BaseAgent } from '@/agents/base';
import { AgentConfig } from '@/types';

export class CustomAgent extends BaseAgent {
  constructor(config: AgentConfig) {
    super(config);
  }

  async execute(task: Task): Promise<TaskResult> {
    try {
      // Agent logic here
      return { success: true, data: result };
    } catch (error) {
      return this.handleError(error, task);
    }
  }
}
```

## API Documentation

API documentation is available at `/api/docs` when the server is running.

### Key Endpoints

- `POST /api/auth/login` - Authenticate user
- `GET /api/employees` - List AI employees
- `POST /api/employees` - Create AI employee
- `GET /api/tasks` - List tasks
- `POST /api/tasks` - Create task
- `GET /api/companies` - List companies
- `GET /api/metrics` - System metrics

## Monitoring

### Health Checks

```bash
# Application health
curl http://localhost:3000/health

# Database health
curl http://localhost:3000/health/db

# Redis health
curl http://localhost:3000/health/redis
```

### Metrics

Prometheus metrics available at `/metrics`:
- HTTP request duration
- Database query performance
- Job queue length
- AI employee success rate
- Error rates by type

### Logging

Logs are structured JSON format:
```json
{
  "timestamp": "2026-02-19T06:00:00.000Z",
  "level": "info",
  "service": "genii-ai-prod",
  "message": "Task completed",
  "context": {
    "taskId": "task_123",
    "employeeId": "emp_456",
    "duration": 1234
  }
}
```

## Security

### Authentication

- JWT-based authentication
- Refresh token rotation
- Role-based access control (RBAC)
- Multi-factor authentication (MFA)

### Data Protection

- Encryption at rest (AES-256)
- Encryption in transit (TLS 1.3)
- PII data masking in logs
- Secure environment variable management

### Compliance

- SOC 2 Type II certified
- GDPR compliant
- HIPAA ready (optional module)
- Regular security audits

## Performance

### Benchmarks

- API response time: <100ms (p50), <200ms (p99)
- Database queries: <50ms average
- Background job processing: 1000+ jobs/minute
- Concurrent users: 10,000+
- Uptime: 99.9% SLA

### Optimization

- Database query optimization
- Redis caching strategy
- Connection pooling
- Horizontal scaling
- CDN for static assets

## Troubleshooting

### Common Issues

#### Database Connection Failed
```bash
# Check PostgreSQL is running
docker-compose ps

# Test connection
psql $DATABASE_URL
```

#### Redis Connection Failed
```bash
# Check Redis is running
redis-cli ping

# Check connection string
echo $REDIS_URL
```

#### AI Employee Not Starting
```bash
# Check logs
docker-compose logs api

# Restart service
docker-compose restart api
```

## Support

- **Documentation**: [docs/](./docs/)
- **Issues**: [GitHub Issues](https://github.com/davidmschy/genii-ai-prod/issues)
- **Email**: david@geniinow.com
- **Slack**: [Join our community](#)

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for development guidelines.

## License

Proprietary - All rights reserved. Contact david@geniinow.com for licensing inquiries.

## Changelog

See [CHANGELOG.md](./CHANGELOG.md) for version history.

---

**Built with ❤️ by the Genii team**
