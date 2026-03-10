# MERN SaaS Launch Roadmap

> **A repeatable blueprint for shipping subscription-based web apps as a solo developer.**

---

## Technology Assignments

### Core Stack

| Layer | Choice | Fit | Alternative Suggestion |
| ----- | ------ | --- | ---------------------- |
| Runtime | Node.js + Express.js | **Best** — lightweight API server built for I/O-heavy workloads (auth, CRUD, Stripe webhooks, Resend calls). Single language across frontend and backend eliminates context-switching for a solo dev. Express is unopinionated enough to port your existing session-based auth pattern directly without fighting framework conventions. | — |
| Database | MongoDB Atlas | **Best** — SaaS user data is document-shaped (user profiles, subscription records, token collections). Schema flexibility lets you iterate fast without migrations. Atlas handles backups, scaling, monitoring, and TTL indexes for auto-cleanup of sessions, login attempts, and email tokens — all critical to your auth pattern. | — |
| ODM | Mongoose | **Best** — schema validation at the application layer catches bad data before it hits the database. Middleware hooks (pre-save password hashing, pre-remove cleanup) centralize logic. Model definitions become reusable across every app — User, Subscription, EmailToken, LoginAttempt are always the same shape. | — |
| Frontend | React + Vite | **Best** — React handles the SPA architecture you need for dashboard-style subscription apps (auth state, protected routes, modals, forms). Vite gives instant dev server startup, sub-second hot reload, and optimized production builds via Rollup. | — |
| Styling | Custom CSS (Tailwind when needed) | **Best** — no framework overhead by default. You write what you need, nothing more. Keeps bundle size minimal and avoids locking every project into a utility-class workflow. Bring in Tailwind only when a project benefits from its structure (rapid prototyping, design-system-heavy UIs). | Tailwind CSS — **Strong** — useful when you need to move fast on UI-heavy apps. Add it per-project, not as a default. |
| Client Routing | React Router | **Best** — industry standard for React SPAs. Supports the nested route pattern you need: public routes (landing, login, pricing), auth-protected routes (dashboard, settings), and subscription-gated routes (premium features). Outlet-based layouts keep the component tree clean. | — |
| Server State | TanStack Query (React Query) | **Best** — eliminates the repetitive useEffect + useState fetch boilerplate that otherwise appears in every component. Built-in caching, automatic background refetching, loading/error states, and mutation invalidation. Handles the auth check on app load and subscription status polling cleanly. | — |
| Client State | Zustand | **Best** — covers the thin layer of global state that TanStack Query doesn't: UI state (sidebar, modals, toasts) and cached auth flags for instant access. 1KB, no boilerplate, works with React without providers or context wrappers. | — |
| Input Validation | Zod | **Best** — lightweight, TypeScript-native schema validation. Define the shape of request body, params, and query strings once, reuse across routes. Key advantage: validation schemas can be shared between Express backend and React frontend, so you define your rules once. Strips unknown fields by default (critical for NoSQL injection prevention). | — |
| Logging | pino | **Best** — structured JSON logging built for Node.js. Fastest logger in the ecosystem (low overhead in production). Structured output means logs are searchable and filterable. pino-pretty for readable dev output, raw JSON for production. | — |

### Auth & Security

| Layer | Choice | Fit | Alternative Suggestion |
| ----- | ------ | --- | ---------------------- |
| Authentication | Session-based (express-session + connect-mongo) | **Best** — server-side sessions with HTTP-only cookies are inherently more secure than JWT for browser-based SPAs. No token stored in localStorage (XSS-proof), no client-side expiration logic, and session revocation is instant (delete from MongoDB). Your entire auth pattern depends on this: password change invalidates all sessions, login lockouts persist across restarts, session regeneration prevents fixation attacks. JWT would require workarounds for every one of those features. connect-mongo keeps sessions in your existing Atlas cluster with no additional dependencies. Running a single Express service means you don't need stateless auth across multiple servers. | — |
| Password Hashing | Node.js crypto.scrypt | **Best** — built into Node, zero external dependencies. Memory-hard hashing function recommended by OWASP. Stronger brute-force resistance than bcrypt because it's tunable on both CPU and memory cost. No native compilation issues since it's part of the Node runtime. | — |
| CSRF Protection | X-header + constant-time comparison | **Best** — double-submit pattern via custom header avoids the complexity of CSRF token middleware libraries while providing equivalent protection. Constant-time comparison prevents timing attacks against the token. | — |
| Rate Limiting | MongoDB sliding window | **Best** — single rate limiting system for everything: general API throttling, login attempt tracking, and account lockouts. Uses your existing Atlas infrastructure with no additional dependencies. Sliding window is more accurate than fixed window. TTL indexes auto-clean expired records. Persistent storage means counters survive server restarts — critical for lockout security. Thresholds are tuned per route: 10 login attempts per 15 min, 5 registrations per hour, 3 password resets per hour, 100 general API calls per minute. | — |
| Bot Protection | Cloudflare Turnstile | **Best** — invisible CAPTCHA alternative that doesn't degrade UX. Free tier is generous. Server-side verification is a single POST to Cloudflare. Applied to all public-facing forms: registration, login, password reset, and any app-specific submission forms. | — |
| Security Headers | Helmet.js | **Best** — single middleware call sets all recommended HTTP security headers (X-Frame-Options, X-Content-Type-Options, Strict-Transport-Security, CSP, etc.). Zero configuration for the defaults, customizable when needed. | — |
| CORS | cors (Express middleware) | **Best** — even with a single-service deployment (Express serves the SPA), CORS is a defense-in-depth layer that restricts which origins can call your API. Simple configuration, keeps your security posture tight if you ever split services later. | — |
| Admin Role | Database role column | **Best** — `role` field on the User model with values `user` (default) and `owner`. Industry standard RBAC pattern used by every major framework. No redeploy to transfer ownership — update one document. Queryable, auditable, and scales naturally if you ever add a third role. The owner authenticates through the normal login flow like every other user. | — |

### Payments & Email

| Layer | Choice | Fit | Alternative Suggestion |
| ----- | ------ | --- | ---------------------- |
| Billing | Stripe Checkout (hosted) + Webhooks | **Best** — hosted checkout pushes PCI compliance entirely to Stripe. You never touch card numbers. Webhook-driven subscription lifecycle is Stripe's recommended pattern and keeps your billing state in sync without polling. Customer Portal gives users self-service subscription management for free. | — |
| Transactional Email | Resend + HTML template literals | **Best** — Resend has the cleanest developer experience of any email API (single function call to send). Plain HTML template literals keep the stack lean with zero extra build tooling. Sufficient for transactional emails (verify, reset, receipt). Covers verification, password reset, payment receipts, and onboarding drip sequences. | React Email — **Strong** — add per-project if you need complex branded email templates with component reuse. Adds build dependencies. |

### Infrastructure & Monitoring

| Layer | Choice | Fit | Alternative Suggestion |
| ----- | ------ | --- | ---------------------- |
| Hosting | Render (single service) | **Best** — Express serves the built React SPA and the API from one web service. One URL, one deployment, one bill. Blueprint-as-code (render.yaml) makes deployment reproducible across apps. Auto-deploy from GitHub on push to main. Free SSL via Let's Encrypt. No infrastructure management overhead for a solo dev. Split into two services only if you need independent scaling or a CDN for static assets. | — |
| Env Management | dotenv | **Best** — standard pattern for loading environment variables from .env files in development. Production uses Render's built-in env var management. The .env.example file in your template repo documents every required variable for each new app. | — |
| Error Tracking | Sentry | **Best** — captures runtime errors in both frontend and backend with full stack traces, breadcrumbs, and user context. Alerts you to issues before users report them. The free tier covers a solo dev's volume easily. Performance monitoring at 10% sample rate gives you baseline metrics without cost overhead. | — |
| Analytics | PostHog | **Best** — product analytics that goes far beyond traffic tracking. Feature usage, retention cohorts, session replays, churn funnels, and custom events. Free tier (1M events/month) is generous for a solo dev. Tells you *how* users behave inside your app, not just *that* they visited. Self-hosted option available if you want full data ownership. | — |
| CI/CD | GitHub Actions | **Best** — runs tests on every PR, verifies the production build succeeds, and gates merges to main. Combined with Render's auto-deploy on push, this gives you a complete pipeline with zero infrastructure to manage. Free tier covers the volume of a solo dev easily. | — |
| Version Control | GitHub (template repo) | **Best** — the template repo feature is purpose-built for your workflow. "Use this template" creates a fresh repo with full commit history isolation. Every new app starts from the same proven boilerplate without inheriting git history from previous projects. | — |

### Development Tools

| Layer | Choice | Fit | Alternative Suggestion |
| ----- | ------ | --- | ---------------------- |
| IDE | Visual Studio Code | **Best** — dominant editor for web development. Extensions for ESLint, Prettier, Tailwind IntelliSense (when needed), MongoDB, and GitLens. Integrated terminal for running dev servers. Free. | — |
| AI Assistant | Claude Code | **Best** — runs in your terminal alongside VS Code. Understands your full codebase context, generates and edits files directly, and can run tests and lint. Purpose-built for the agentic coding workflow a solo dev needs — scaffold a new feature, write tests, debug, and commit without leaving the terminal. | — |

### Edge Case Tools (per-app as needed)

| Layer | Choice | Fit | Alternative Suggestion |
| ----- | ------ | --- | ---------------------- |
| Canvas / Drawing | Raw Canvas 2D API | **Best** — zero dependencies, full pixel-level control, no abstraction overhead. The `useCanvas` hook in your template handles DPI scaling and cleanup. Reach for this whenever an app needs drawing, annotations, image manipulation, or visual editing. | — |
| Large File Storage | MongoDB GridFS | **Strong** — stores files larger than MongoDB's 16MB document limit directly in your existing Atlas cluster. No second service to manage, no separate billing, no CORS configuration. Combined with multer for uploads. | AWS S3 or Cloudflare R2 — **Best** — purpose-built for object storage. Better performance for high-volume file serving and files over 100MB. Switch when file storage becomes a core feature rather than a supporting one. |
| Markdown Processing | marked | **Best** — Node-native Markdown parser that stays in your JS-only stack. Fast, extensible, and well-maintained. Always paired with DOMPurify to sanitize rendered HTML. | — |
| Audio Encoding | ffmpeg (system binary) | **Best** — the industry standard for audio/video processing. Pre-installed on Render's build images, so no installation step. Wrapped in a promise-based service layer for clean async usage in Express. For files over 50MB, combine with a background job queue (BullMQ) instead of processing inline. | — |
| Extended Bot Protection | Cloudflare Turnstile | **Best** — same tool as the core stack, extended to additional surfaces. Reusable `TurnstileWidget.jsx` component drops into any form that faces the public internet: contact forms, waitlist signups, user-generated content submissions, file upload endpoints. | — |

---

## Phase 0: GitHub Template Repo

Before building any app, you maintain a single **template repository** that contains the entire boilerplate. Every new project starts with "Use this template" on GitHub.

### Repo Structure

```
saas-boilerplate/
├── client/                     # React + Vite frontend
│   ├── public/
│   │   └── robots.txt
│   ├── src/
│   │   ├── api/                # TanStack Query hooks & Axios instance
│   │   │   ├── axiosInstance.js
│   │   │   ├── useAuth.js
│   │   │   └── useSubscription.js
│   │   ├── components/
│   │   │   ├── auth/           # Login, Register, ForgotPassword, ResetPassword
│   │   │   ├── layout/         # Navbar, Footer, Sidebar, ProtectedRoute, OwnerRoute
│   │   │   ├── payments/       # PricingTable, SubscriptionStatus
│   │   │   └── ui/             # Shared buttons, modals, inputs
│   │   ├── pages/
│   │   │   ├── Landing.jsx
│   │   │   ├── Login.jsx
│   │   │   ├── Register.jsx
│   │   │   ├── Dashboard.jsx
│   │   │   ├── Settings.jsx
│   │   │   ├── Pricing.jsx
│   │   │   ├── NotFound.jsx        # 404 page
│   │   │   └── admin/          # AdminDashboard, AdminUsers (owner-only)
│   │   ├── store/              # Zustand stores
│   │   │   └── useAuthStore.js
│   │   ├── utils/
│   │   │   └── turnstile.js
│   │   ├── hooks/
│   │   │   └── useCanvas.js    # Raw Canvas 2D hook (delete if not needed)
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── index.css           # Custom CSS (add Tailwind config if needed per-project)
│   ├── index.html
│   ├── vite.config.js
│   └── package.json
│
├── server/                     # Express.js backend
│   ├── config/
│   │   ├── db.js               # Mongoose connection
│   │   ├── session.js          # express-session config
│   │   ├── logger.js           # Pino logger setup
│   │   ├── stripe.js           # Stripe client init
│   │   ├── resend.js           # Resend client init
│   │   ├── turnstile.js        # Turnstile verification helper
│   │   └── gridfs.js           # GridFS bucket init (delete if not needed)
│   ├── models/
│   │   ├── User.js
│   │   ├── LoginAttempt.js     # TTL-indexed lockout tracking
│   │   ├── EmailToken.js       # TTL-indexed verification/reset tokens
│   │   ├── ProcessedEvent.js   # TTL-indexed Stripe webhook deduplication
│   │   └── Session.js          # connect-mongo session store
│   ├── validators/
│   │   ├── auth.js             # Zod schemas: register, login, reset
│   │   ├── user.js             # Zod schemas: profile update, delete account
│   │   └── billing.js          # Zod schemas: checkout session creation
│   ├── middleware/
│   │   ├── validate.js         # Zod validation middleware
│   │   ├── errorHandler.js     # Centralized error handler
│   │   ├── requestLogger.js    # Pino HTTP request logging
│   │   ├── requireAuth.js      # Session-based auth check
│   │   ├── requireOwner.js     # Owner role check (user.role === 'owner')
│   │   ├── requireSubscription.js  # Stripe subscription gate
│   │   ├── csrfProtection.js   # X-header CSRF with constant-time comparison
│   │   ├── rateLimiter.js      # MongoDB sliding window rate limiter
│   │   └── validateTurnstile.js    # Turnstile server-side verification
│   ├── routes/
│   │   ├── auth.js             # Register, login, logout, verify, reset
│   │   ├── user.js             # Profile, settings, password change, export, delete
│   │   ├── admin.js            # Owner-only admin endpoints
│   │   ├── billing.js          # Stripe checkout session, portal, webhooks
│   │   ├── email-webhooks.js   # Resend bounce/complaint webhook handler
│   │   ├── files.js            # GridFS upload/download (delete if not needed)
│   │   ├── sitemap.js          # Dynamic sitemap.xml generation
│   │   └── health.js           # Health check endpoint for Render
│   ├── services/
│   │   ├── emailService.js     # Resend send helpers (checks bounce/suppress flags)
│   │   ├── stripeService.js    # Stripe business logic
│   │   └── audioService.js     # ffmpeg wrappers (delete if not needed)
│   ├── emails/                 # HTML email template functions
│   │   ├── verifyEmail.js
│   │   ├── resetPassword.js
│   │   ├── paymentReceipt.js
│   │   └── welcomeOnboard.js
│   ├── utils/
│   │   ├── asyncHandler.js     # Async route error wrapper
│   │   ├── timingSafeCompare.js
│   │   ├── generateToken.js
│   │   └── markdown.js         # marked + DOMPurify (delete if not needed)
│   ├── __tests__/              # Jest test suites
│   │   ├── auth.test.js
│   │   ├── validation/
│   │   │   └── auth.test.js    # Input validation edge cases
│   │   ├── webhooks.test.js    # Stripe webhook idempotency tests
│   │   └── middleware.test.js
│   ├── app.js                  # Express app setup + graceful shutdown
│   └── package.json
│
├── scripts/
│   └── promote-owner.js        # One-time CLI script to assign owner role
├── .github/
│   ├── workflows/
│   │   └── ci.yml              # CI pipeline (hard deploy gate)
│   └── dependabot.yml          # Automated dependency updates
├── .env.example                # Template environment variables
├── .node-version               # Pin Node.js version for Render (e.g., 20.18.0)
├── .gitignore
├── render.yaml                 # Render Blueprint (IaC)
└── README.md
```

### Template Repo Workflow

1. **Click "Use this template"** on GitHub to create a new repo.
2. **Clone it locally**, run `npm install` in both `/client` and `/server`.
3. **Copy `.env.example` to `.env`** and fill in your Atlas, Stripe, Resend, and Turnstile keys.
4. **Rename the app** — update `package.json` name fields, Render service names, and Stripe product references.
5. **Strip the placeholder pages** you don't need. Keep all the auth, billing, and security scaffolding.
6. **Start building your app-specific features.**

### README.md Template

Every app gets a README. The template repo includes a pre-structured one — update the placeholders per project.

```markdown
# [App Name]

[One-line description of what this app does.]

## Tech Stack

- **Frontend:** React + Vite
- **Backend:** Express.js + Node.js
- **Database:** MongoDB Atlas + Mongoose
- **Auth:** Session-based (express-session + connect-mongo)
- **Payments:** Stripe Checkout + Customer Portal
- **Email:** Resend + HTML template literals
- **Bot Protection:** Cloudflare Turnstile
- **Hosting:** Render (single service)

## Getting Started

### Prerequisites

- Node.js 20+
- MongoDB Atlas account
- Stripe account
- Resend account
- Cloudflare account (Turnstile)

### Setup

1. Clone the repo:
   ```bash
   git clone https://github.com/[your-username]/[repo-name].git
   cd [repo-name]
   bash
   npm install
   cd client && npm install && cd ..
   bash
   cp .env.example .env
   bash
   npm run dev
   
client/          # React + Vite frontend
server/          # Express.js backend
  config/        # DB, session, Stripe, Resend, logger configs
  models/        # Mongoose schemas
  validators/    # Zod validation schemas
  middleware/    # Auth, CSRF, rate limiting, error handling
  routes/        # API endpoints
  services/      # Business logic (email, Stripe)
  emails/        # HTML email template functions
  __tests__/     # Test suites


### .gitignore

```gitignore
# Dependencies
node_modules/
client/node_modules/

# Environment
.env
.env.local
.env.production

# Build output
client/dist/

# OS files
.DS_Store
Thumbs.db

# IDE
.vscode/
!.vscode/settings.json
!.vscode/extensions.json
*.swp
*.swo

# Logs
logs/
*.log
npm-debug.log*

# Testing
coverage/
client/coverage/

# MongoDB backups (local only — never commit)
backup-*/

# Misc
.cache/
```

### .env.example

This file is committed to the repo. It documents every required variable without exposing secrets.

```env
# Server
NODE_ENV=development
PORT=5000
SESSION_SECRET=
LOG_LEVEL=debug
APP_URL=http://localhost:5000
APP_NAME=

# MongoDB
MONGODB_URI=

# Stripe
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
STRIPE_PRICE_ID_MONTHLY=
STRIPE_PRICE_ID_YEARLY=

# Resend
RESEND_API_KEY=
EMAIL_FROM=

# Cloudflare Turnstile
TURNSTILE_SECRET_KEY=

# Sentry
SENTRY_DSN=

# PostHog
VITE_POSTHOG_KEY=
VITE_POSTHOG_HOST=

# Admin (optional — one-time seed, remove after first deploy)
SEED_OWNER_EMAIL=
```

---

## Phase 1: Project Setup & Infrastructure

### 1.1 — Create the Repo

- Create a new repo from your GitHub template.
- Clone locally, install dependencies.
- Copy `.env.example` → `.env` and fill in placeholder values.

### 1.2 — MongoDB Atlas Setup

- Create a new Atlas project for this app.
- Create an M0 (free) cluster to start — upgrade to M10+ when you approach launch.
- Set up a database user with least-privilege access.
- Whitelist Render's outbound IPs (or use `0.0.0.0/0` for dev, restrict for prod).
- Create the following collections with TTL indexes:

### 1.3 — Environment Variables

Copy `.env.example` (documented in Phase 0) to `.env` and fill in real values. Here's what they look like populated:

```env
# Server
NODE_ENV=development
PORT=5000
SESSION_SECRET=<generate-a-64-char-random-string>
LOG_LEVEL=debug
APP_URL=http://localhost:5000
APP_NAME=MyApp

# MongoDB
MONGODB_URI=mongodb+srv://<user>:<pass>@<cluster>.mongodb.net/<dbname>

# Stripe
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PRICE_ID_MONTHLY=price_...
STRIPE_PRICE_ID_YEARLY=price_...

# Resend
RESEND_API_KEY=re_...
EMAIL_FROM=noreply@yourdomain.com

# Cloudflare Turnstile
TURNSTILE_SECRET_KEY=0x...

# Sentry
SENTRY_DSN=https://...@sentry.io/...

# PostHog
VITE_POSTHOG_KEY=phc_...
VITE_POSTHOG_HOST=https://us.i.posthog.com

# Admin (optional — one-time seed, remove after first deploy)
SEED_OWNER_EMAIL=you@yourdomain.com
```

---

## Phase 2: Authentication System

This is the core of every app. Port your battle-tested Flask auth pattern directly to Express.

### 2.1 — Session Configuration

Configure `express-session` with `connect-mongo` as the session store. Sessions stored in your existing Atlas cluster. Cookie settings: `httpOnly: true`, `secure: true` in production, `sameSite: 'lax'`, 24-hour max age. Match TTL between the session store and the cookie expiry.

### 2.2 — Password Hashing (crypto.scrypt)

Zero external dependencies. Uses Node's built-in `crypto` module.

`timingSafeEqual` prevents timing attacks on password comparison. The salt is stored alongside the hash, separated by `:`.

### 2.3 — Auth Flow Summary

```
Registration
├── Validate input
├── Verify Cloudflare Turnstile token (server-side)
├── Check if email already exists (timing-safe)
├── Hash password with crypto.scrypt (Node native)
├── Create user in MongoDB (emailVerified: false)
├── Generate verification token → store in email_tokens (TTL: 24h)
├── Send verification email via Resend
└── Return success (do NOT auto-login)

Login
├── Validate input
├── Verify Cloudflare Turnstile token
├── Check login_attempts for lockout status
│   ├── ≥10 attempts → hard lockout (return generic error)
│   └── ≥5 attempts → cooldown period (return generic error)
├── Find user by email
├── Compare password with crypto.scrypt (timing-safe)
├── Check emailVerified === true
├── Regenerate session (prevent fixation)
├── Reset login_attempts for this email
└── Return user data + set session cookie

Logout
├── Destroy server-side session
└── Clear session cookie

Password Reset
├── User submits email
├── Generate reset token → store in email_tokens (TTL: 1h)
├── Send reset email via Resend
├── User clicks link → validate token
├── Hash new password, update user
├── Delete all sessions for this user (force re-login everywhere)
└── Delete the used token

Password Change (authenticated)
├── Verify current password
├── Hash new password, update user
├── Delete all OTHER sessions for this user
└── Regenerate current session
```

### 2.4 — CSRF Protection

**CRITICAL: CSRF + CORS must work together.** The double-submit cookie pattern relies on the browser's same-origin policy to prevent cross-site requests from reading the cookie. However, if your CORS configuration is too permissive (e.g., `origin: '*'`), an attacker on another domain can make requests with credentials and bypass the custom header check. Your CORS must strictly whitelist only your own frontend origin:

Never use `origin: '*'` or `origin: true` in production. A single misconfiguration here undermines your entire CSRF defense.

### 2.5 — Rate Limiting (MongoDB Sliding Window)

Use a MongoDB collection to track request counts per IP per time window. On each request: push the current timestamp, filter out expired ones, check count against threshold. Thresholds per route: login (10 per 15 min), registration (5 per hour), password reset (3 per hour), general API (100 per minute). TTL index auto-cleans expired records.

---

## Phase 3: Cloudflare Turnstile Integration

### 3.1 — Frontend (React Component)

Place the Turnstile widget on login, registration, and password reset forms.

### 3.2 — Backend Verification

Server-side verification: POST the client token to `https://challenges.cloudflare.com/turnstile/v0/siteverify` with your `TURNSTILE_SECRET_KEY`. If `success === false`, reject the request. Apply to: registration, login, password reset, and any public-facing forms.

---

## Phase 4: Stripe Checkout Integration

### 4.1 — Stripe Setup

- Create your Stripe account and enable test mode.
- Create a Product with Monthly and Yearly Price objects.
- Set up the Customer Portal for self-service subscription management.

### 4.2 — Checkout Flow

```
User clicks "Subscribe" on Pricing page
├── Frontend calls POST /api/billing/create-checkout-session
├── Backend creates a Stripe Checkout Session:
│   ├── mode: 'subscription'
│   ├── customer_email: user's email (or existing Stripe customer ID)
│   ├── price: STRIPE_PRICE_ID_MONTHLY or _YEARLY
│   ├── success_url: /dashboard?session_id={CHECKOUT_SESSION_ID}
│   └── cancel_url: /pricing
├── Backend returns the Checkout Session URL
├── Frontend redirects user to Stripe-hosted checkout
└── Stripe handles payment, then redirects back to success_url
```

### 4.3 — Webhook Handler (Idempotent)

```
POST /api/billing/webhook (raw body, no JSON parsing)
├── Verify Stripe signature using STRIPE_WEBHOOK_SECRET
├── Check idempotency:
│   ├── Look up event.id in a processed_events collection
│   ├── If found → return 200 immediately (already handled)
│   └── If not found → process the event, then store event.id with TTL (30 days)
├── Handle events:
│   ├── checkout.session.completed
│   │   └── Link Stripe customer ID to your user, set subscription active
│   ├── invoice.paid
│   │   └── Update subscription period, send receipt via Resend
│   ├── invoice.payment_failed
│   │   └── Flag user, send payment failure email
│   ├── customer.subscription.updated
│   │   └── Handle plan changes, update user record
│   └── customer.subscription.deleted
│       └── Revoke access, update user record
└── Return 200 immediately (process async if needed)
```

**Why idempotency matters:** Stripe can (and will) send the same webhook multiple times — on network timeouts, retries, or their own infrastructure hiccups. Without deduplication, you'll double-credit subscriptions, send duplicate receipts, or corrupt user records.

### 4.4 — Subscription Gating Middleware

Create a `requireSubscription` middleware that checks `user.subscriptionStatus === 'active'` or `'trialing'`. If not, return 403 with `{ error: 'subscription_required' }`. Frontend catches this and redirects to the Pricing page.

### 4.5 — Customer Portal

```
User clicks "Manage Subscription" in Settings
├── Backend calls stripe.billingPortal.sessions.create()
├── Returns portal URL
└── Frontend redirects — user can cancel, update payment, switch plans
```

---

## Phase 5: Resend Email System

### 5.1 — Email Types

| Email                    | Trigger                                   | Template              |
| ------------------------ | ----------------------------------------- | --------------------- |
| Email Verification       | User registers                            | verifyEmail.js        |
| Password Reset           | User requests reset                       | resetPassword.js      |
| Welcome / Onboarding     | User verifies email                       | welcomeOnboard.js     |
| Payment Receipt          | Stripe `invoice.paid` webhook             | paymentReceipt.js     |
| Payment Failed           | Stripe `invoice.payment_failed` webhook   | paymentFailed.js      |
| Subscription Cancelled   | Stripe `customer.subscription.deleted`    | subCancelled.js       |

### 5.2 — HTML Email Templates

Each email template is a function that returns an HTML string. No build step, no extra dependencies.

Follow this same pattern for all templates. Keep the styling inline (email clients ignore external CSS). Keep the structure simple — a heading, a short message, a call-to-action button, and a footer note.

### 5.3 — Resend Service

Create an email service module that initializes the Resend client with your API key. Export helper functions for each email type (e.g., `sendVerificationEmail`, `sendResetEmail`, `sendReceiptEmail`). Each function generates the HTML string from its template function and sends via `resend.emails.send()`. Check `emailBounced` and `emailSuppressed` flags before sending.

### 5.4 — Domain Configuration

- Add your sending domain in Resend dashboard.
- Add the DNS records (DKIM, SPF, DMARC) to your domain registrar.
- Verify the domain before going to production.

### 5.5 — Bounce & Complaint Handling

If you ignore bounces and complaints, your sender reputation degrades and emails start hitting spam for all users.

```
Resend Webhook Events to Handle
├── email.bounced
│   ├── Hard bounce → mark email as undeliverable in your User model
│   ├── Stop sending to this address immediately
│   └── Flag user in dashboard (prompt to update email)
├── email.complained (spam report)
│   ├── Immediately suppress all future sends to this address
│   └── Log for monitoring — high complaint rates trigger Resend account review
└── email.delivered
    └── Optional: update send status for audit trail
```

Check `emailBounced` and `emailSuppressed` flags before sending any email. Never send to a suppressed address.

---

## Phase 6: Frontend Architecture

### 6.1 — Routing Structure

Define your routes in `App.jsx` using React Router. Structure: public routes (landing, pricing, login, register, forgot-password, reset-password, verify-email), protected routes wrapped in `ProtectedRoute` (dashboard, settings), subscription-gated routes wrapped in `SubscriptionGate` (app features), owner-only routes wrapped in `OwnerRoute` (admin dashboard, admin users), and a `*` catch-all for the 404 page.

### 6.2 — API Layer (TanStack Query + Axios)

Create an Axios instance with `baseURL` pointed at your API, `withCredentials: true` to send session cookies, and interceptors for 401 (redirect to login) and 403 `subscription_required` (redirect to pricing). Build TanStack Query hooks: `useAuth` for current user queries and login/register/logout mutations, `useSubscription` for subscription status and checkout/portal session creation.

### 6.3 — Zustand Auth Store

Minimal global state for what TanStack Query doesn't cover: `isAuthenticated` flag (for instant access without waiting for query), UI state (sidebar open/closed, modal state, toast queue).

### 6.4 — Landing Page Strategy

Every app needs a landing page that converts visitors to signups. Standard structure:

```
Landing Page
├── Hero section — headline, subheadline, CTA button
├── Problem/solution — what pain does this solve?
├── Feature highlights — 3-4 key features with icons
├── Social proof — testimonials, user count, logos (when available)
├── Pricing section — embedded PricingTable component
├── FAQ — 4-6 common questions
└── Footer — links, legal pages, social links
```

Build this once in the template. Swap copy and images per app.

### 6.5 — 404 Page

Keep it simple. Include a link back to the landing page. Match the app's visual style. The `<Route path="*">` catch-all in `App.jsx` handles any URL that doesn't match a defined route.

### 6.6 — robots.txt

```
// client/public/robots.txt
User-agent: *
Allow: /
Disallow: /dashboard
Disallow: /settings
Disallow: /admin
Disallow: /app

Sitemap: https://yourdomain.com/sitemap.xml
```

Lives in `client/public/` so Vite includes it in the build output. Tells search engines to index your public pages (landing, pricing, legal) but not authenticated pages (dashboard, settings, admin, app features). Update the domain per project.

### 6.7 — sitemap.xml

Generated dynamically by Express so it always reflects your actual public routes. Only include pages you want search engines to index — never authenticated routes. Mount this route **before** the SPA catch-all in `app.js`.

### 6.8 — SEO Meta Tags & Open Graph

Every public page needs proper meta tags for search engines and social media previews. Since your app is a React SPA, set these in `client/index.html` as defaults, then override per-page with a lightweight head manager.

**Default meta tags in `index.html`:**

```html
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>AppName — one-line tagline</title>
  <meta name="description" content="A clear, keyword-rich description of what your app does. 150-160 characters." />

  <!-- Open Graph (Facebook, LinkedIn, Slack previews) -->
  <meta property="og:type" content="website" />
  <meta property="og:title" content="AppName — one-line tagline" />
  <meta property="og:description" content="Same description as above." />
  <meta property="og:image" content="https://yourdomain.com/og-image.png" />
  <meta property="og:url" content="https://yourdomain.com" />

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:title" content="AppName — one-line tagline" />
  <meta name="twitter:description" content="Same description as above." />
  <meta name="twitter:image" content="https://yourdomain.com/og-image.png" />
</head>
```

**Per-page overrides:** Use `react-helmet-async` to set page-specific titles and descriptions on your landing, pricing, and legal pages. Authenticated pages don't need SEO — they're blocked by `robots.txt`.

**Open Graph image:** Create a 1200×630px image for each app. This is what shows up when someone shares your link on social media. Keep it in `client/public/og-image.png`.

**Checklist per project:**
- Update `<title>` and `<meta name="description">` with app-specific copy
- Update all `og:` and `twitter:` tags
- Create and place `og-image.png` (1200×630px)
- Test with https://opengraph.xyz or Facebook's Sharing Debugger

### 6.9 — Accessibility Baseline

You don't need a full WCAG audit before launch, but cover the basics that affect the most users:

```
Accessibility baseline:
├── Semantic HTML — use <nav>, <main>, <button>, <form>, not divs for everything
├── Keyboard navigation — every interactive element reachable with Tab, activatable with Enter/Space
├── Focus indicators — never remove :focus outlines without replacing them
├── Alt text — every <img> has a descriptive alt attribute
├── Form labels — every input has an associated <label> (not just placeholder text)
├── Color contrast — text meets WCAG AA minimum (4.5:1 for normal text, 3:1 for large text)
├── Error messages — form validation errors are announced to screen readers (aria-live or role="alert")
└── Skip to content — add a "Skip to main content" link as the first focusable element
```

**Tools:** Run Lighthouse accessibility audit in Chrome DevTools before every launch. Aim for 90+ score. Fix any errors rated "critical" or "serious." Run it on your landing page and login page at minimum.

---

## Phase 7: Deployment on Render

### 7.1 — Why Deploys Fail (Common Pitfalls)

Most deployment failures on Render come from the same handful of issues. Fix these before your first deploy and you'll avoid the majority of problems.

**1. Missing or wrong Node.js version**

Render's default Node version may not match what you develop with. If your `package-lock.json` was generated with Node 20 but Render uses a different version, dependencies can fail to install or behave differently.

**Fix:** Always pin your Node version. Add a `.node-version` file to the root of your repo:

```
20.18.0
```

This is the highest priority method Render uses to detect your Node version. You can also set `NODE_VERSION=20.18.0` as an environment variable, or use the `engines` field in `package.json`:

```json
{
  "engines": {
    "node": ">=20.0.0 <21.0.0"
  }
}
```

Use all three for maximum clarity. Always include an upper bound — Render warns against unbounded ranges like `>=20` because they'll silently upgrade to Node 21+ when it releases.

**2. Wrong build command**

Render runs your build command from the repo root. For your single-service setup (Express serves the React SPA), the build command must install dependencies for both server and client, then build the React app.

**Fix:** Use a multi-step build command:

```
npm install && cd client && npm install && npm run build
```

This installs server dependencies, then client dependencies, then builds the React app to `client/dist/`. If you skip `cd client && npm install`, Vite won't have its dependencies and the build fails.

**3. Wrong start command**

Render needs to know how to start your Express server. `npm start` only works if your `package.json` has a `start` script defined.

**Fix:** Either define the script in your root `package.json`:

```json
{
  "scripts": {
    "start": "node server/app.js"
  }
}
```

Or use the explicit command in Render: `node server/app.js`. Never use `nodemon` or `vite dev` — those are development tools, not production starters.

**4. Express not binding to the correct port**

Render forwards traffic to your app on the port defined by the `PORT` environment variable. The default is `10000`. If your Express app hardcodes `5000` or any other port, Render can't reach it and the deploy fails with a port detection error.

**Fix:** Always use `process.env.PORT`:

Bind to `0.0.0.0`, not `localhost` — Render requires this to route traffic to your service.

**5. Missing environment variables**

Your app works locally because `.env` exists. Render doesn't have your `.env` file. Every variable must be set manually in the Render dashboard or defined in `render.yaml`.

**Fix:** Set every env var in the Render dashboard before deploying. Use `.env.example` as your checklist — every variable listed there must exist in Render.

**6. `package-lock.json` not committed**

If `package-lock.json` is in `.gitignore`, Render runs `npm install` without a lockfile. This means dependency versions can drift from what you tested locally.

**Fix:** Always commit `package-lock.json`. Never add it to `.gitignore`.

### 7.2 — Render Blueprint (`render.yaml`)

```yaml
services:
  - type: web
    name: myapp
    runtime: node
    region: oregon
    plan: starter
    buildCommand: npm install && cd client && npm install && npm run build
    startCommand: node server/app.js
    healthCheckPath: /api/health
    envVars:
      - key: NODE_ENV
        value: production
      - key: NODE_VERSION
        value: "20.18.0"
      - key: MONGODB_URI
        sync: false
      - key: SESSION_SECRET
        sync: false
      - key: STRIPE_SECRET_KEY
        sync: false
      - key: STRIPE_WEBHOOK_SECRET
        sync: false
      - key: RESEND_API_KEY
        sync: false
      - key: TURNSTILE_SECRET_KEY
        sync: false
      - key: SENTRY_DSN
        sync: false
      - key: SEED_OWNER_EMAIL
        sync: false  # Optional — remove after first deploy
```

`sync: false` means the value must be set manually in the Render dashboard — it won't be committed to your repo (which is correct for secrets).

### 7.3 — Express Serves the SPA

Note: If you're using ES modules (`"type": "module"` in `package.json`), `__dirname` is not available by default. The `fileURLToPath` workaround above handles this.

### 7.4 — Required Files in Repo Root

```
Files Render needs in your repo root:
├── package.json          # With "start" script and "engines.node" field
├── package-lock.json     # ALWAYS commit this — never .gitignore it
├── .node-version         # Pin your Node.js version (e.g., "20.18.0")
├── render.yaml           # Blueprint for infrastructure-as-code
├── server/               # Your Express app
└── client/               # Your React + Vite app
    ├── package.json      # Client dependencies (React, Vite, etc.)
    ├── package-lock.json # Client lockfile — commit this too
    └── vite.config.js    # Vite build configuration
```

### 7.5 — Root `package.json` Setup

Your root `package.json` should look like this:

```json
{
  "name": "myapp",
  "version": "1.0.0",
  "type": "module",
  "engines": {
    "node": ">=20.0.0 <21.0.0"
  },
  "scripts": {
    "start": "node server/app.js",
    "dev": "concurrently \"cd server && npm run dev\" \"cd client && npm run dev\"",
    "build": "cd client && npm run build"
  },
  "dependencies": {
    "express": "^4.21.0",
    "mongoose": "^8.0.0"
  }
}
```

Key points: `"type": "module"` enables ES module imports. The `engines` field tells Render which Node version to use (backup to `.node-version`). The `start` script is what Render calls.

### 7.6 — Deployment Verification (First Deploy)

After your first deploy, verify everything in this order:

```
1. Check Render dashboard → Events tab for build logs
   ├── Build command ran successfully (both npm installs + Vite build)
   ├── Start command ran (server started, no crash)
   └── Health check passed at /api/health

2. Test the live URL
   ├── Visit https://yourapp.onrender.com → landing page loads (React SPA)
   ├── Navigate to /login → React Router handles it (no 404 from Express)
   ├── Visit /api/health → returns { "status": "ok", "db": "connected" }
   ├── Check browser console → no CORS errors, no failed API calls
   └── Visit a nonexistent route → 404 page renders

3. Test authenticated flows
   ├── Register a new account
   ├── Check email (Resend delivering via verified domain)
   ├── Login → session cookie set correctly (Secure, HttpOnly, SameSite)
   └── Stripe Checkout → test card works, webhook fires

4. If something fails, check in this order:
   ├── Render build logs (Events tab) → did the build complete?
   ├── Render runtime logs (Logs tab) → did the server crash on start?
   ├── Environment variables → are all required vars set?
   ├── Port binding → is Express using process.env.PORT?
   └── MongoDB Atlas → is Render's outbound IP whitelisted?
```

### 7.7 — Render-Specific Best Practices

**Health checks:** Render pings your health check path periodically. If it returns a non-200 response, Render flags your service as unhealthy. Your `/api/health` endpoint should verify the MongoDB connection is alive.

**Auto-deploy:** By default, Render deploys on every push to `main`. Combined with your GitHub Actions CI pipeline, the flow is: push → tests run → merge to main → Render deploys. If CI fails, the merge is blocked, so Render only sees tested code.

**Zero-downtime deploys:** On paid plans, Render runs the new version alongside the old one. Once the new version passes the health check, traffic switches over. If the health check fails, the deploy rolls back automatically.

**Rollbacks:** If a deploy breaks production, you can roll back to any previous deploy from the Render dashboard. This is instant — no rebuild required.

**Logs:** View real-time logs from the Render dashboard under the Logs tab. Your pino structured logging makes these searchable and filterable.

**Outbound IPs:** If you've restricted MongoDB Atlas to specific IPs, you need to whitelist Render's outbound IPs. Find them in Render's docs under "Outbound IP Addresses." For development, `0.0.0.0/0` works but restrict it for production.

**Free tier limitations:** Free web services sleep after 15 minutes of inactivity. The first request after sleep takes 30-60 seconds (cold start). Upgrade to Starter ($7/mo) before launch to eliminate this.

### 7.8 — Deployment Checklist

- [ ] `.node-version` file exists in repo root with exact version (e.g., `20.18.0`)
- [ ] `engines.node` field in root `package.json` matches
- [ ] `package-lock.json` committed (both root and client)
- [ ] `"start"` script defined in root `package.json`
- [ ] Express binds to `process.env.PORT` on `0.0.0.0`
- [ ] Build command installs both server and client dependencies
- [ ] All environment variables set in Render dashboard
- [ ] `NODE_ENV=production` set in Render
- [ ] MongoDB Atlas IP whitelist includes Render's outbound IPs
- [ ] Health check endpoint responding at `/api/health`
- [ ] Custom domain configured
- [ ] Auto-SSL enabled (Let's Encrypt — automatic on Render)
- [ ] Stripe webhook endpoint pointing to production URL
- [ ] `render.yaml` committed to repo

---

## Phase 8: Testing Strategy (Hard Deploy Gate)

**Tests are not optional. No test suite, no deploy.** GitHub Actions enforces this — if tests fail, the PR cannot merge and Render never sees the code.

### 8.1 — Backend (Express)

**Tools:** Jest + Supertest

```
Priority test coverage:
├── Auth flows — register, login, logout, verify, reset (highest priority)
├── Input validation — malformed payloads, missing fields, boundary values, injection attempts
├── Stripe webhook handler — mock webhook events, verify DB updates, test idempotency (duplicate events)
├── Middleware — requireAuth, requireSubscription, rateLimiter, validate
├── Rate limiting — verify lockout thresholds, verify counter persistence
├── CSRF — verify rejection of requests without valid token
├── Error handling — verify no stack traces leak to client, verify error response format
└── Account deletion — verify all user data is removed, verify Stripe cancellation
```

### 8.2 — Input Validation Tests

This is its own category because it's the most common source of security bugs.

Write equivalent tests for every route that accepts input: login, password reset, profile update, checkout session creation.

### 8.3 — Frontend (React)

**Tools:** Vitest + React Testing Library

```
Priority test coverage:
├── Auth forms — validation messages display correctly, submit disabled until valid
├── Protected routes — redirect behavior when not authenticated
├── Subscription gate — redirect when subscription inactive
├── API error handling — 401/403 interceptor behavior, error toast display
└── Account settings — export and delete flows render confirmation modals
```

### 8.4 — Testing Rules for a Solo Dev

1. **Auth, payments, and input validation** — these are the three areas where bugs cost you users, money, or security. Test them thoroughly.
2. **Webhook handler** — test idempotency specifically. Fire the same event twice, verify the database state doesn't double-apply.
3. **Happy path E2E** — one end-to-end test: register → verify → subscribe → access feature → cancel.
4. **Validation edge cases** — test every boundary (min/max length, empty strings, null values, wrong types). These are the bugs that slip through manual testing.
5. **Never skip tests to ship faster.** The CI pipeline won't let you, and that's by design.

---

## Phase 9: CI/CD Pipeline (Tests Block Deploys)

### 9.1 — GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test-server:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: cd server && npm ci
      - run: cd server && npm audit --audit-level=high
      - run: cd server && npm test

  test-client:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: cd client && npm ci
      - run: cd client && npm audit --audit-level=high
      - run: cd client && npm test
      - run: cd client && npm run build  # Verify production build succeeds
```

### 9.2 — Branch Protection Rules

Configure on GitHub → Settings → Branches → `main`:

- Require pull request before merging.
- Require status checks to pass: `test-server` and `test-client`.
- No direct pushes to main.

This makes the test gate **absolute** — there is no way to deploy untested code.

### 9.3 — Pipeline Flow

```
Push to feature branch → GitHub Actions runs tests + audit
├── Tests pass + no high/critical vulnerabilities → PR is mergeable
├── Merge to main → Render auto-deploys
├── Tests fail → PR blocked, fix before merging
└── npm audit fails → PR blocked, update vulnerable packages
```

---

## Phase 10: Monitoring & Error Tracking

### 10.1 — Sentry Setup

**Backend:**

**Frontend:**

### 10.2 — PostHog Analytics

**Frontend setup:**

**Identify users after login:**

**Track key conversion events:**

**Reset on logout:**

**What PostHog gives you that GA doesn't:** session replays (watch exactly where users get stuck), feature flags (roll out features to a percentage of users), retention cohorts (which users come back and why), and funnels built from your custom events.

### 10.3 — Health Monitoring

Set Render's health check to hit this endpoint. You'll get alerts if your server goes down.

---

## Phase 11: Legal Pages

### 11.1 — Required Pages

**Stripe will not let you go live without a Privacy Policy and Terms of Service.** This is the only true launch blocker outside of code — everything else improves reliability, but legal pages are a hard gate for accepting payments.

Every SaaS app needs:

- **Terms of Service** — governs use of your application. Required by Stripe for live mode.
- **Privacy Policy** — required by Stripe, by GDPR (EU), and by CCPA (California). Explains what data you collect (email, payment info via Stripe, cookies, analytics) and how you use it.
- **Cookie Policy** — if using PostHog and session cookies, you need this.

### 11.2 — Approach

- Use a generator (Termly, Iubenda, or similar) to create a baseline.
- Customize for your specific data practices.
- Host as static pages in your React app (`/terms`, `/privacy`, `/cookies`).
- Link from the footer on every page and from the registration form.
- **Have a lawyer review before launch** if your app handles sensitive data.

---

## Phase 12: Edge Case Tooling

These are the specialized tools you reach for when a specific app requires capabilities beyond the standard stack. They're not in every app, but when you need them, the decision is already made.

### 12.1 — Canvas Rendering: Raw Canvas 2D API

**When:** Any app that needs drawing, image manipulation, charts from scratch, annotations, or visual editors.

**Why raw Canvas 2D over a library:** Zero dependencies, full pixel-level control, no abstraction overhead. Libraries like Fabric.js or Konva add convenience but also add bundle size and opinions you don't need for most use cases.

**Standard pattern:**

**Template repo location:** Include `useCanvas.js` in `client/src/hooks/` — delete if not needed per app.

### 12.2 — Large File Storage: MongoDB GridFS

**When:** Any app that needs to store files larger than 16MB (MongoDB's document size limit). Common use cases: user-uploaded images, audio files, PDFs, exports.

**Why GridFS over S3:** Keeps everything in your existing MongoDB Atlas infrastructure. No second service to manage, no separate billing, no CORS configuration. For a solo dev, fewer moving parts wins. Move to S3 only if you hit GridFS performance limits at scale.

**Standard pattern:**

**Express route pattern:**

**Additional dependency:** `multer` for handling multipart file uploads.

### 12.3 — Markdown Processing: marked

**When:** Any app where users write or consume Markdown — blog platforms, documentation tools, note-taking apps, content editors with preview.

**Standard pattern:**

**Why marked:** Node-native, fast, extensible, well-maintained, and stays in your JS-only stack. Supports GFM (GitHub Flavored Markdown) out of the box including tables, strikethrough, and task lists.

**Security note:** Always sanitize rendered Markdown HTML before sending to the frontend. User-generated Markdown is an XSS vector. `isomorphic-dompurify` works on both server and client.

### 12.4 — MP3 Encoding: ffmpeg

**When:** Any app that processes audio — podcast tools, voice note apps, music utilities, transcription preprocessors.

**Render deployment note:** ffmpeg is available on Render's native environments. You do not need to install it separately — it's included in Render's build images.

**Standard pattern:**

**Integration with GridFS:** For apps that both store and process audio, combine ffmpeg with GridFS:

```
User uploads audio file
├── multer receives file → temp directory
├── ffmpeg converts to MP3 → temp directory
├── Upload MP3 to GridFS → get fileId
├── Delete temp files
└── Return fileId to client
```

**Processing considerations:**

- Always process audio asynchronously — never block the Express request/response cycle for long conversions.
- For files over 50MB, consider a background job queue (BullMQ + Redis) instead of processing inline.
- Set reasonable file size limits in multer to prevent abuse.

### 12.5 — Bot Protection: Cloudflare Turnstile (Extended)

Turnstile is already in the standard stack for auth forms, but some apps need it on additional surfaces.

**Extended use cases:**

```
Standard (always on)
├── Registration form
├── Login form
└── Password reset form

App-specific (add as needed)
├── Contact / support forms
├── Public-facing submission forms (e.g., user-generated content)
├── API endpoints exposed without auth (e.g., waitlist signup)
├── Rate-limited actions (combine Turnstile + rate limiter for extra protection)
└── File upload endpoints (prevent bot-driven storage abuse)
```

**Reusable Turnstile React component:**

### Edge Case Decision Matrix

| Need                          | Tool                  | npm Package / Binary  | Notes                                    |
| ----------------------------- | --------------------- | --------------------- | ---------------------------------------- |
| 2D drawing / image editing    | Canvas 2D API         | None (browser-native) | Use `useCanvas` hook                     |
| File storage > 16MB           | MongoDB GridFS        | `mongodb` (built-in)  | Add `multer` for uploads                 |
| Markdown → HTML               | marked                | `marked`                | Always sanitize with `isomorphic-dompurify` |
| Audio conversion to MP3       | ffmpeg                | System binary         | Pre-installed on Render                  |
| Bot protection (extended)     | Cloudflare Turnstile  | None (script tag)     | Reuse `TurnstileWidget.jsx` component    |

---

## Phase 13: Input Validation & Sanitization

Every Express route that accepts user input must validate it before processing. No exceptions.

### 13.1 — Zod Validation Middleware

Create a `validate(schema)` middleware that calls `schema.parse(req.body)`. Zod strips unknown fields by default and throws a `ZodError` with descriptive messages on failure. On error, return 400 with `{ error: 'Validation failed', details: [...] }`. On success, replace `req.body` with the parsed (sanitized) value and call `next()`.

### 13.2 — Schema Definitions

Define Zod schemas for every route that accepts input. Each schema should enforce strict types (`z.string()`, never `z.any()`) to prevent NoSQL injection. Use `.email()` for email fields, `.min()/.max()` for length boundaries, `.trim()` and `.toLowerCase()` for normalization. Group schemas by route file: `validators/auth.js` (register, login, reset), `validators/user.js` (profile update, delete account), `validators/billing.js` (checkout session).

**Shared schemas:** These same Zod schemas can be imported on the frontend to validate form input before it's sent to the API. Define once, enforce everywhere.

### 13.3 — Usage in Routes

Import the `validate` middleware and your Zod schemas in each route file. Stack `validate(registerSchema)` before the route handler on every endpoint that accepts input. The validated and sanitized body is available as `req.body` in the handler.

### 13.4 — Validation Rules by Route Category

```
Auth Routes
├── Registration: email (valid, lowercase, trimmed), password (8-128 chars), name (1-100 chars, trimmed), turnstileToken
├── Login: email, password, turnstileToken
├── Password reset request: email
├── Password reset confirm: token, new password (8-128 chars)
└── Password change: currentPassword, newPassword (8-128 chars)

Billing Routes
├── Create checkout session: priceId (must match known price IDs)
└── Webhook: raw body (no Zod — Stripe signature verification handles this)

User Routes
├── Update profile: name (1-100 chars, trimmed), email (valid, lowercase)
└── Delete account: password (confirmation)

File Upload Routes (if using GridFS)
├── Upload: multer handles file validation (mimetype, size limit)
└── Download: fileId (valid MongoDB ObjectId format)
```

### 13.5 — CRITICAL: NoSQL Injection Prevention

MongoDB is immune to SQL injection but vulnerable to NoSQL injection. An attacker can send `{"email": {"$gt": ""}}` instead of a string, and MongoDB will happily query with it — potentially bypassing authentication.

**Your defense is Zod's `z.string()`.** When you define `email: z.string()`, Zod will reject any value that isn't a string. An object like `{"$gt": ""}` fails validation before it ever reaches Mongoose.

**This only works if every field in every schema has an explicit type.** If you accidentally use `z.any()` or skip validation on a field, you've opened a NoSQL injection vector.

**Defense in depth:** Even with Zod, add Mongoose's `sanitizeFilter` option as a second layer:

This tells Mongoose to strip `$`-prefixed operators from query filters, catching anything that slips past validation.

---

## Phase 14: Centralized Error Handling

### 14.1 — Express Error Handler

Create an error-handling middleware (`err, req, res, next`) that: logs the full error server-side via pino (message, stack, method, URL, userId), handles Mongoose `ValidationError` (400), Mongoose duplicate key error code 11000 (409), Zod `ZodError` (400), and defaults to 500 for everything else. Never leak stack traces to the client — return `'Internal server error'` for 500s.

### 14.2 — Async Route Wrapper

Express doesn't catch async errors by default. Wrap every async route handler:

### 14.3 — Registration Order in app.js

Middleware registration order matters. From top to bottom: (1) Sentry request handler, (2) session/helmet/cors middleware, (3) Stripe webhook route with raw body parser (before `express.json()`), (4) `express.json()` for all other routes, (5) API routes (auth, user, billing), (6) admin routes (`/api/admin/*`), (7) sitemap route, (8) SPA catch-all serving React build, (9) Sentry error handler, (10) custom `errorHandler` (must be last).

---

## Phase 15: Graceful Shutdown

Render sends SIGTERM on every deploy. Without handling it, in-flight requests get dropped and database connections hang.

---

## Phase 16: Structured Logging

### 16.1 — Pino Setup

Create a logger module that initializes pino with `level` from `LOG_LEVEL` env var (default `'info'`). In development, use `pino-pretty` transport for readable output. In production, output raw JSON (no transport).

### 16.2 — Request Logging Middleware

Use `pino-http` middleware with your logger instance. Set custom log levels: 500+ errors → `'error'`, 400+ → `'warn'`, everything else → `'info'`. Exclude health check requests from logging to reduce noise.

### 16.3 — What to Log

```
Always log:
├── Errors with full stack traces and request context
├── Auth events: login success/failure, registration, password reset, session invalidation
├── Payment events: checkout created, webhook received, subscription status changes
├── Rate limit triggers: IP, endpoint, current count
└── Startup and shutdown events

Never log:
├── Passwords (even hashed)
├── Session tokens or secrets
├── Full credit card numbers (Stripe handles this, but be cautious)
└── Health check requests (noise)
```

---

## Phase 17: Database Indexing Strategy

### 17.1 — Required Indexes

Create indexes for: TTL cleanup (email tokens, login attempts, sessions, processed events), unique user lookups (`email` unique, `stripeCustomerId` sparse), email token lookups (`token`, `userId`), rate limiting lookups (`ip` + `endpoint`), and Stripe event deduplication (`eventId` unique). Define indexes in your Mongoose schemas so they're created automatically on connection.

### 17.2 — Mongoose Model Indexes

Define indexes in your Mongoose schemas so they're created automatically:

### 17.3 — Monitoring Index Performance

- Use Atlas Performance Advisor (free on M10+) to identify slow queries and missing indexes.
- On M0 (free tier), check `db.collection.getIndexes()` and review query patterns manually.
- Rule of thumb: if a query runs more than once per request and filters on a field, that field needs an index.

---

## Phase 18: Backup & Disaster Recovery

### 18.1 — Atlas Backup Configuration

| Atlas Tier | Backup Type | Frequency | Retention |
| ---------- | ----------- | --------- | --------- |
| M0 (free)  | None — Atlas does not backup free clusters | — | — |
| M2/M5      | Daily snapshots | Every 24 hours | Varies by tier |
| M10+       | Continuous backup + point-in-time recovery | Continuous | Configurable |

**Recommendation:** Start on M0 for development. Upgrade to M10 before launch. The cost is worth it for continuous backups and point-in-time recovery.

### 18.2 — Manual Backup Strategy (M0)

If you're on the free tier during development:

### 18.3 — Disaster Recovery Checklist

```
If your database is corrupted or data is lost:
├── M10+: Restore from Atlas point-in-time recovery (pick exact timestamp)
├── M0: Restore from most recent mongodump
├── Stripe data: Stripe is your source of truth for billing — resync from Stripe API
├── Sessions: Users will need to log in again (acceptable)
└── Email tokens: Expired tokens are regenerated on request (acceptable)
```

### 18.4 — What You Cannot Lose

```
Critical data (must be backed up):
├── User accounts (email, hashed password, profile)
├── Subscription records (stripeCustomerId, subscription status)
└── App-specific user data (whatever your SaaS product stores)

Regenerable data (loss is inconvenient but not catastrophic):
├── Sessions → users log in again
├── Email tokens → users request new verification/reset
├── Login attempts → rate limits reset (brief window of exposure)
└── Processed webhook events → small risk of duplicate processing
```

---

## Phase 19: Dependency Security

### 19.1 — GitHub Dependabot

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/server"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5

  - package-ecosystem: "npm"
    directory: "/client"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5
```

### 19.2 — npm Audit in CI

Add to your GitHub Actions workflow:

```yaml
- run: cd server && npm audit --audit-level=high
- run: cd client && npm audit --audit-level=high
```

This fails the build if any high or critical vulnerabilities are found.

### 19.3 — Dependency Review Workflow

```
Weekly Dependabot PRs arrive
├── Review the changelog for breaking changes
├── CI runs tests automatically
├── If tests pass → merge
├── If tests fail → investigate before merging
└── npm audit --audit-level=high catches newly disclosed vulnerabilities
```

---

## Phase 20: Data Export & Account Deletion (GDPR)

Required by GDPR (EU), CCPA (California), and an increasing number of privacy laws worldwide. Even if your initial audience is US-only, build this in from the start — retrofitting it is painful.

### 20.1 — Data Export

Create a `GET /api/user/export` endpoint (requires auth) that fetches the user's account data (excluding password hash), active session count, and any app-specific data. Return as a downloadable JSON file with `Content-Disposition: attachment` header.

### 20.2 — Account Deletion

Create a `DELETE /api/user/account` endpoint (requires auth) that: requires password confirmation, cancels any active Stripe subscription, deletes all user data (user document, sessions, email tokens, login attempts, app-specific data, GridFS files if applicable), destroys the current session, and returns a confirmation. This must be thorough — GDPR requires complete data removal.

### 20.3 — Frontend

- Add "Export My Data" button in Settings → downloads JSON file.
- Add "Delete My Account" button in Settings → confirmation modal → requires password → calls API → redirects to landing page.

---

## Phase 21: Owner Role (Admin Access)

Two roles: `user` and `owner`. The role is stored as a field on the User document in MongoDB. This is the industry-standard RBAC pattern — the same approach used by Django, Rails, Laravel, and every major framework.

### 21.1 — How It Works

```
User signs up → normal registration flow → role defaults to "user"
You promote a user to owner via a one-time CLI script or Atlas query
On every admin request:
├── requireAuth checks session (is this person logged in?)
├── requireOwner checks role === 'owner' on the user document
├── Both pass → admin access granted
└── Either fails → 403 Forbidden

Transfer ownership:
├── Set new user's role to 'owner'
├── Set old user's role back to 'user'
└── Takes effect immediately — no redeploy needed
```

### 21.2 — User Model

Add a `role` field to your User schema: `{ type: String, enum: ['user', 'owner'], default: 'user' }`. Include all standard fields: email (unique, lowercase, trimmed), password, name, role, emailVerified, stripeCustomerId (sparse index), subscriptionStatus, emailBounced, emailSuppressed, plus timestamps.

### 21.3 — Owner Middleware

Stack it after `requireAuth` on every admin route. The owner must be fully authenticated through the normal login flow — Turnstile, rate limiting, session, everything. The role only determines *what* an authenticated user can do.

### 21.4 — Promoting the First Owner

You need a way to make the first user an owner. Three options, all valid:

**Option A — One-time CLI script (recommended):**

Run it once after deploying: `node scripts/promote-owner.js you@yourdomain.com`

**Option B — Atlas UI:** Open your Atlas cluster, find the user document, set `role: "owner"`. Quick and simple.

**Option C — Seed on startup (for the template repo):**

This uses an optional `SEED_OWNER_EMAIL` env var to promote a user on the first deploy. Remove the env var after the seed runs. The database is the source of truth from that point forward.

### 21.5 — Admin Routes

Create a dedicated admin router mounted at `/api/admin/*`. Apply `requireAuth` and `requireOwner` middleware to the entire router (not per-route). Log every admin action via pino with `adminAction: true`. Endpoints: list all users, view user details, get app-wide stats (total users, active subscriptions), and transfer ownership (demote current owner, promote new owner, destroy former owner's session).

### 21.6 — Expose Role to the Frontend

In the `GET /api/user/me` endpoint, include an `isOwner` flag in the response: `isOwner: user.role === 'owner'`. The frontend uses this flag to show or hide admin UI. Never expose the raw role string or other users' roles to non-admin users.

### 21.7 — Frontend Admin Gate

Create an `OwnerRoute` component that wraps admin routes. It checks `user.isOwner` from the auth query. If the user is not the owner, redirect to `/dashboard`. If loading, render nothing. If owner, render the `Outlet` for nested admin routes.

### 21.8 — What the Owner Can Do

```
Owner-only capabilities:
├── View all registered users and their subscription status
├── View app-wide stats (total users, active subscriptions)
├── View a specific user's account details (for support)
├── Transfer ownership to another registered user
├── Access admin dashboard at /admin
└── Any app-specific admin features you add per project

What the owner CANNOT bypass:
├── Normal login flow (Turnstile, rate limiting, lockout)
├── Session-based auth (no special session, no backdoor)
├── Password requirements (same rules as every user)
└── CSRF protection (same for all requests)
```

### 21.9 — Security Rules

1. **Never bypass auth.** The owner logs in like everyone else. The `role` field only controls authorization, never authentication.
2. **Log everything.** Every admin action goes through pino with `adminAction: true` so you can filter and audit.
3. **Admin routes are a separate router.** Mounted at `/api/admin/*`, isolated from user routes. The `requireOwner` middleware runs on every request in the router, not per-route (so you can't accidentally forget it).
4. **The database is the single source of truth.** The role lives in the User document. No env vars, no config files, no second source.
5. **Ownership transfer is logged and destroys the former owner's session.** This prevents the former owner from continuing to make admin requests on a stale session.

---

## Launch Checklist

This is the final gate before every app goes live.

### Testing (Hard Gate)

- [ ] All backend tests passing (Jest + Supertest)
- [ ] All frontend tests passing (Vitest + React Testing Library)
- [ ] Input validation tests cover every route: missing fields, boundary values, injection attempts
- [ ] Stripe webhook idempotency tested (duplicate event produces no duplicate side effects)
- [ ] npm audit shows no high or critical vulnerabilities (both client and server)
- [ ] CI pipeline green — GitHub Actions enforces all of the above

### Security

- [ ] All environment variables set in production (no `.env` file deployed)
- [ ] `NODE_ENV=production`
- [ ] Session cookie: `httpOnly`, `secure`, `sameSite: lax`
- [ ] Helmet.js enabled with all default headers
- [ ] CORS configured
- [ ] CSRF protection active on all state-changing routes
- [ ] Rate limiting active on all endpoints (auth + general API)
- [ ] Turnstile active on registration, login, and password reset forms
- [ ] Input validation (Zod) on every route that accepts user input
- [ ] Every Zod schema uses explicit types (`z.string()`, never `z.any()`) — NoSQL injection prevention
- [ ] Mongoose `sanitizeFilter: true` enabled as defense in depth
- [ ] CORS origin is strictly set to your frontend URL (never `*` or `true`)
- [ ] Centralized error handler registered — no stack traces leak to client
- [ ] MongoDB Atlas IP whitelist configured for Render IPs
- [ ] All secrets are strong, random, and unique per app

### Auth

- [ ] Registration → verification email → login flow works end-to-end
- [ ] Login lockout triggers at correct thresholds (5/10 attempts)
- [ ] Password reset flow works end-to-end
- [ ] Password change invalidates other sessions
- [ ] Unverified emails cannot log in
- [ ] Account deletion removes all user data and cancels Stripe subscription
- [ ] Data export downloads complete JSON of user data
- [ ] Owner user promoted via CLI script or Atlas (role: 'owner' in database)
- [ ] Owner can access `/admin` routes after normal login
- [ ] Non-owner users get 403 on `/api/admin/*` endpoints
- [ ] `isOwner` flag returned from `/api/user/me` only for users with role 'owner'
- [ ] Admin actions logged with `adminAction: true` in pino
- [ ] Ownership transfer endpoint works and destroys former owner's session

### Payments

- [ ] Stripe keys swapped from test to live
- [ ] Webhook endpoint configured and verified in Stripe dashboard
- [ ] Webhook handler is idempotent (ProcessedEvent collection active)
- [ ] Checkout flow completes successfully (test with Stripe test cards)
- [ ] Customer portal accessible from settings
- [ ] Subscription status correctly gates premium features
- [ ] Payment receipt emails send on successful charge
- [ ] Payment failure emails send on failed charge
- [ ] Cancellation flow works and revokes access at period end

### Email

- [ ] Resend domain verified with DNS records
- [ ] All transactional emails rendering correctly
- [ ] From address matches verified domain
- [ ] Unsubscribe link present in marketing/onboarding emails
- [ ] Resend bounce/complaint webhook configured
- [ ] Bounced and suppressed email flags checked before sending

### Infrastructure

- [ ] `.node-version` file in repo root with pinned version (e.g., `20.18.0`)
- [ ] `engines.node` in root `package.json` matches `.node-version`
- [ ] `package-lock.json` committed (both root and `client/`)
- [ ] `"start"` script defined in root `package.json`
- [ ] Express binds to `process.env.PORT` on host `0.0.0.0`
- [ ] All environment variables set in Render dashboard
- [ ] Render service deployed and healthy
- [ ] Custom domain configured with SSL
- [ ] Health check endpoint responding at `/api/health`
- [ ] `render.yaml` blueprint committed
- [ ] `README.md` updated with app-specific details
- [ ] `.gitignore` in place — no `.env`, `node_modules`, or `dist/` committed
- [ ] Graceful shutdown handles SIGTERM (in-flight requests complete before exit)
- [ ] Structured logging active (pino in production, pino-pretty in dev)
- [ ] Request logging middleware active (excluding health checks)
- [ ] 404 page renders for unknown routes
- [ ] `robots.txt` accessible and blocks authenticated routes
- [ ] `sitemap.xml` accessible and lists only public pages
- [ ] `sitemap.xml` domain matches production URL
- [ ] Meta title and description set for landing and pricing pages
- [ ] Open Graph tags set (og:title, og:description, og:image, og:url)
- [ ] Twitter Card tags set
- [ ] `og-image.png` (1200×630px) created and accessible
- [ ] Lighthouse accessibility score 90+ on landing page and login page
- [ ] All form inputs have associated labels
- [ ] Keyboard navigation works on all interactive elements

### Database

- [ ] All required indexes created (user email, stripeCustomerId, rate limit lookups)
- [ ] All TTL indexes active (sessions, email tokens, login attempts, processed events)
- [ ] Atlas tier upgraded to M10+ for production (continuous backups enabled)
- [ ] Backup strategy verified — can restore from point-in-time or snapshot

### Monitoring

- [ ] Sentry capturing errors in both frontend and backend
- [ ] PostHog tracking key conversion events (signup, verify, subscribe, cancel)
- [ ] Render health check alerts enabled

### Dependency Security

- [ ] Dependabot enabled for both client and server directories
- [ ] npm audit passing with no high/critical vulnerabilities
- [ ] CI pipeline includes audit step

### Legal & Privacy (Stripe hard gate — cannot accept payments without these)

- [ ] Terms of Service page live (required by Stripe for live mode)
- [ ] Privacy Policy page live (required by Stripe, GDPR, CCPA — includes data export and deletion rights)
- [ ] Cookie consent banner (if required for your audience)
- [ ] Links in footer and registration form
- [ ] Data export endpoint functional
- [ ] Account deletion endpoint functional

### Final Smoke Test

- [ ] Register a new account with real email
- [ ] Verify email
- [ ] Log in
- [ ] Subscribe via Stripe Checkout
- [ ] Access premium features
- [ ] Manage subscription via Customer Portal
- [ ] Cancel subscription
- [ ] Verify access revoked at period end
- [ ] Reset password
- [ ] Log in with new password
- [ ] Export account data
- [ ] Promote your account to owner — verify admin dashboard accessible
- [ ] Verify non-owner account cannot access /admin
- [ ] Transfer ownership via admin endpoint — verify new owner has access, old owner does not
- [ ] Visit a nonexistent URL — verify 404 page renders
- [ ] Visit /robots.txt — verify it's accessible
- [ ] Visit /sitemap.xml — verify it lists public pages with correct domain
- [ ] Test landing page URL on https://opengraph.xyz — verify preview image and text render
- [ ] Run Lighthouse accessibility audit on landing page — score 90+
- [ ] Delete account — verify all data removed

---

## Quick Reference: New App Workflow

```
1.  GitHub      → "Use this template" to create new repo
2.  Local       → Clone, npm install, copy .env.example → .env
3.  Atlas       → Create new project + cluster (M10+ for production)
4.  Stripe      → Create new product + price objects
5.  Resend      → Add sending domain + verify DNS + configure bounce webhook
6.  Turnstile   → Create new site in Cloudflare dashboard
7.  Build       → Strip placeholder pages, build your app-specific features
8.  README      → Update README.md with app name, description, and specifics
9.  Test        → Write tests for new features, run full test suite (hard gate)
10. Render      → Deploy via Blueprint, set env vars, configure domain
11. Owner       → Register your account, promote to owner via CLI script or Atlas
12. Stripe      → Point webhook to production API URL, switch to live keys
13. Sentry      → Create new project, add DSN to env vars
14. PostHog     → Create new project, add API key to env vars
15. Dependabot  → Verify enabled on new repo
16. Legal       → Generate/customize ToS + Privacy Policy
17. Launch      → Run the launch checklist above
```

---

## Post-Launch

These are not blocking launch but should be tackled once the app is live and you have real users.

### API Documentation

Once your app is stable, document your API endpoints for your own reference and for any future collaborators.

```
Options (pick one):
├── Swagger / OpenAPI spec — auto-generate interactive docs from route definitions
├── Simple Markdown doc — document each endpoint manually in a /docs directory
└── Postman collection — export your API calls as a shareable collection
```

For a solo dev, a Markdown file in your repo (`docs/API.md`) is the fastest option. Document each endpoint with method, path, auth requirement, request body shape, and response shape. Graduate to Swagger if you ever open your API to third parties.

### Load Testing

Before scaling or running a promotion that might spike traffic, test your app under load.

```
Tools:
├── k6 (recommended) — scriptable, runs from CLI, free, excellent for Node APIs
├── Artillery — YAML-based, good for quick tests
└── Apache Bench (ab) — simplest option for basic throughput tests

What to test:
├── Landing page (unauthenticated) — baseline response time
├── Login endpoint — verify rate limiting holds under pressure
├── Authenticated API calls — verify session handling under concurrent users
└── Stripe webhook endpoint — verify it handles burst webhook deliveries

Targets for a single Render starter instance:
├── < 200ms average response time at 50 concurrent users
├── < 1% error rate at 100 concurrent users
└── Rate limiter correctly rejects excess requests without crashing
```

Run load tests against a staging environment, never production. If you hit limits, upgrade your Render plan before launch.

### CDN for Static Assets

Your single-service Render deployment serves static assets (JS, CSS, images) from Express. This works fine at low to moderate traffic. If you hit performance issues or want faster global delivery, add a CDN in front.

```
Options:
├── Cloudflare (recommended) — you already use Turnstile, so you have an account.
│   Point your domain's DNS through Cloudflare, enable caching. Zero code changes.
├── Split deployment — move React to a separate Render static site (has built-in CDN)
└── AWS CloudFront — overkill for a solo dev unless you're on AWS already
```

This is an optimization, not a requirement. Most SaaS apps with fewer than 10K users won't notice the difference.

---

## Dependency Reference

### Vendor Accounts (9 companies — requires signup)

| Vendor | Purpose | URL |
| ------ | ------- | --- |
| MongoDB | Database hosting (Atlas) | https://www.mongodb.com/atlas |
| Stripe | Payments | https://stripe.com |
| Resend | Transactional email | https://resend.com |
| Cloudflare | Bot protection (Turnstile) + CDN (post-launch) | https://www.cloudflare.com |
| Render | Hosting | https://render.com |
| Sentry | Error tracking | https://sentry.io |
| PostHog | Analytics | https://posthog.com |
| GitHub | Version control, CI/CD, Dependabot | https://github.com |
| Anthropic | Claude Code (AI assistant) | https://www.anthropic.com |

### Open Source Dependencies (no account required — install and use)

| Dependency | Purpose | URL |
| ---------- | ------- | --- |
| Node.js | JavaScript runtime | https://nodejs.org |
| Express.js | Backend API framework | https://expressjs.com |
| React | Frontend UI framework | https://react.dev |
| Vite | Frontend build tool | https://vite.dev |
| Mongoose | MongoDB ODM | https://mongoosejs.com |
| React Router | Client-side routing | https://reactrouter.com |
| TanStack Query | Server state management | https://tanstack.com/query |
| Zustand | Client state management | https://zustand.docs.pmnd.rs |
| Tailwind CSS (when needed) | Utility-first CSS framework | https://tailwindcss.com |
| react-helmet-async | SEO meta tag management | https://github.com/staylor/react-helmet-async |
| express-session | Session middleware | https://github.com/expressjs/session |
| connect-mongo | MongoDB session store | https://github.com/jdesrosi/connect-mongo |
| Helmet.js | Security headers | https://helmetjs.github.io |
| cors | CORS middleware | https://github.com/expressjs/cors |
| Zod | Input validation | https://zod.dev |
| dotenv | Environment variables | https://github.com/motdotla/dotenv |
| pino | Structured logging | https://github.com/pinojs/pino |
| pino-pretty | Dev-friendly log output | https://github.com/pinojs/pino-pretty |
| pino-http | HTTP request logging | https://github.com/pinojs/pino-http |
| marked | Markdown processing | https://marked.js.org |
| isomorphic-dompurify | HTML sanitization | https://github.com/kkomelin/isomorphic-dompurify |
| multer | File upload handling | https://github.com/expressjs/multer |
| Jest | Backend testing | https://jestjs.io |
| Supertest | HTTP assertion testing | https://github.com/ladakh/supertest |
| Vitest | Frontend testing | https://vitest.dev |
| React Testing Library | Component testing | https://testing-library.com/docs/react-testing-library/intro |
| k6 (post-launch) | Load testing | https://k6.io |
| ffmpeg | Audio/video encoding | https://ffmpeg.org |

### Browser-Native (zero dependency — built into the platform)

| API | Purpose |
| --- | ------- |
| Canvas 2D API | Drawing, image manipulation |
| Node.js crypto (scrypt) | Password hashing |

---

### Detailed Pricing Breakdown

### Core Stack

| Dependency | URL | Pricing |
| ---------- | --- | ------- |
| Node.js | https://nodejs.org | Free, open source |
| Express.js | https://expressjs.com | Free, open source |
| React | https://react.dev | Free, open source |
| Vite | https://vite.dev | Free, open source |
| MongoDB Atlas | https://www.mongodb.com/atlas | Free tier (M0): 512MB storage, shared cluster. Paid starts at M10 (~$57/mo) for dedicated cluster + continuous backups. Upgrade before launch. |
| Mongoose | https://mongoosejs.com | Free, open source |

### Frontend Libraries

| Dependency | URL | Pricing |
| ---------- | --- | ------- |
| React Router | https://reactrouter.com | Free, open source |
| TanStack Query (React Query) | https://tanstack.com/query | Free, open source |
| Zustand | https://zustand.docs.pmnd.rs | Free, open source |
| Tailwind CSS (when needed) | https://tailwindcss.com | Free, open source |
| react-helmet-async | https://github.com/staylor/react-helmet-async | Free, open source |
| PostHog JS | https://github.com/PostHog/posthog-js | Free, open source (client library — usage billed through PostHog cloud) |

### Auth & Security

| Dependency | URL | Pricing |
| ---------- | --- | ------- |
| express-session | https://github.com/expressjs/session | Free, open source |
| connect-mongo | https://github.com/jdesrosi/connect-mongo | Free, open source |
| Helmet.js | https://helmetjs.github.io | Free, open source |
| cors | https://github.com/expressjs/cors | Free, open source |
| Zod | https://zod.dev | Free, open source |
| Cloudflare Turnstile | https://developers.cloudflare.com/turnstile | Free, unlimited. No paid tier required for Turnstile specifically. |

### Payments & Email

| Dependency | URL | Pricing |
| ---------- | --- | ------- |
| Stripe Node SDK | https://github.com/stripe/stripe-node | Free (library). Stripe charges 2.9% + $0.30 per transaction. No monthly fee. |
| Stripe Checkout Docs | https://docs.stripe.com/payments/checkout | Included with Stripe account |
| Stripe Customer Portal | https://docs.stripe.com/billing/subscriptions/integrating-customer-portal | Included with Stripe account |
| Stripe Webhooks | https://docs.stripe.com/webhooks | Included with Stripe account |
| Resend | https://resend.com | Free tier: 3,000 emails/month, 100/day, 1 domain. Paid (Pro): $20/mo for 50,000 emails/month. |
| React Email (alternative) | https://react.email | Free, open source |

### Infrastructure & Monitoring

| Dependency | URL | Pricing |
| ---------- | --- | ------- |
| Render | https://render.com | Free tier: 750 hours/month (sleeps after 15 min inactivity). Starter: $7/mo (no sleep, always on). Upgrade before launch. |
| Sentry (Node) | https://docs.sentry.io/platforms/javascript/guides/express | Free tier: 5K errors/month, 10K transactions/month. Paid (Team): $26/mo for 50K errors. |
| Sentry (React) | https://docs.sentry.io/platforms/javascript/guides/react | Same account as above |
| PostHog | https://posthog.com | Free tier: 1M analytics events/month, 5K session recordings, 1M feature flag requests, 1 project. No credit card required. Paid: usage-based after free limits. |
| pino | https://github.com/pinojs/pino | Free, open source |
| pino-pretty | https://github.com/pinojs/pino-pretty | Free, open source |
| pino-http | https://github.com/pinojs/pino-http | Free, open source |
| dotenv | https://github.com/motdotla/dotenv | Free, open source |

### Testing

| Dependency | URL | Pricing |
| ---------- | --- | ------- |
| Jest | https://jestjs.io | Free, open source |
| Supertest | https://github.com/ladakh/supertest | Free, open source |
| Vitest | https://vitest.dev | Free, open source |
| React Testing Library | https://testing-library.com/docs/react-testing-library/intro | Free, open source |

### CI/CD & Dependency Management

| Dependency | URL | Pricing |
| ---------- | --- | ------- |
| GitHub Actions | https://docs.github.com/en/actions | Free tier: 2,000 minutes/month on public repos (unlimited), 2,000 min/month on private repos. More than enough for a solo dev. |
| GitHub Template Repos | https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-template-repository | Included with GitHub account |
| Dependabot | https://docs.github.com/en/code-security/dependabot | Free, included with GitHub |

### Edge Case Tools

| Dependency | URL | Pricing |
| ---------- | --- | ------- |
| Canvas 2D API (MDN) | https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API | Free, browser-native |
| MongoDB GridFS | https://www.mongodb.com/docs/manual/core/gridfs | Included with MongoDB Atlas (uses your cluster storage) |
| multer | https://github.com/expressjs/multer | Free, open source |
| marked | https://marked.js.org | Free, open source |
| isomorphic-dompurify | https://github.com/kkomelin/isomorphic-dompurify | Free, open source |
| ffmpeg | https://ffmpeg.org | Free, open source. Pre-installed on Render. |
| Node.js crypto (scrypt) | https://nodejs.org/api/crypto.html#cryptoscryptpassword-salt-keylen-options-callback | Free, built into Node.js |

### Development Tools

| Dependency | URL | Pricing |
| ---------- | --- | ------- |
| Visual Studio Code | https://code.visualstudio.com | Free |
| Claude Code | https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview | Requires Anthropic API key or Max subscription. Usage-based API pricing. |
| GitHub | https://github.com | Free tier: unlimited public/private repos, 2,000 Actions minutes/month. Paid (Pro): $4/mo for advanced features. Free tier is sufficient. |

### Post-Launch

| Dependency | URL | Pricing |
| ---------- | --- | ------- |
| k6 (load testing) | https://k6.io | Free, open source (CLI). Cloud: free tier for 50 tests/month. |
| Artillery (load testing) | https://artillery.io | Free, open source (CLI). Cloud: paid plans for distributed testing. |
| Swagger / OpenAPI | https://swagger.io | Free (open source spec + editor). Paid tools available but not needed. |
| Cloudflare (CDN) | https://www.cloudflare.com | Free tier: DNS, CDN, DDoS protection, SSL. Covers everything a solo dev needs. |

### Legal & SEO

| Dependency | URL | Pricing |
| ---------- | --- | ------- |
| Termly (legal page generator) | https://termly.io | Free tier: 1 policy. Paid: $10/mo for full compliance suite. |
| Iubenda (legal page generator) | https://www.iubenda.com | Free tier: basic privacy policy. Paid: ~$29/yr for full suite. |
| Open Graph Debugger | https://opengraph.xyz | Free |
| Facebook Sharing Debugger | https://developers.facebook.com/tools/debug | Free |
| Lighthouse (Chrome DevTools) | https://developer.chrome.com/docs/lighthouse | Free, built into Chrome |

### Monthly Cost Summary (Free Tier)

| Service | Free Tier Limit | When You'll Need to Pay |
| ------- | --------------- | ----------------------- |
| MongoDB Atlas | 512MB storage | Before launch — upgrade to M10 (~$57/mo) for backups |
| Render | 750 hrs/mo (sleeps) | Before launch — upgrade to Starter ($7/mo) to prevent cold starts |
| Stripe | No monthly fee | Per-transaction only (2.9% + $0.30) |
| Resend | 3,000 emails/mo | ~100+ active users sending transactional emails regularly |
| PostHog | 1M events/mo | ~5K-10K monthly active users depending on event volume |
| Sentry | 5K errors/mo | Unlikely to hit unless you have a major bug in production |
| GitHub Actions | 2,000 min/mo | Unlikely to hit as a solo dev |
| Cloudflare | Unlimited (basic) | Unlikely to need paid tier |
| Claude Code | Usage-based | From first use (requires API key or Max subscription) |

**Minimum monthly cost to launch:** ~$64/mo (Atlas M10 + Render Starter) plus Stripe transaction fees. Everything else stays free until you grow.
