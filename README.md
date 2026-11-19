# ŚATOR / SARTOR

**The creator fashion house — discover original drops, build your personal style, and bring designs to life.**

[Live storefront](https://sator-fashion.netlify.app/) · [Customer world](https://sator-fashion.netlify.app/customer) · [Creator studio](https://sator-fashion.netlify.app/creator) · [Architecture](docs/PRODUCT_ARCHITECTURE.md) · [Deployment](docs/NETLIFY_DEPLOYMENT.md)

ŚATOR connects customers, creators, designers and emerging fashion businesses. Customers discover creator collections and save their favourite pieces; professionals build portfolios, collaborate on designs and manage products and orders. The SARTOR 100 programme supports curated creator recognition, with selection controlled by the platform's configured administrators.

**Founders:** Akshaj Reddy and Kamal.

This repository contains the complete current application source: frontend, API routes, database schema and migrations, garment studio assets, tests, documentation and deployment adapters. Production customer records, uploaded files, passwords and service credentials are not source code and are not included.

## What is included

| Area | Features in the application |
| --- | --- |
| Customer world | Creator discovery, searchable collections, product details, saved designs and drops, shopping bag, personal style rooms and lookbooks |
| Creator and designer studio | Profiles, portfolios, designer partnerships, collection publishing, client lookbooks and shared design projects |
| Garment design | Animated 3D garment previews, men's and women's garment choices, fabric colours, text, artwork and branding placement, front/back views, drafts and publishing |
| Community | Posts, reels, follows, likes, comments and in-app notifications |
| Messaging | Private conversations, paginated history, discoverable member search, blocking and retry-safe sends |
| Commerce | Product inventory, size selection, saved delivery addresses, supported COD orders, payment-provider adapters, seller fulfilment updates and order history |
| Customer care | Private order-help threads, return/exchange requests, seller responses and notification updates |
| Account safety | Email/password accounts, recovery keys, protected session cookies, session revocation, server-side ownership checks and request limits |
| Optional AI | Configurable assistance, translation, image-search and speech adapters; provider availability and consent are checked explicitly |

Garment and camera previews are visual design tools, not guarantees of physical fit. SARTOR 100 members, creator partnerships and sales are not fabricated or automatically granted to new accounts.

## Deployment status

The public address is **https://sator-fashion.netlify.app/**.

It is a **Netlify reverse-proxy deployment**. The app runtime and its existing D1 database and R2 uploads remain on the current Sites host. Netlify forwards requests to that app; this is not an independent migration of the backend or customer data to Netlify.

| Target | Runtime and storage | Configuration |
| --- | --- | --- |
| Current public storefront | Netlify proxy → existing Sites app | [`deploy/netlify/`](deploy/netlify/) |
| Current application backend | Vinext / Cloudflare Workers, D1 and R2 | [`vite.config.ts`](vite.config.ts), [`.openai/hosting.json`](.openai/hosting.json) |
| Standalone deployment option | Next.js on Node, SQLite and private persistent disk | [`runtime/node.mjs`](runtime/node.mjs), [`Dockerfile`](Dockerfile), [`compose.yaml`](compose.yaml) |

Publishing the existing Sites app updates what the Netlify address serves. A GitHub push alone does **not** publish that backend: the current Netlify project was deployed with Netlify Drop and is not automatically linked to this repository.

Email users sign in again on the Netlify domain because cookies are scoped to each host. ChatGPT authentication remains owned by the original platform host and can navigate away from the Netlify address.

### Netlify configuration

To reproduce the current storefront, upload the **contents** of `deploy/netlify/` as a zip with `_redirects` and `index.html` at its root. Keep the existing application backend published.

The backend must configure:

```env
SATOR_SECONDARY_ORIGIN=https://sator-fashion.netlify.app
```

Only that exact HTTPS origin is accepted for proxy writes. Other Netlify projects and deploy-preview origins are not allowlisted. Changing the public project name requires updating the backend setting and checking account and shopping writes again.

Do not deploy only `public/` as the application, or place SQLite and customer uploads in a serverless `/tmp` directory. A native Netlify Functions migration requires durable SQL and object-storage adapters. See [the complete Netlify deployment notes](docs/NETLIFY_DEPLOYMENT.md).

## Technology

- **Application:** React 19, TypeScript, Next.js 16 and Vinext/Vite for the Workers target.
- **Interface:** Tailwind CSS, Radix UI, Framer Motion and Lucide icons.
- **3D studio:** Three.js, React Three Fiber, Drei, postprocessing, Zustand and canvas-based graphic tools.
- **Data:** Drizzle ORM and SQLite-compatible storage; D1/R2 when hosted, SQLite/private files when standalone.
- **Optional services:** Payment, text/vision AI and transcription adapters. Credentials and external services are configured separately.
- **Package manager:** `pnpm@11.25.0`, pinned in `package.json`; keep `pnpm-lock.yaml` and `pnpm-workspace.yaml` together.

Exact dependency versions are recorded in [`package.json`](package.json) and [`pnpm-lock.yaml`](pnpm-lock.yaml).

## Run locally with the standalone backend

Use **Node.js 24**; the supplied Docker image pins `24.19.0`.

```bash
git clone https://github.com/yerrakamalkumar1/SARTOR-.git
cd SARTOR-
corepack enable
pnpm install --frozen-lockfile
pnpm run build:node
```

Start on Linux/macOS:

```bash
export SATOR_DATA_DIR="$PWD/data"
export HOSTNAME=0.0.0.0
export PORT=3000
pnpm run start:node
```

Or Windows PowerShell:

```powershell
$env:SATOR_DATA_DIR = Join-Path $PWD 'data'
$env:HOSTNAME = '0.0.0.0'
$env:PORT = '3000'
pnpm run start:node
```

Open `http://localhost:3000`. Use HTTPS for deployed environments and for testing secure-cookie and camera/microphone behaviour on other devices. When a reverse proxy terminates TLS, set `SATOR_PUBLIC_ORIGIN` to the exact external origin, such as `https://your-domain.example`.

`SATOR_DATA_DIR` must be private, writable and persistent. The adapter applies the SQL migrations on initialization. A new standalone instance starts with its own database; it does not copy current production accounts or uploads.

The storage health endpoint is **`/api/health`**. A working runtime returns `{"status":"ok"}`.

### Docker

```bash
docker build -t sator-fashion .
docker volume create sator-data
docker run --rm -p 3000:3000 -v sator-data:/var/data/sator sator-fashion
```

Use a TLS reverse proxy for production. Back up the volume and test restoration before accepting real orders. The standalone SQLite adapter targets one instance with one persistent disk; do not scale replicas with separate copies of the database.

See [`deploy/self-hosted.env.example`](deploy/self-hosted.env.example) and [self-hosting instructions](docs/SELF_HOSTING.md) for optional services, backups and data migration. The retained [`render.yaml`](render.yaml) describes an optional paid service with a persistent disk; it is not the current public deployment.

## Commands and verification

| Command | Purpose |
| --- | --- |
| `pnpm run dev` | Run the Vinext development target |
| `pnpm run typecheck` | Generate route types and check TypeScript |
| `pnpm run test:launch` | Run the 14 isolated launch regression suites |
| `pnpm run test:services` | Run focused service and commerce checks |
| `pnpm run build` | Build the hosted Workers target |
| `pnpm run build:node` | Build the standalone Node target |
| `pnpm run start:node` | Start the built standalone server |
| `pnpm run backup:node -- /private/backups/sator-snapshot` | Back up the configured standalone database and files |

The latest deployment source passed TypeScript, the hosted production build and all 14 launch suites, including the additional exact-origin Netlify checks. Tests cover real route logic and migrations with isolated fixtures; payment and AI providers are simulated. These results do not verify live merchant activation, model quality, device compatibility or production capacity.

For additional focused checks, see [`tests/`](tests/), including the garment, branding, creative collaboration and standalone runtime suites. Do not run hosted and Node builds concurrently because they share generated route types.

## Repository map

| Directory | Contents |
| --- | --- |
| [`app/`](app/) | Pages, styles, customer and professional interfaces |
| [`app/api/`](app/api/) | Accounts, commerce, chat, search, studio, media and service routes |
| [`app/lib/`](app/lib/) | Authentication, access rules, 3D helpers and service adapters |
| [`db/`](db/) | Drizzle schema and database entry point |
| [`drizzle/`](drizzle/) | Versioned SQL migrations and schema snapshots |
| [`runtime/`](runtime/) | Standalone runtime and persistent storage adapters |
| [`public/`](public/) | Images, garment/fabric assets, browser libraries and licence notices |
| [`design-system/`](design-system/) | Design tokens and interaction guidance |
| [`scripts/`](scripts/) | Build, development, validation and backup scripts |
| [`tests/`](tests/) | Automated regression suites and fixtures |
| [`deploy/`](deploy/) | Netlify proxy files and configuration templates |
| [`docs/`](docs/) | Product architecture, operations and feature documentation |

## Before taking real payments or launching at scale

- **Payments:** supply merchant credentials, verify provider sandbox flows and webhooks, and define reconciliation, refunds and fulfilment operations. COD depends on the product's configuration. No live payment or automatic royalty payout is enabled by cloning this repository.
- **AI:** configure the intended provider and models, then verify actual replies, consent, quotas and error handling. See [AI setup](docs/AI_BACKEND.md).
- **Accounts:** email/SMS delivery and ownership verification require separate setup. Current recovery uses a one-time recovery key.
- **Creator selection:** configure `HUB_ADMIN_USER_IDS` with verified application user IDs before official SARTOR 100 selections or awards. See [creator hub notes](docs/CREATOR_HUB.md).
- **Operations:** verify real-account shopping and chat, device/browser behaviour, backup restoration, abuse handling and expected traffic. The current proxy may share upstream network rate limits; validate capacity before a campaign.

Useful guides: [commerce setup](COMMERCE_SETUP.md), [customer care](docs/CUSTOMER_CARE.md), [launch acceptance](docs/LAUNCH_ACCEPTANCE.md), [3D studio](docs/LIVING-ATELIER.md).

## Security and licences

Secrets belong in deployment environment settings. Do not commit API keys, `.env` files, session cookies, customer databases, backups or private uploads. Keep the existing hosting identity when maintaining this app; deploying a separate instance does not confer access to production data.

Third-party software, assets and browser libraries retain their licences; see [`LICENSES.md`](LICENSES.md), [`docs/DEPENDENCY-LICENSES.md`](docs/DEPENDENCY-LICENSES.md) and asset-level notices. Editorial artwork is illustrative and is not evidence of real inventory or creator participation. No blanket open-source licence has been assigned to ŚATOR's application code.
