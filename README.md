# Genii AI Production Deployment Environment

This repository contains the production deployment environment for Genii AI services.

## Overview

Genii AI is an enterprise-grade AI agent platform that powers intelligent business automation across multiple domains:

- **Business Operations**: ERPNext integration, multi-company management, automated workflows
- **Sales & Marketing**: Lead generation, pipeline management, content creation
- **Real Estate**: Property analysis, landowner recruitment, deal acceleration
- **Manufacturing**: Partner recruitment, capacity planning, supply chain optimization

## Architecture

This production environment is built with:

- **Runtime**: Node.js 20 LTS
- **Language**: TypeScript
- **Framework**: Express.js
- **Deployment**: Docker containers
- **Infrastructure**: Cloud-native, scalable architecture

## Getting Started

### Prerequisites

- Node.js 20 or higher
- Docker and Docker Compose
- Environment variables (see `.env.example`)

### Installation

```bash
# Clone the repository
git clone https://github.com/davidmschy/genii-ai-prod.git
cd genii-ai-prod

# Install dependencies
npm install

# Copy environment template
cp .env.example .env

# Configure your environment variables
# Edit .env with your settings
```

### Development

```bash
# Run in development mode with hot reload
npm run dev

# Build TypeScript
npm run build

# Run production build
npm start
```

### Docker Deployment

```bash
# Build the Docker image
docker build -t genii-ai-prod .

# Run the container
docker run -p 3000:3000 --env-file .env genii-ai-prod
```

## Project Structure

```
genii-ai-prod/
├── src/
│   ├── index.ts          # Application entry point
│   ├── routes/           # API routes
│   ├── services/         # Business logic
│   ├── middleware/       # Express middleware
│   └── utils/            # Utility functions
├── tests/                # Test suites
├── .env.example          # Environment template
├── Dockerfile            # Container definition
├── package.json          # Dependencies
├── tsconfig.json         # TypeScript configuration
└── README.md             # This file
```

## Environment Variables

See `.env.example` for required configuration. Key variables include:

- `NODE_ENV`: Runtime environment (development/production)
- `PORT`: Server port (default: 3000)
- `API_KEYS`: Service authentication tokens
- `DATABASE_URL`: Database connection string

## Testing

```bash
# Run test suite
npm test

# Run with coverage
npm run test:coverage
```

## Deployment

Production deployment is managed through CI/CD pipelines. Commits to `main` trigger automated:

1. Linting and type checking
2. Test suite execution
3. Docker image build
4. Container registry push
5. Rolling deployment to production cluster

## Monitoring & Operations

- **Health checks**: `/health` endpoint
- **Metrics**: Prometheus-compatible `/metrics`
- **Logging**: Structured JSON logs to stdout
- **Tracing**: Distributed tracing with OpenTelemetry

## Security

- All secrets stored in environment variables or secret management service
- API authentication via bearer tokens
- Rate limiting and DDoS protection
- Regular security audits and dependency updates

## Contributing

This is a production deployment repository. For feature development, see the main platform repositories:

- [genii-platform](https://github.com/davidmschy/genii-platform) - Core platform code
- [erpnext-business-dashboards](https://github.com/davidmschy/erpnext-business-dashboards) - ERPNext integrations

## Support

For issues or questions:
- Email: david@geniinow.com
- Repository Issues: [GitHub Issues](https://github.com/davidmschy/genii-ai-prod/issues)

## License

Proprietary - All rights reserved. Unauthorized copying or distribution is prohibited.

---

**Status**: Production Ready  
**Last Updated**: February 2026  
**Maintained by**: Genii AI Team
