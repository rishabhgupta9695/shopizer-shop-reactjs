# Shopizer Shop — Developer Onboarding Guide

Welcome to the team! This doc will get you up to speed on the project quickly.

---

## 1. What Is This Project?

This is the **customer-facing storefront** for [Shopizer](https://www.shopizer.com/) — an open-source e-commerce platform. This React app talks to a Shopizer Java backend via REST APIs and provides a full shopping experience: browsing, cart, checkout, orders, and account management.

---

## 2. Tech Stack

| Layer | Technology |
|---|---|
| UI Framework | React 16 |
| State Management | Redux + Redux Thunk |
| Routing | React Router DOM v5 |
| Styling | SCSS + Bootstrap 4 |
| HTTP Client | Axios |
| Payments | Stripe, Nuvei |
| i18n | redux-multilanguage (EN, FR) |
| Containerization | Docker + Nginx |
| CI/CD | CircleCI |

---

## 3. System Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Browser                          │
│                                                     │
│   ┌─────────────────────────────────────────────┐   │
│   │         React App (this repo)               │   │
│   │                                             │   │
│   │  ┌──────────┐   ┌──────────┐  ┌─────────┐  │   │
│   │  │  Pages   │──▶│Components│  │ Redux   │  │   │
│   │  │          │   │/Wrappers │  │  Store  │  │   │
│   │  └──────────┘   └──────────┘  └────┬────┘  │   │
│   │                                    │        │   │
│   │              ┌─────────────────────┘        │   │
│   │              ▼                              │   │
│   │        ┌──────────┐                         │   │
│   │        │WebService│  (Axios + interceptors) │   │
│   │        └────┬─────┘                         │   │
│   └─────────────┼───────────────────────────────┘   │
└─────────────────┼───────────────────────────────────┘
                  │ REST API (Bearer token)
                  ▼
        ┌──────────────────┐
        │  Shopizer Backend │
        │  (Java / Spring)  │
        │  localhost:8080   │
        └──────────────────┘
```

---

## 4. Project Structure

```
shopizer-shop-reactjs/
├── public/
│   ├── env-config.js        ← Runtime config (API URL, keys, theme)
│   └── assets/              ← Static images
├── src/
│   ├── index.js             ← App entry point, Redux store setup
│   ├── App.js               ← Route definitions
│   ├── pages/               ← Top-level page components
│   │   ├── home/            ← Homepage
│   │   ├── category/        ← Product listing by category
│   │   ├── product-details/ ← Single product view
│   │   ├── search-product/  ← Search results
│   │   ├── content/         ← CMS content pages
│   │   └── other/           ← Cart, Checkout, Auth, Account, Orders
│   ├── components/          ← Reusable UI components
│   ├── wrappers/            ← Section-level layout wrappers
│   ├── redux/
│   │   ├── actions/         ← Async action creators (API calls)
│   │   └── reducers/        ← State slices
│   ├── util/
│   │   ├── webService.js    ← Axios wrapper + interceptors
│   │   └── constant.js      ← API endpoint constants
│   ├── helpers/             ← Pure utility functions
│   ├── translations/        ← EN / FR JSON language files
│   └── assets/scss/         ← Global styles
├── conf/                    ← Nginx config (used in Docker)
├── Dockerfile
├── .env                     ← Local env defaults
└── .circleci/config.yml     ← CI pipeline
```

---

## 5. Application Routes

```
/                          → Home
/category/:id              → Product category listing
/product/:id               → Product detail
/search/:id                → Search results
/content/:id               → CMS content page
/cart                      → Shopping cart
/checkout                  → Checkout flow
/order-confirm             → Post-order confirmation
/order-details/:id         → Specific order detail
/recent-order              → Recent orders list
/login                     → Login
/register                  → Register
/forgot-password           → Forgot password
/customer/:code/reset/:id  → Password reset
/my-account                → Account dashboard
/contact                   → Contact page
```

---

## 6. Redux State Shape

```
store
├── multilanguage       ← Current language (en/fr)
├── productData         ← Product listings
├── merchantData        ← Store/merchant info
├── cartData            ← Cart items + cart ID
├── loading             ← Global loader flag
├── userData            ← Logged-in user info
└── content             ← CMS content
```

Redux state is **persisted to localStorage** via `redux-localstorage-simple`.

---

## 7. Data Flow — Add to Cart Example

```
User clicks "Add to Cart"
        │
        ▼
  cartActions.addToCart()
        │
        ├─ dispatch(setLoader(true))
        │
        ├─ POST /api/v1/cart?store=DEFAULT   (new cart)
        │  PUT  /api/v1/cart/:id?store=DEFAULT  (existing)
        │
        ├─ Response → setShopizerCartID()
        │              └─ saves cart ID to cookie + localStorage
        │
        ├─ dispatch(getCart())  ← refreshes cart state
        │
        └─ dispatch(setLoader(false))
```

---

## 8. Authentication Flow

```
User submits login form
        │
        ▼
  POST /api/v1/auth/login
        │
        ▼
  JWT token received
        │
        ▼
  Stored in localStorage ("token")
        │
        ▼
  axios.interceptors.request  ← attaches "Authorization: Bearer <token>"
  on every subsequent request
```

---

## 9. Local Setup

### Prerequisites
- Node.js v16.13.0
- A running Shopizer backend at `http://localhost:8080`

### Steps

```bash
# 1. Clone and install
npm install
# if that fails:
npm install --legacy-peer-deps

# 2. Configure the backend URL
# Edit public/env-config.js:
window._env_ = {
  APP_BASE_URL: "http://localhost:8080",  # ← point to your backend
  APP_MERCHANT: "DEFAULT",
  APP_PAYMENT_TYPE: "STRIPE",
  APP_STRIPE_KEY: "your_stripe_key",
  APP_THEME_COLOR: "#D1D1D1",
  ...
}

# 3. Start the dev server
npm run dev
# App runs at http://localhost:3000
```

---

## 10. Environment Configuration

All runtime config lives in `public/env-config.js` (loaded before React boots).

| Variable | Purpose |
|---|---|
| `APP_BASE_URL` | Shopizer backend URL |
| `APP_API_VERSION` | API version prefix (`/api/v1/`) |
| `APP_MERCHANT` | Merchant code (default: `DEFAULT`) |
| `APP_PRODUCT_GRID_LIMIT` | Products per page |
| `APP_PAYMENT_TYPE` | `STRIPE` or `NUVEI` |
| `APP_STRIPE_KEY` | Stripe publishable key |
| `APP_THEME_COLOR` | Primary theme color (CSS variable) |
| `APP_MAP_API_KEY` | Google Maps API key |

> In Docker, these are injected at container startup via `env.sh` which rewrites `env-config.js` from environment variables.

---

## 11. Docker Deployment

```bash
# Build image
docker build . -t shopizerecomm/shopizer-shop:latest

# Run container
docker run \
  -e "APP_MERCHANT=DEFAULT" \
  -e "APP_BASE_URL=http://your-backend:8080" \
  -e "APP_THEME_COLOR=#D1D1D1" \
  -it --rm -p 80:80 \
  shopizerecomm/shopizer-shop:latest

# App available at http://localhost
```

The Docker build is multi-stage:
1. **Builder** — Node 13 Alpine, runs `npm run build`
2. **Production** — Nginx Alpine, serves the static build, runs `env.sh` on startup to inject env vars

---

## 12. Key Files to Know First

| File | Why it matters |
|---|---|
| `src/App.js` | All routes defined here |
| `src/index.js` | Redux store creation |
| `src/util/webService.js` | All API calls go through here |
| `src/util/constant.js` | All API endpoint strings |
| `public/env-config.js` | Runtime configuration |
| `src/redux/reducers/rootReducer.js` | Full state shape |
| `src/pages/other/Checkout.js` | Most complex page — good to read early |

---

## 13. Adding a New Page (Checklist)

- [ ] Create component in `src/pages/`
- [ ] Add a lazy import in `src/App.js`
- [ ] Add a `<Route>` in the `<Switch>` in `src/App.js`
- [ ] Add any new API endpoints to `src/util/constant.js`
- [ ] Add translation keys to both `src/translations/english.json` and `french.json`

---

## 14. Common Gotchas

- **Cart ID** is stored in both a cookie and localStorage. If cart behaves oddly, clear both.
- **`window._env_`** is used everywhere for config — never use `process.env` directly in components.
- The `npm run dev` and `npm start` scripts both start the dev server. `npm run intergartion` is for Docker-style env injection locally.
- React version is 16 — hooks are available but class components may exist in older parts of the code.
- Payment provider is toggled via `APP_PAYMENT_TYPE` — Stripe is default.
