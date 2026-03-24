# Shopizer Admin — Project Documentation

## Overview

**Shopizer Admin** is an Angular-based single-page application (SPA) that serves as the back-office management panel for the [Shopizer](https://www.shopizer.com) e-commerce platform. It communicates with the Shopizer REST API backend and provides a full-featured admin UI for managing stores, products, orders, customers, content, shipping, payments, and taxes.

- **Framework:** Angular 11
- **UI Library:** Nebular (by Akveo)
- **Default URL:** `http://localhost:4200`
- **Backend API:** `http://localhost:8080/api`
- **Default Credentials:** `admin@shopizer.com` / `password`

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Browser (SPA)                           │
│                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │  Auth Module │    │ Pages Module │    │ Theme Module │  │
│  │  (login /    │    │ (all feature │    │ (layout,     │  │
│  │   reset pwd) │    │  modules)    │    │  styles,     │  │
│  └──────┬───────┘    └──────┬───────┘    │  components) │  │
│         │                   │            └──────────────┘  │
│         └─────────┬─────────┘                              │
│                   │                                         │
│          ┌────────▼────────┐                               │
│          │   AppModule     │                               │
│          │  (root module)  │                               │
│          └────────┬────────┘                               │
│                   │                                         │
│          ┌────────▼────────┐                               │
│          │  HTTP Interceptors                              │
│          │  - AuthInterceptor (JWT token injection)        │
│          │  - GlobalHttpInterceptorService (error handler) │
│          └────────┬────────┘                               │
└───────────────────┼─────────────────────────────────────────┘
                    │ HTTP (REST)
                    ▼
┌─────────────────────────────────────────────────────────────┐
│              Shopizer Backend API                           │
│              http://localhost:8080/api                      │
└─────────────────────────────────────────────────────────────┘
```

---

## Module Structure

```
src/app/
├── app.module.ts              ← Root module
├── app-routing.module.ts      ← Top-level routes
├── @core/                     ← Core services, data models, utilities
│   ├── core.module.ts
│   ├── data/                  ← Data service interfaces
│   ├── mock/                  ← Mock data providers
│   └── utils/                 ← Utility services
├── @theme/                    ← Shared UI: layouts, components, pipes, styles
│   ├── theme.module.ts
│   ├── components/            ← Shared components (header, footer, TinyMCE, etc.)
│   ├── layouts/               ← Page layout wrappers
│   ├── pipes/                 ← Custom Angular pipes
│   └── styles/                ← Global SCSS styles
└── pages/                     ← Feature modules (lazy-loaded)
    ├── auth/                  ← Login, password reset
    ├── home/                  ← Dashboard
    ├── user-management/       ← Admin users
    ├── store-management/      ← Store configuration
    ├── catalogue/             ← Products, categories, brands, options
    ├── content/               ← CMS pages, boxes, images
    ├── shipping/              ← Shipping config, methods, origin, packaging
    ├── payment/               ← Payment methods
    ├── tax-management/        ← Tax classes and rates
    ├── customers/             ← Customer list and options
    ├── orders/                ← Order management
    └── shared/                ← Guards, interceptors, validators, shared components
```

---

## Routing Map

```
/                        → redirects to /pages
/auth                    → Login page
/user/:id/reset/:id      → Password reset
/pages                   → Main app shell (requires AuthGuard)
  /home                  → Dashboard
  /user-management
    /profile             → My Profile
    /create-user         → Create User (Admin only)
    /users               → User List (Admin only)
  /store-management
    /store               → Edit current store
    /stores-list         → All stores (Admin only)
    /create-store        → Create store (Superadmin / AdminRetail)
  /catalogue             → (requires SuperadminStoreRetailCatalogueGuard)
    /categories/...      → Category list, create, hierarchy
    /products/...        → Product list, product ordering
    /options/...         → Options, option values, option sets, variations
    /brands/...          → Brand list, create brand
    /products-groups/... → Product groups
    /types/...           → Product types
  /content
    /pages/list          → CMS pages
    /boxes/list          → Content boxes
    /images/list         → Content images
  /shipping
    /config              → Expedition config
    /methods             → Shipping methods
    /origin              → Shipping origin
    /packaging           → Packaging
  /payment/methods       → Payment methods
  /tax-management
    /classes-list        → Tax classes
    /rate-list           → Tax rates
  /customer/list         → Customer list
  /orders                → Order list
```

---

## Role-Based Access Control

Access to menu items and routes is controlled by roles stored in `localStorage` under the `roles` key.

| Role               | Access                                                                 |
|--------------------|------------------------------------------------------------------------|
| `isSuperadmin`     | Full access to all modules including store creation and global categories |
| `isAdmin`          | Store management, user management, catalogue, orders                   |
| `isAdminRetail`    | Store management, catalogue, orders                                    |
| `isAdminCatalogue` | Catalogue module only                                                  |
| `isAdminStore`     | Store management only                                                  |
| `isAdminOrder`     | Order management only                                                  |
| `isAdminContent`   | Content management only                                                |
| `isCustomer`       | Customer-facing access                                                 |

---

## Operation Modes

Configured in `src/environments/environment.ts` via the `mode` field:

| Mode          | Behaviour                                                    |
|---------------|--------------------------------------------------------------|
| `STANDARD`    | Default. Categories and options are store-specific.          |
| `MARKETPLACE` | Categories and options are global; only Superadmin manages them. |
| `BTB`         | Business-to-business mode.                                   |

---

## Key Dependencies

| Package                  | Purpose                              |
|--------------------------|--------------------------------------|
| `@angular/core` v11      | Core framework                       |
| `@nebular/theme` v6      | UI component library                 |
| `@nebular/auth` v5       | Authentication (JWT)                 |
| `@ngx-translate/core`    | i18n / translations                  |
| `ng2-smart-table`        | Data tables                          |
| `ngx-toastr`             | Toast notifications                  |
| `ngx-summernote`         | Rich text editor                     |
| `primeng`                | Additional UI components             |
| `rxjs` v6                | Reactive programming                 |
| `bootstrap` v4           | CSS grid and utilities               |

---

## Internationalization (i18n)

Translation files live in `src/assets/i18n/`:

| File      | Language |
|-----------|----------|
| `en.json` | English  |
| `fr.json` | French   |
| `es.json` | Spanish  |
| `ru.json` | Russian  |

Default language is `en`, configurable in `environment.ts` under `client.language.default`.

---

## Environment Configuration

### Development (`src/environments/environment.ts`)
```ts
{
  production: false,
  mode: 'STANDARD',           // STANDARD | MARKETPLACE | BTB
  apiUrl: 'http://localhost:8080/api',
  shippingApi: 'http://localhost:9090/shipping/api/v1',
  googleApiKey: '',
  client: {
    language: { default: 'en', array: ['fr', 'en'] }
  }
}
```

### Production
Built with `ng build --prod`. Environment values are swapped from `environment.prod.ts`.  
At runtime in Docker, `APP_BASE_URL` env var is injected into `assets/env.js` via `envsubst`.

---

## Getting Started

### Prerequisites
- Node.js v12.22.7
- Angular CLI 13.3.x: `npm install -g @angular/cli@13.3.x`
- Shopizer backend running at `http://localhost:8080`

### Run Locally
```bash
npm install --legacy-peer-deps
ng serve -o
# Opens http://localhost:4200
```

### Build for Production
```bash
ng build
# Output goes to /dist
```

---

## Docker Deployment

```
┌──────────────────────────────────────────────────────┐
│                  Docker Container                    │
│                                                      │
│   nginx:alpine                                       │
│   ├── /usr/share/nginx/html  ← Angular /dist build  │
│   ├── /etc/nginx/conf.d/     ← Custom nginx config  │
│   └── assets/env.js          ← Runtime API URL       │
│                                                      │
│   Port: 80 (mapped to host port 4200)               │
└──────────────────────────────────────────────────────┘
```

```bash
docker run \
  -e "APP_BASE_URL=http://localhost:9090/api" \
  -it --rm -p 4200:80 \
  shopizerecomm/shopizer-admin
```

The `APP_BASE_URL` environment variable overrides the backend API URL at container startup via `envsubst` on `env.template.js`.

---

## CI/CD

CircleCI configuration is at `.circleci/config.yml`. It handles automated build and deployment pipelines.

---

## Useful Scripts

| Script                  | Description                              |
|-------------------------|------------------------------------------|
| `npm start`             | Start dev server                         |
| `npm run build`         | Production build (8GB heap)              |
| `npm run lint`          | Run ESLint                               |
| `npm run lint:styles`   | Run Stylelint on SCSS files              |
| `npm run docs`          | Generate Compodoc API documentation      |
| `npm run docs:serve`    | Serve Compodoc docs locally              |
| `npm run test`          | Run unit tests (Karma + Jasmine)         |
| `npm run e2e`           | Run end-to-end tests (Protractor)        |
| `npm run release:changelog` | Generate CHANGELOG.md              |
