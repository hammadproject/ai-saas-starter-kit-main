# AI SaaS Platform

A full-stack foundation for building and launching AI-powered SaaS products with authentication, subscription billing, AI image generation, file management, administration, and production-oriented backend services.

## Overview

This project provides a reusable starting point for AI SaaS applications instead of requiring each product to rebuild authentication, billing, storage, user management, and AI infrastructure from scratch.

The application combines a **Next.js frontend** with a **FastAPI backend** and provides:

- User authentication and protected application routes
- User profiles and role-based administration
- Subscription billing with Stripe
- AI text-to-image generation
- S3-compatible object storage through Backblaze B2
- File upload, browsing, preview, download, and deletion
- Usage and activity dashboards
- Administrative data views and audit information
- API health and metrics endpoints
- Automated tests and structural architecture checks

Optional integrations can be enabled independently, allowing the core application to run without configuring every external service.

---
![Sign in screen with password and email-code tabs](docs/images/signin.png)
## Architecture

```text
                    ┌──────────────────────┐
                    │     Next.js Web App  │
                    │  React + TypeScript  │
                    └──────────┬───────────┘
                               │
                         HTTP / API
                               │
                               ▼
                    ┌──────────────────────┐
                    │     FastAPI API      │
                    │ Authentication       │
                    │ Billing              │
                    │ Files                │
                    │ AI Generation        │
                    │ Admin                │
                    └───────┬───────┬──────┘
                            │       │
             ┌──────────────┘       └──────────────┐
             ▼                                     ▼
      ┌──────────────┐                      ┌──────────────┐
      │   Supabase   │                      │ Backblaze B2 │
      │ Auth + DB    │                      │ Object Store │
      └──────────────┘                      └──────────────┘
             │
             │
      ┌──────┴─────────┐
      ▼                ▼
┌────────────┐   ┌───────────────┐
│   Stripe   │   │ NVIDIA NIM    │
│  Billing   │   │ AI Generation│
└────────────┘   └───────────────┘
```
![Dashboard with plan, storage, generation stat cards, an upload-activity chart, and recent generations](docs/images/dashboard.png)

The frontend communicates with the FastAPI service, while authentication and application data are handled through Supabase. Files are stored in Backblaze B2 using its S3-compatible interface. Stripe and NVIDIA NIM are optional integrations for billing and AI generation.

---
![Generate page with a prompt form and a gallery of generated images](docs/images/generate.png)

## Key Features

### Authentication

- Email/password authentication
- Passwordless email-code authentication
- Protected application routes
- User profiles
- Role-based administration
- Server-side authentication checks

### Subscription Billing

Stripe integration provides:

- Checkout
- Billing Portal
- Free, Pro, and Team plan support
- Subscription synchronization through webhooks
- Plan-based feature restrictions

Billing is optional during development. The application can run without configuring Stripe.

### AI Image Generation

The `/generate` workflow accepts a text prompt and generates an image through:

```text
User Prompt
    ↓
FastAPI
    ↓
NVIDIA NIM
    ↓
Image Result
    ↓
Backblaze B2
```

The current implementation uses the NVIDIA NIM `flux.1-dev` model through the Genblaze SDK.

Generated files are stored in B2 and include a SHA-256 provenance manifest.

AI generation is Pro-plan gated in the current application.

### File Management

The application includes a complete file-management workflow:

- Drag-and-drop uploads
- Upload progress
- Browser-based file listing
- File previews
- Downloads
- File deletion
- File metadata
- Checksums
- Image dimensions
- EXIF metadata where available

Uploads can be sent directly from the browser to B2 using presigned requests, avoiding unnecessary transfer through the application server.

### Dashboard

The application dashboard provides an overview of:

- Current subscription plan
- Storage usage
- AI generation activity
- Upload activity
- Recently uploaded files

### Admin Console

Administrators can manage and inspect application resources including:

- Users
- Subscriptions
- Jobs
- Files
- Provider runs
- Storage information
- Audit information

Administrative routes are protected by the application's role system.

### API Reliability & Observability

The backend includes:

- Health checks
- Metrics endpoint
- Structured request logging
- Request tracing
- Rate limiting
- HTTP error handling
- Input validation
- Automated structural checks

---

## Application Workflow

### Authentication

```text
User
  ↓
Sign Up / Sign In
  ↓
Supabase Authentication
  ↓
User Profile
  ↓
Protected Application
```

### File Upload

```text
Browser
  ↓
Request Upload Authorization
  ↓
Presigned B2 Upload
  ↓
Backblaze B2
  ↓
File Metadata Stored
  ↓
File Browser
```

### AI Generation

```text
Prompt
  ↓
Generation API
  ↓
NVIDIA NIM
  ↓
Generated Image
  ↓
B2 Storage
  ↓
Generation History
```

### Billing

```text
User
  ↓
Select Plan
  ↓
Stripe Checkout
  ↓
Stripe Webhook
  ↓
Subscription Record
  ↓
Plan-Based Access
```

---

## Tech Stack

### Frontend

- Next.js 16
- React 19
- TypeScript
- Tailwind CSS v4
- shadcn/ui
- TanStack Query
- Recharts

### Backend

- Python 3.11+
- FastAPI
- Pydantic
- boto3
- httpx
- Pillow
- PyPDF2
- Ruff
- Pytest

### Platform Services

- Supabase — authentication and PostgreSQL
- Backblaze B2 — S3-compatible object storage
- Stripe — subscriptions and payments
- NVIDIA NIM — AI image generation
- Genblaze SDK — AI generation orchestration

### Development & Testing

- pnpm workspaces
- ESLint
- Playwright
- Vitest
- Pytest
- Ruff
- Pre-commit
- GitHub Actions

---

## Project Structure

```text
.
├── apps/
│   └── web/
│       ├── e2e/
│       ├── public/
│       └── src/
│           └── app/
│               ├── (app)/
│               │   ├── account/
│               │   ├── admin/
│               │   ├── billing/
│               │   ├── dashboard/
│               │   ├── design/
│               │   ├── files/
│               │   ├── generate/
│               │   ├── settings/
│               │   └── upload/
│               ├── (auth)/
│               │   ├── signin/
│               │   └── signup/
│               ├── auth/
│               └── ...
│
├── services/
│   └── api/
│       ├── app/
│       ├── tests/
│       ├── main.py
│       ├── pyproject.toml
│       ├── requirements.txt
│       ├── railway.json
│       └── vercel.json
│
├── scripts/
│   ├── configure_b2_cors.py
│   ├── dev.sh
│   ├── doctor.mjs
│   ├── pick-port.mjs
│   ├── sync-stripe-env.mjs
│   └── sync-supabase-env.mjs
│
├── supabase/
│   ├── config.toml
│   └── ...
│
├── docs/
│   ├── features/
│   ├── images/
│   ├── deployment.md
│   ├── design-system.md
│   ├── dev-workflows.md
│   ├── app-workflows.md
│   ├── SECURITY.md
│   └── RELIABILITY.md
│
├── .env.example
├── AGENTS.md
├── ARCHITECTURE.md
├── LICENSE
├── package.json
├── pnpm-lock.yaml
└── pnpm-workspace.yaml
```

---

## Requirements

Before starting development, install:

- Node.js 20 or newer
- pnpm 9 or newer
- Python 3.11 or newer
- Docker for a local Supabase environment
- Supabase CLI
- Stripe CLI if local billing development is required

You will also need accounts or credentials for the services you enable:

- Backblaze B2
- Supabase
- Stripe
- NVIDIA

---

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/hammadproject/ai-saas-starter-kit-main.git
cd ai-saas-starter-kit-main
```

### 2. Install Frontend Dependencies

```bash
pnpm install
```

### 3. Create the Python Environment

```bash
cd services/api
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cd ../..
```

On Windows, activate the environment with:

```powershell
services\api\.venv\Scripts\activate
```

### 4. Configure Environment Variables

Create the local environment file:

```bash
cp .env.example .env
```

Then add the credentials required for your selected services.

---

## Environment Configuration

The project uses a single `.env` file at the repository root.

### Backblaze B2

The following values are required for file storage:

```env
B2_APPLICATION_KEY_ID=
B2_APPLICATION_KEY=
B2_BUCKET_NAME=
B2_REGION=
```

An optional public URL can be configured with:

```env
B2_PUBLIC_URL_BASE=
```

If it is not configured, the application can use short-lived presigned URLs.

### Supabase

Authentication and PostgreSQL require:

```env
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
```

The service-role key is server-only and must never be exposed to browser-side code.

For local development:

```bash
supabase start
node scripts/sync-supabase-env.mjs
```

### NVIDIA AI Generation

AI image generation can be enabled with:

```env
NVIDIA_API_KEY=
```

The application can start without this variable; the generation endpoint remains unavailable until the service is configured.

### Stripe

Billing can be enabled with:

```env
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
STRIPE_PRICE_PRO=
STRIPE_PRICE_TEAM=
```

The project includes scripts for creating the test prices and forwarding Stripe webhooks locally.

### Optional API Configuration

The environment template also supports configuration for:

```env
NEXT_PUBLIC_API_URL=
ENABLE_DOCS=
API_CORS_ORIGINS=
BILLING_SUCCESS_URL=
BILLING_CANCEL_URL=
BILLING_PORTAL_RETURN_URL=
TRUST_PROXY=
METRICS_TOKEN=
AUTH_CACHE_TTL_SECONDS=
```

Only configure these when the corresponding deployment or security requirement applies.

---

## Local Development

Start the complete development environment with:

```bash
pnpm dev
```

The frontend is available at:

```text
http://localhost:3000
```

The API runs at:

```text
http://localhost:8000
```

The development command runs the project's preflight checks before starting the application.

You can also run the checks independently:

```bash
pnpm doctor
```

### Frontend Only

```bash
pnpm dev:web
```

### API Only

```bash
pnpm dev:api
```

---

## Supabase Setup

For local development, start the Supabase stack:

```bash
supabase start
```

Then synchronize the generated local credentials:

```bash
node scripts/sync-supabase-env.mjs
```

The local Supabase environment provides the authentication and database services required by the application.

For hosted environments, use your Supabase project's API credentials and apply the database migrations according to the deployment documentation.

---

## Stripe Setup

Stripe is optional.

To configure local billing:

1. Add a test-mode Stripe secret key to `.env`.
2. Run:

```bash
pnpm stripe:seed
```

3. Start webhook forwarding:

```bash
pnpm stripe:listen
```

4. Restart the application.

The Stripe seed script creates the configured Pro and Team prices and writes their identifiers to the environment file.

For detailed billing configuration, see `docs/stripe-setup.md`.

---

## Production Deployment

The repository contains deployment configuration for the frontend and API.

The documented production setup supports:

- Next.js frontend deployment
- FastAPI backend deployment
- Hosted Supabase
- Stripe live-mode billing
- Backblaze B2 storage

Deployment configuration and environment requirements are documented in:

```text
docs/deployment.md
```

For a production deployment, configure the frontend API URL, CORS origins, Supabase credentials, storage credentials, and any enabled third-party services.

---

## Configuration & Customization

### Application Branding

Application-level branding is centralized in:

```text
apps/web/src/lib/app-config.ts
```

The current configuration exposes:

```ts
APP_NAME
APP_DESCRIPTION
```

Update these values to match your product before deploying.

### Design System

Reusable UI components and design guidance are documented in:

```text
docs/design-system.md
```

The application also exposes a design page during development for reviewing UI primitives.

### Authentication

Authentication behavior can be adapted through the Supabase integration and the application's authentication routes.

Relevant application areas include:

```text
apps/web/src/app/(auth)/
apps/web/src/app/auth/
```

### Billing

Subscription plans and billing behavior are implemented in the billing application and API modules.

The Stripe setup documentation should be used when changing products, prices, webhook behavior, or deployment configuration.

### Storage

The storage layer uses the S3-compatible interface provided by Backblaze B2.

The architecture can therefore be adapted to another S3-compatible provider, although the current project configuration is built and documented around B2.

---

## Testing

The project includes frontend, backend, structural, and end-to-end tests.

### Frontend Tests

```bash
pnpm test:web
```

### Backend Tests

```bash
pnpm test:api
```

### Structural Tests

```bash
pnpm check:structure
```

### End-to-End Tests

```bash
pnpm test:e2e
```

### Type Checking

```bash
pnpm typecheck
```

### Linting

```bash
pnpm lint
pnpm lint:api
```

### Production Build

```bash
pnpm build
```

---

## Useful Commands

| Command | Purpose |
|---|---|
| `pnpm dev` | Start frontend and backend development services |
| `pnpm dev:web` | Start only the Next.js frontend |
| `pnpm dev:api` | Start only the FastAPI backend |
| `pnpm doctor` | Run development environment checks |
| `pnpm build` | Build the web application |
| `pnpm typecheck` | Type-check the frontend |
| `pnpm lint` | Lint the frontend |
| `pnpm lint:api` | Lint the Python backend |
| `pnpm test:web` | Run frontend tests |
| `pnpm test:api` | Run backend tests |
| `pnpm check:structure` | Validate backend architecture rules |
| `pnpm test:e2e` | Run Playwright end-to-end tests |
| `pnpm stripe:seed` | Create/configure Stripe test prices |
| `pnpm stripe:listen` | Forward Stripe webhooks locally |

---

## Security

This project handles authentication credentials, payment configuration, user information, and cloud-storage access. Treat all production credentials and application data as sensitive.

### Never commit

- `.env`
- API keys
- Supabase service-role keys
- Backblaze application keys
- Stripe secret keys
- Stripe webhook secrets
- OAuth credentials
- Production tokens
- Private customer or user data

### Recommended practices

- Use separate credentials for development and production.
- Use least-privilege access wherever possible.
- Keep service-role and secret credentials server-side.
- Restrict production CORS origins.
- Protect the metrics endpoint on public deployments.
- Review generated AI content before using it in customer-facing workflows.
- Keep dependencies updated.
- Follow the security guidance in `docs/SECURITY.md`.

If a credential is accidentally exposed, revoke and replace it immediately.

---

## Reliability & Observability

The backend includes several mechanisms intended to make the application easier to operate:

- Health endpoints
- Metrics
- Structured request logging
- Request tracing
- Rate limiting
- Input validation
- Error handling
- Automated structural tests

Additional reliability guidance is available in:

```text
docs/RELIABILITY.md
```

---

## Documentation

| Document | Purpose |
|---|---|
| `AGENTS.md` | Repository conventions and development guidance |
| `ARCHITECTURE.md` | Application architecture and technical boundaries |
| `docs/deployment.md` | Deployment configuration |
| `docs/SECURITY.md` | Security practices |
| `docs/RELIABILITY.md` | Reliability and operational guidance |
| `docs/app-workflows.md` | Application workflows |
| `docs/dev-workflows.md` | Development and testing workflows |
| `docs/design-system.md` | UI and design-system guidance |
| `docs/stripe-setup.md` | Stripe configuration |
| `docs/features/` | Feature-specific documentation |

---

## Project Status

This repository provides a reusable foundation for AI SaaS applications. Core authentication, file management, administration, billing, and AI-generation workflows are implemented, while deployment and third-party integrations require environment-specific configuration.

Before using the platform in production, configure the required services, review security settings, run the automated test suite, and replace all development credentials and URLs with production values.

---

## Contributing

Contributions and improvements are welcome.

Before submitting a change:

1. Keep the existing project architecture intact unless the change requires an architectural update.
2. Add or update tests for affected functionality.
3. Run the relevant linting and test commands.
4. Update documentation when configuration or behavior changes.
5. Keep secrets and private data out of commits.

---

## License

This project is distributed under the **MIT License**.

See the `LICENSE` file included in the repository for the complete license terms and required copyright notice.
