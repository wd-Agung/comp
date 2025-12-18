## Self-hosting Comp (Apps + Portal)

This guide walks you through running the Comp app and portal with Docker.

### Overview

- You will run two services: `app` (primary) and `portal` (customer portal).
- **PostgreSQL** and **MinIO** (S3-compatible storage) are included in the Docker stack for a fully self-contained setup.
- You must provide email (Resend) and Trigger.dev credentials for email login and automated workflows.

### Prerequisites

- Docker Desktop (or Docker Engine) installed
- Resend account and API key for transactional email (magic links, OTP)
- Trigger.dev account and project for automated workflows

### Included Services

The docker-compose stack includes:

- **postgres**: PostgreSQL 16 database with persistent storage
- **minio**: MinIO S3-compatible object storage with web console (port 9001)
- **minio-init**: Automatic bucket creation and configuration
- **api**: NestJS backend API service (port 3333)
- **app**: Main Comp application (port 3000)
- **portal**: Customer portal (port 3002)

### Required environment variables

#### PostgreSQL Configuration

- `POSTGRES_DB` (default: `comp`): Database name
- `POSTGRES_USER` (default: `comp`): Database user
- `POSTGRES_PASSWORD` (default: `comp_password`): Database password - **Change this in production!**

#### MinIO Configuration

- `MINIO_ROOT_USER` (default: `minioadmin`): MinIO admin username - **Change this in production!**
- `MINIO_ROOT_PASSWORD` (default: `minioadmin`): MinIO admin password - **Change this in production!**
- `MINIO_BUCKET_NAME` (default: `comp-storage`): General file storage bucket
- `MINIO_QUESTIONNAIRE_BUCKET` (default: `comp-questionnaires`): Questionnaire uploads bucket
- `MINIO_KNOWLEDGE_BASE_BUCKET` (default: `comp-knowledge-base`): Knowledge base documents bucket
- `MINIO_ORG_ASSETS_BUCKET` (default: `comp-org-assets`): Organization assets bucket

#### API (`apps/api`)

- `DATABASE_URL` (required): Postgres connection string. For local stack: `postgresql://comp:comp_password@postgres:5432/comp`
- `BETTER_AUTH_URL` (required): Base URL of the app server (e.g., `http://localhost:3000`) - used for authentication
- `APP_AWS_REGION` (required): AWS region or `us-east-1` for MinIO
- `APP_AWS_ACCESS_KEY_ID` (required): AWS/MinIO access key
- `APP_AWS_SECRET_ACCESS_KEY` (required): AWS/MinIO secret key
- `APP_AWS_ENDPOINT` (optional): MinIO endpoint `http://minio:9000` (omit for AWS S3)
- `APP_AWS_BUCKET_NAME` (required): S3/MinIO bucket name for general storage
- `RESEND_API_KEY` (required): From Resend dashboard for sending emails
- `TRIGGER_SECRET_KEY` (required): From Trigger.dev project settings
- Optional features:
  - `UPSTASH_VECTOR_REST_URL`, `UPSTASH_VECTOR_REST_TOKEN`: Vector database for AI features
  - `OPENAI_API_KEY`: For AI-powered features
  - `BROWSERBASE_API_KEY`, `BROWSERBASE_PROJECT_ID`: For browser automation

#### App (`apps/app`)

- `DATABASE_URL` (required): Postgres connection string. For local stack: `postgresql://comp:comp_password@postgres:5432/comp`
- `AUTH_SECRET` (required): 32-byte base64. Generate with `openssl rand -base64 32`
- `RESEND_API_KEY` (required): From Resend dashboard
- `REVALIDATION_SECRET` (required): Any random string
- `BETTER_AUTH_URL` (required): Base URL of the app server (e.g., `http://localhost:3000`)
- `NEXT_PUBLIC_BETTER_AUTH_URL` (required): Same as above for client code
- `NEXT_PUBLIC_PORTAL_URL` (required): Base URL of the portal server (e.g., `http://localhost:3002`)
- `NEXT_PUBLIC_API_URL` (required): Base URL of the API server (e.g., `http://localhost:3333` or `http://api:3333` for internal Docker networking)
- `TRIGGER_SECRET_KEY` (required for workflows): From Trigger.dev project settings
- Optional (infrastructure): `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`

#### Portal (`apps/portal`)

- `DATABASE_URL` (required): Same Postgres connection string as app
- `BETTER_AUTH_SECRET` (required): A secret used by portal auth (distinct from app `AUTH_SECRET`)
- `BETTER_AUTH_URL` (required): Base URL of the portal (e.g., `http://localhost:3002`)
- `NEXT_PUBLIC_BETTER_AUTH_URL` (required): Same as portal base URL for client code
- `RESEND_API_KEY` (required): Same Resend key

### Optional environment variables

#### Using MinIO (Local S3-Compatible Storage)

For the included MinIO storage, configure these environment variables:

- **APP_AWS_REGION**: Set to `us-east-1` (MinIO default)
- **APP_AWS_ACCESS_KEY_ID**: Set to `${MINIO_ROOT_USER}` (default: `minioadmin`)
- **APP_AWS_SECRET_ACCESS_KEY**: Set to `${MINIO_ROOT_PASSWORD}` (default: `minioadmin`)
- **APP_AWS_ENDPOINT**: Set to `http://minio:9000` (internal) or `http://localhost:9000` (external)
- **APP_AWS_FORCE_PATH_STYLE**: Set to `true` (required for MinIO)
- **APP_AWS_BUCKET_NAME**: Set to `${MINIO_BUCKET_NAME}` (default: `comp-storage`)
- **APP_AWS_QUESTIONNAIRE_UPLOAD_BUCKET**: Set to `${MINIO_QUESTIONNAIRE_BUCKET}` (default: `comp-questionnaires`)
- **APP_AWS_KNOWLEDGE_BASE_BUCKET**: Set to `${MINIO_KNOWLEDGE_BASE_BUCKET}` (default: `comp-knowledge-base`)
- **APP_AWS_ORG_ASSETS_BUCKET**: Set to `${MINIO_ORG_ASSETS_BUCKET}` (default: `comp-org-assets`)

#### Using AWS S3 (Alternative to MinIO)

If you prefer using AWS S3 instead of MinIO:

- **APP_AWS_REGION**: Your AWS region (e.g., `us-east-1`)
- **APP_AWS_ACCESS_KEY_ID**: AWS access key
- **APP_AWS_SECRET_ACCESS_KEY**: AWS secret key
- **APP_AWS_BUCKET_NAME**: S3 bucket for general uploads
- **APP_AWS_QUESTIONNAIRE_UPLOAD_BUCKET**: S3 bucket for questionnaire uploads
- **APP_AWS_KNOWLEDGE_BASE_BUCKET**: S3 bucket for knowledge base documents
- **APP_AWS_ORG_ASSETS_BUCKET**: S3 bucket for organization assets

#### Other Optional Features

App (`apps/app`):

- **OPENAI_API_KEY**: Enables AI features that call OpenAI models.
- **UPSTASH_REDIS_REST_URL**, **UPSTASH_REDIS_REST_TOKEN**: Optional Redis (Upstash) used for rate limiting/queues/caching.
- **UPSTASH_VECTOR_REST_URL**, **UPSTASH_VECTOR_REST_TOKEN**: Required for vector database operations (questionnaire auto-answer, SOA auto-fill, knowledge base search).
- **NEXT_PUBLIC_POSTHOG_KEY**, **NEXT_PUBLIC_POSTHOG_HOST**: Client analytics via PostHog; leave unset to disable.
- **NEXT_PUBLIC_GTM_ID**: Google Tag Manager container ID for client tracking.
- **NEXT_PUBLIC_LINKEDIN_PARTNER_ID**, **NEXT_PUBLIC_LINKEDIN_CONVERSION_ID**: LinkedIn insights/conversion tracking.
- **NEXT_PUBLIC_GOOGLE_ADS_CONVERSION_LABEL**: Google Ads conversion tracking label.
- **DUB_API_KEY**, **DUB_REFER_URL**: Dub.co link shortener/referral features.
- **FIRECRAWL_API_KEY**: Optional LLM/crawling providers for research features.
- **SLACK_SALES_WEBHOOK**: Slack webhook for sales/lead notifications.
- **GA4_API_SECRET**, **GA4_MEASUREMENT_ID**: Google Analytics 4 server/client tracking.
- **NEXT_PUBLIC_API_URL**: Override client API base URL (defaults to same origin).

Portal (`apps/portal`):

- **NEXT_PUBLIC_POSTHOG_KEY**, **NEXT_PUBLIC_POSTHOG_HOST**: Client analytics via PostHog for portal.
- **UPSTASH_REDIS_REST_URL**, **UPSTASH_REDIS_REST_TOKEN**: Optional Redis if you enable portal-side rate limiting/queues.

### Accessing Services

After starting the stack:

- **App**: http://localhost:3000
- **Portal**: http://localhost:3002
- **API**: http://localhost:3333
  - Swagger documentation: http://localhost:3333/api-docs
  - Health check: http://localhost:3333/v1/health
- **MinIO Console**: http://localhost:9001 (login with `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD`)
- **MinIO API**: http://localhost:9000
- **PostgreSQL**: localhost:5432

### docker-compose.yml uses `.env` (no direct edits needed)

The `docker-compose.yml` includes PostgreSQL and MinIO services with automatic bucket creation. All services are configured via environment variables in `.env` files.

#### `.env` example for local stack with PostgreSQL and MinIO

Create environment files with your values (never commit real secrets):

**Root `.env` (for docker-compose.yml):**

```bash
# PostgreSQL Configuration
POSTGRES_DB=comp
POSTGRES_USER=comp
POSTGRES_PASSWORD=change_this_in_production

# MinIO Configuration
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=change_this_in_production
MINIO_BUCKET_NAME=comp-storage
MINIO_QUESTIONNAIRE_BUCKET=comp-questionnaires
MINIO_KNOWLEDGE_BASE_BUCKET=comp-knowledge-base
MINIO_ORG_ASSETS_BUCKET=comp-org-assets

# App URLs
BETTER_AUTH_URL=http://localhost:3000
BETTER_AUTH_URL_PORTAL=http://localhost:3002
NEXT_PUBLIC_API_URL=http://localhost:3333

# Traefik domains (for production with HTTPS)
# APP_DOMAIN=app.yourdomain.com
# PORTAL_DOMAIN=portal.yourdomain.com
# API_DOMAIN=api.yourdomain.com
```

**`packages/db/.env` (for migrations):**

```bash
# Local PostgreSQL (using docker service name)
DATABASE_URL=postgresql://comp:change_this_in_production@postgres:5432/comp
```

**`apps/api/.env`:**

```bash
# Database (using docker service name)
DATABASE_URL=postgresql://comp:change_this_in_production@postgres:5432/comp

# Auth (required - points to main app for user authentication)
BETTER_AUTH_URL=http://localhost:3000

# MinIO S3-Compatible Storage (required)
APP_AWS_REGION=us-east-1
APP_AWS_ACCESS_KEY_ID=minioadmin
APP_AWS_SECRET_ACCESS_KEY=change_this_in_production
APP_AWS_ENDPOINT=http://minio:9000
APP_AWS_BUCKET_NAME=comp-storage

# Email (required)
RESEND_API_KEY=your_resend_api_key

# Workflows (Trigger.dev hosted)
TRIGGER_SECRET_KEY=your_trigger_secret_key

# Optional AI Features
# OPENAI_API_KEY=
# UPSTASH_VECTOR_REST_URL=
# UPSTASH_VECTOR_REST_TOKEN=

# Optional Browser Automation
# BROWSERBASE_API_KEY=
# BROWSERBASE_PROJECT_ID=
```

**`apps/app/.env`:**

```bash
# Database (using docker service name)
DATABASE_URL=postgresql://comp:change_this_in_production@postgres:5432/comp

# App auth + URLs (required)
AUTH_SECRET=your_generated_secret_here
BETTER_AUTH_URL=http://localhost:3000
NEXT_PUBLIC_BETTER_AUTH_URL=http://localhost:3000
NEXT_PUBLIC_PORTAL_URL=http://localhost:3002
NEXT_PUBLIC_API_URL=http://api:3333
REVALIDATION_SECRET=your_random_secret_here

# Email (required)
RESEND_API_KEY=your_resend_api_key

# Workflows (Trigger.dev hosted)
TRIGGER_SECRET_KEY=your_trigger_secret_key

# MinIO S3-Compatible Storage (local)
APP_AWS_REGION=us-east-1
APP_AWS_ACCESS_KEY_ID=minioadmin
APP_AWS_SECRET_ACCESS_KEY=change_this_in_production
APP_AWS_ENDPOINT=http://minio:9000
APP_AWS_FORCE_PATH_STYLE=true
APP_AWS_BUCKET_NAME=comp-storage
APP_AWS_QUESTIONNAIRE_UPLOAD_BUCKET=comp-questionnaires
APP_AWS_KNOWLEDGE_BASE_BUCKET=comp-knowledge-base
APP_AWS_ORG_ASSETS_BUCKET=comp-org-assets

# Optional
# OPENAI_API_KEY=
# UPSTASH_REDIS_REST_URL=
# UPSTASH_REDIS_REST_TOKEN=
# UPSTASH_VECTOR_REST_URL=
# UPSTASH_VECTOR_REST_TOKEN=
```

**`apps/portal/.env`:**

```bash
# Database (using docker service name)
DATABASE_URL=postgresql://comp:change_this_in_production@postgres:5432/comp

# Portal auth + URLs (required)
BETTER_AUTH_SECRET=your_portal_secret_here
BETTER_AUTH_URL=http://localhost:3002
NEXT_PUBLIC_BETTER_AUTH_URL=http://localhost:3002

# Email (required)
RESEND_API_KEY=your_resend_api_key
```

**Note**: When accessing PostgreSQL or MinIO from within Docker containers, use the service names (`postgres`, `minio`). When accessing from your host machine, use `localhost`.

#### What the `migrator` and `seeder` services do

- **postgres**: PostgreSQL 16 database server with persistent volume storage.
- **minio**: MinIO S3-compatible object storage server with web console.
- **minio-init**: Initialization container that creates required buckets and sets permissions.
- **migrator**: Runs `prisma migrate deploy` using the combined schema from `@trycompai/db`.
  - Purpose: create/update tables, indexes, and constraints in Postgres.
  - Safe to run repeatedly (Prisma applies only pending migrations).

- **seeder**: Generates a Prisma client from the same combined schema and executes the app’s seed script.
  - Purpose: load application reference data (frameworks, controls, relations).
  - Behavior: idempotent upserts by `id`. It does not delete rows; existing rows with matching ids are updated, and relations are connected if missing.
- **api**: NestJS backend API providing REST endpoints for advanced features:
  - Questionnaire parsing and auto-answering
  - Statement of Applicability (SOA) auto-fill
  - Knowledge base document processing
  - Policy AI suggestions
  - Vendor, risk, and asset management
  - Browser-based automations (Browserbase integration)
  - Integration platform (OAuth apps, credential vault)
  - Task automation and execution
- **app**: Main Comp application server (Next.js frontend and API routes).
- **portal**: Customer-facing trust portal.

Notes:

- All services use persistent volumes for data storage (`postgres_data`, `minio_data`).
- The stack automatically waits for PostgreSQL and MinIO to be healthy before starting dependent services.
- The **app** service depends on the **api** service being healthy before starting.
- MinIO buckets are created automatically on first run.

### Trigger.dev (hosted runner)

Trigger.dev powers AI automations and background workflows.

Steps:

1. Create an account at `https://cloud.trigger.dev`
2. Create a project and copy `TRIGGER_SECRET_KEY`
3. From your workstation (not inside Docker):
   ```bash
   cd apps/app
   bunx trigger.dev@latest login
   bunx trigger.dev@latest deploy
   ```
4. Set `TRIGGER_SECRET_KEY` in the `app` service environment.

### Resend (email)

- Create a Resend account and get `RESEND_API_KEY`
- Add a domain if you plan to send emails from a custom domain
- Set `RESEND_API_KEY` in both `app` and `portal` services

### Build & run

#### Prepare environment

Copy the example and fill real values (kept out of git):

```bash
cp .env.example .env
# edit .env with your production secrets and URLs
```

#### Fresh install (optional clean):

```bash
docker compose down --rmi all --volumes --remove-orphans
docker builder prune --all --force
```

#### Build images:

```bash
docker compose build --no-cache
```

#### Start all services:

```bash
docker compose up -d
```

This will:
1. Start PostgreSQL and wait for it to be healthy
2. Start MinIO and create all required buckets
3. Run database migrations
4. Run database seeding
5. Start the API service
6. Start the app and portal (which depend on the API)

#### Verify health:

```bash
# Check API health
curl -s http://localhost:3333/v1/health

# Check app health
curl -s http://localhost:3000/api/health

# Check portal health
curl -s http://localhost:3002/

# Check service status
docker compose ps
```

#### View logs:

```bash
# All services
docker compose logs -f

# Specific service
docker compose logs -f api
docker compose logs -f app
docker compose logs -f portal
docker compose logs -f postgres
docker compose logs -f minio
```

### Production tips

#### Security

- **Change default passwords**: Update `POSTGRES_PASSWORD`, `MINIO_ROOT_USER`, and `MINIO_ROOT_PASSWORD` before deploying
- **Use strong secrets**: Generate secure values for `AUTH_SECRET`, `BETTER_AUTH_SECRET`, and `REVALIDATION_SECRET`
- **Rotate secrets periodically**: Have a process for rotating database passwords and API keys

#### Networking

- Set real domains and HTTPS (behind a reverse proxy / load balancer like Nginx, Traefik, or Caddy)
- Update `BETTER_AUTH_URL`, `NEXT_PUBLIC_BETTER_AUTH_URL`, `NEXT_PUBLIC_API_URL`, and portal equivalents to the public domains
- Don't expose PostgreSQL (port 5432) and MinIO API (port 9000) to the public internet - use internal Docker networking
- For production, the API service should be accessible internally via Docker networking (`http://api:3333`) or externally via reverse proxy (`https://api.yourdomain.com`)
- Consider using MinIO with HTTPS for production

#### Storage & Backups

- Regularly backup the `postgres_data` and `minio_data` volumes
- Monitor disk usage for PostgreSQL and MinIO storage
- Consider using external PostgreSQL and S3 for production if you need managed backups and high availability

#### Scaling

- For production workloads, consider:
  - Using managed PostgreSQL (RDS, DigitalOcean, Neon) for better availability and automated backups
  - Using AWS S3 or similar object storage instead of MinIO for unlimited scalability
  - Running multiple API and app instances behind a load balancer
  - Setting up Redis (Upstash) for caching and session management
  - Enabling Upstash Vector for AI-powered features (questionnaire auto-answer, SOA auto-fill)
  - Using Browserbase for browser automation if running task automations

#### Monitoring

- Set up health check monitoring for all services:
  - API: `http://localhost:3333/v1/health`
  - App: `http://localhost:3000/api/health`
  - Portal: `http://localhost:3002/`
- Monitor PostgreSQL disk usage and query performance
- Monitor MinIO storage usage
- Monitor API response times and error rates (check API logs for performance issues)
- Use logging aggregation for centralized logs (e.g., Loki, ELK)

### Troubleshooting

#### API service fails to start

If the API service fails with Prisma-related errors:

```bash
# Check API logs
docker compose logs api

# Ensure migrations ran successfully
docker compose logs migrator

# Restart the API service
docker compose restart api
```

#### App cannot connect to API

If the app shows errors related to API calls:

1. Verify API is running and healthy:
   ```bash
   curl http://localhost:3333/v1/health
   ```

2. Check `NEXT_PUBLIC_API_URL` is set correctly:
   - For external access: `http://localhost:3333`
   - For internal Docker networking: `http://api:3333`

3. Check API logs for authentication errors:
   ```bash
   docker compose logs api | grep -i error
   ```

#### API service is slow

For performance issues:

1. Enable Upstash Vector for AI features (required for fast questionnaire/SOA operations)
2. Check if OpenAI API key is set (needed for AI-powered features)
3. Monitor API container resources:
   ```bash
   docker stats api
   ```
