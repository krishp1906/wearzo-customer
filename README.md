# WEARZO Customer Website

**Purpose:** Customer shopping experience for discovering nearby fashion retailers and placing a single-store order for quick local delivery.

# WEARZO — Project Overview

WEARZO is a web-first hyperlocal fashion quick-commerce marketplace. Customers discover approved nearby fashion retailers, browse products and variants held by those stores, place a **single-store order**, and receive it through either a WEARZO-assigned delivery partner or the store's own delivery operation.

**20-day target:** a narrow, controlled MVP pilot in a limited Ahmedabad service area. This is not a promise of a fully automated, production-scale marketplace.

## Product decisions

- One order belongs to exactly one store; no multi-store cart or split checkout.
- One NestJS backend and one PostgreSQL database serve all four frontend apps.
- Backend is the source of truth for identity, permissions, prices, stock, orders, payments and delivery state.
- Inventory is manually updated, with barcode/SKU-assisted lookup. POS integrations are deferred.
- Delivery is hybrid: platform rider or vendor-managed delivery.
- Customer web is mobile-friendly; delivery app is a mobile-first web app/PWA.
- Rider assignment is manual for the MVP.
- Do not promise 30-minute delivery across Ahmedabad until pilot data proves it.

## System overview

```text
Customer Web ─┐
Vendor Panel ─┤
Delivery PWA ─┼── REST / HTTPS ── Backend API ── PostgreSQL + PostGIS
Admin Panel ──┘                       ├── Razorpay
                                     ├── Cloudinary / S3
                                     └── Notifications
```

Frontends never connect directly to the database and must not implement authoritative business rules.

## Five repositories

| Repository        | Owns                                                                            |
| ----------------- | ------------------------------------------------------------------------------- |
| `wearzo-backend`  | API, domain logic, auth, database, stock, pricing, order/payment/delivery state |
| `wearzo-customer` | Discovery, shopping, cart/checkout UI, order history and tracking UI            |
| `wearzo-vendor`   | Store profile, catalog, inventory and vendor order-operation UI                 |
| `wearzo-delivery` | Assigned jobs, pickup/drop actions and delivery confirmation UI                 |
| `wearzo-admin`    | Vendor approval, order oversight, manual rider assignment and interventions     |

## Shared technical conventions

- TypeScript; Node.js 24 LTS and the same pnpm version on office Linux and home Windows.
- NestJS backend; Next.js App Router for all four frontends.
- PostgreSQL + PostGIS; Prisma ORM.
- REST API prefix `/api/v1`; Swagger/OpenAPI documentation.
- UUID identifiers; money stored as integer paise; timestamps stored in UTC.
- Validate all writes on the backend. Frontend validation is for usability only.
- Keep `.env` and `.env.local` out of Git. Commit `.env.example` with placeholders only.
- No secrets in `NEXT_PUBLIC_` variables.
- One Git repository per app; do not merge the repos into a monorepo.

## 20-day roadmap

| Day | Focus               | Outcome                                                             |
| --: | ------------------- | ------------------------------------------------------------------- |
|   1 | Foundation          | Five app shells, common config, Docker database and health endpoint |
|   2 | Database/API        | Initial Prisma schema, migration, seed and Swagger                  |
|   3 | Auth/access         | Roles, sessions and resource ownership checks                       |
|   4 | Vendor onboarding   | Vendor submits store; admin approves/rejects                        |
|   5 | Catalog             | Products, variants, listings, prices and images                     |
|   6 | Inventory           | Manual stock, movement history and safe reservation primitive       |
|   7 | Customer foundation | Customer sign-in and addresses                                      |
|   8 | Discovery           | Nearby approved stores and product browsing                         |
|   9 | Cart/checkout       | Single-store cart, server-calculated totals and serviceability      |
|  10 | Orders              | Order snapshots, idempotency and inventory reservation              |
|  11 | Payments            | Razorpay test flow, verified webhooks and controlled COD            |
|  12 | Vendor orders       | Accept/reject and preparation workflow                              |
|  13 | Delivery            | Delivery tasks, hybrid mode and manual rider assignment             |
|  14 | Delivery app        | Assigned jobs, pickup/drop, OTP and proof                           |
|  15 | Admin               | Vendor approvals, order oversight and interventions                 |
|  16 | Tracking            | Customer timeline and essential notifications                       |
|  17 | Feature freeze      | End-to-end integration across all five repos                        |
|  18 | Testing             | Critical security, stock, payment and workflow tests                |
|  19 | Deployment          | Apps deployed, secrets/backups configured, pilot data ready         |
|  20 | Pilot               | Limited launch, manual monitoring and measured learning             |

This is an aggressive solo-developer schedule. Cut optional polish/features if behind; do not cut authorization, payment verification, stock correctness or end-to-end testing.

## Daily workflow

1. Pull latest changes.
2. Read the day's task and acceptance criteria.
3. Implement one small vertical slice.
4. Run lint/type checks and relevant tests.
5. Test the actual flow in browser/API.
6. Commit and push working changes.
7. Update README if setup or architecture changed.
8. Record completed work, blockers and tomorrow's first task.

## Security and operations

- Enforce role and resource ownership in backend endpoints.
- Provision admin accounts through a controlled seed/invite process; no public admin signup.
- Verify signed payment webhooks server-side; browser redirects are not proof of payment.
- Deduplicate webhook events and make checkout/order requests idempotent.
- Use database transactions/conditional updates for stock; Redis is not stock authority.
- Do not log passwords, OTPs, tokens or payment secrets.
- Validate uploads and restrict access to delivery proof.
- Keep production secrets in hosting secret settings.
- Pilot with selected stores and a limited zone, with manual support and a way to pause order intake.

## Definition of MVP success

A customer discovers an approved nearby store, chooses a variant, adds items from that one store, places an order, completes verified online payment or supported COD, the vendor accepts/prepares it, a platform rider or vendor delivery completes it, the customer sees accurate status, and admin can intervene. Critical authorization, payment and inventory-safety checks pass.

## Technology

Next.js App Router, TypeScript, Tailwind CSS, shared backend API client; use a consistent server-state solution such as TanStack Query.

## Planned repository structure

```text
wearzo-customer/
├── public/
├── src/
│   ├── app/
│   │   ├── (shop)/             # home, stores, products, cart, checkout, orders
│   │   ├── (account)/          # login, profile, addresses
│   │   ├── layout.tsx
│   │   └── globals.css
│   ├── components/             # layout, store, product, cart, order
│   ├── features/               # auth, discovery, catalog, cart, checkout, orders
│   ├── lib/                    # api-client, auth, formatters
│   ├── hooks/
│   └── types/
├── .env.example
└── README.md
```

This is the intended structure, not a demand to create every folder immediately. Add modules and feature folders progressively on their scheduled implementation days.

## What this application is actually building

Customer can select location/address, sign in, discover approved nearby stores, browse store products and variants, add items from one store only, review server-calculated checkout totals, pay online or use supported COD, and view order history/status timeline.

## Local development

Prerequisites: Node.js 24 LTS, pnpm, Git and a running WEARZO backend where API integration is required.

```bash
pnpm install\ncp .env.example .env.local\npnpm dev -- --port 3000
```

`.env.example`:

```env
NEXT_PUBLIC_API_BASE_URL=http://localhost:4000/api/v1
```

## Local service

Customer website http://localhost:3000

## Implementation notes

Build mobile-first responsive screens: home/discovery, store detail, product selection, cart, checkout, login/account, addresses, orders and order timeline. If location permission is denied, allow manual area/address selection. Do not trust browser totals or payment redirects. Create feature folders as needed, not all empty on Day 1.

## Acceptance checklist

Customer can sign in and manage address; see approved serviceable stores; browse variants and store-specific prices; cart enforces one store; checkout shows backend totals; payment is confirmed by backend; customer sees backend order status.

## Day-by-day contribution

Use the shared 20-day roadmap above as the project source of truth. This repository should implement only the work assigned to its role. Keep business rules in `wearzo-backend`; do not duplicate them in this frontend.
