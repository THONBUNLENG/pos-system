# POS System

A modern web-based **Point of Sale (POS) system** built with a **Laravel 12 API backend** and a **Next.js 16 frontend**. Designed for retail operations featuring product inventory, real-time carts, order processing, and native **KHQR Bakong** payment integration.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [API Endpoints](#api-endpoints)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Backend Setup](#backend-setup)
  - [Frontend Setup](#frontend-setup)
- [Configuration](#configuration)
- [Usage](#usage)
- [Testing](#testing)
- [License](#license)

---

## Overview

This system provides a complete POS solution split into two services:

| Service | Path | Description |
| :--- | :--- | :--- |
| **Backend** | `backend/` | Laravel 12 RESTful API with Sanctum auth, inventory, cart, orders, and KHQR payments |
| **Frontend** | `frontend/` | Next.js 16 responsive web client for staff and admin users |

> Refer to the individual sub-package READMEs for detailed setup:
> - [Backend README](backend/README.md)
> - [Frontend README](frontend/README.md)

---

## Features

- **Authentication** — Secure Sanctum-based bearer token login, registration, and logout
- **User Management** — Admin-only user listing and deletion
- **Product Inventory** — Full CRUD with categories and pricing
- **Cart Engine** — Backend-driven carts for cross-device persistence (authenticated)
- **Order Processing** — Automated total calculation, order summaries, and checkout
- **Dual Payment** — Cash checkout and KHQR Bakong QR code payments with verify/cancel flows
- **Admin & Staff UI** — Role-aware interface built with Radix UI components
- **Rate Limiting** — Per-route throttling for security and stability

---

## Tech Stack

### Backend (`backend/`)

| Layer | Technology |
| :--- | :--- |
| **Framework** | Laravel 12.x |
| **Language** | PHP 8.2+ |
| **Auth** | Laravel Sanctum |
| **Payments** | khqr-gateway/bakong-khqr-php |
| **Database** | SQLite (dev) / MySQL or PostgreSQL (prod) |
| **Dev Tools** | Composer, Artisan, PHPUnit, Laravel Pint, Laravel Pail |
| **Asset Build** | Vite |

### Frontend (`frontend/`)

| Layer | Technology |
| :--- | :--- |
| **Framework** | Next.js 16 (App Router) |
| **Language** | TypeScript |
| **UI** | React 19, Radix UI, Tailwind CSS v4 |
| **Forms** | React Hook Form + Zod |
| **Notifications** | Sonner |
| **Icons** | Lucide React |
| **Package Manager** | pnpm |

---

## Project Structure

```
pos-system/
├── backend/                  # Laravel API
│   ├── app/
│   │   ├── Http/Controllers/Api/   # API controllers (User, Product, Cart, Order, Payment)
│   │   ├── Services/               # Business logic (e.g., OrderCalculationService)
│   │   └── Models/                 # Eloquent models
│   ├── routes/api.php              # API route definitions
│   ├── database/                   # Migrations, seeders, factories
│   ├── config/
│   └── Dockerfile
├── frontend/                   # Next.js client
│   ├── app/                      # App Router (admin, staff, login)
│   ├── components/               # Shared UI (Radix + Tailwind)
│   ├── actions/                  # Server actions (data fetching, forms)
│   ├── lib/                      # Utilities
│   ├── schemas/                  # Zod validation schemas
│   └── types/
└── README.md                   # This file
```

---

## API Endpoints

All endpoints are under `api/` and use Sanctum bearer token authentication unless noted.

### Auth

| Method | Endpoint | Description | Auth | Rate Limit |
| :--- | :--- | :--- | :--- | :--- |
| POST | `/user/login` | User login | - | `userRate` |
| POST | `/user/register` | Register new user | Yes | `userRate` |
| POST | `/user/logout` | Logout (revoke token) | Yes | `userRate` |
| GET | `/user` | Current user info | Yes | `userRate` |
| GET | `/users` | List all users | Yes (admin) | `userRate` |
| DELETE | `/user/{id}` | Delete a user | Yes | `userRate` |

### Products

| Method | Endpoint | Description | Auth | Rate Limit |
| :--- | :--- | :--- | :--- | :--- |
| GET | `/products` | List products | - | `productRate` |
| POST | `/products` | Create product | - | `productRate` |
| GET | `/products/{id}` | Show product | - | `productRate` |
| PUT/PATCH | `/products/{id}` | Update product | - | `productRate` |
| DELETE | `/products/{id}` | Delete product | - | `productRate` |

### Carts

| Method | Endpoint | Description | Auth | Rate Limit |
| :--- | :--- | :--- | :--- | :--- |
| GET | `/carts` | List carts | Yes | `cartRate` |
| POST | `/carts` | Create cart | Yes | `cartRate` |
| GET | `/carts/{id}` | Show cart | Yes | `cartRate` |
| PUT/PATCH | `/carts/{id}` | Update cart | Yes | `cartRate` |
| DELETE | `/carts/{id}` | Delete cart | Yes | `cartRate` |

### Orders & Checkout

| Method | Endpoint | Description | Auth | Rate Limit |
| :--- | :--- | :--- | :--- | :--- |
| GET | `/orders` | List orders | Yes | `checkoutRate` |
| GET | `/orders/summary` | Order summary | Yes | `checkoutRate` |
| POST | `/orders/checkout/cash` | Cash checkout | Yes | `checkoutRate` |
| POST | `/orders/checkout/qr` | Generate KHQR payment | Yes | `checkoutRate` |
| POST | `/orders/checkout/qr/verify` | Verify payment | Yes | `checkoutRate` |
| POST | `/orders/checkout/qr/cancel` | Cancel payment | Yes | `checkoutRate` |

---

## Getting Started

### Prerequisites

- **PHP** 8.2+ with required extensions (pdo, json, mbstring, etc.)
- **Composer** 2.x
- **Node.js** 20+
- **pnpm** (recommended) or npm/yarn
- **SQLite** (for local development) or MySQL/PostgreSQL

### Backend Setup

```bash
cd backend

# 1. Install dependencies
composer install

# 2. Configure environment
cp .env.example .env
php artisan key:generate

# 3. Set up the database (SQLite)
touch database/database.sqlite
php artisan migrate --seed

# 4. Serve the API
php artisan serve
```

The API will be available at `http://localhost:8000` with a `/api` prefix.

### Frontend Setup

```bash
cd frontend

# 1. Install dependencies
pnpm install

# 2. Start the development server
pnpm dev
```

The frontend will be available at `http://localhost:3000`.

> The frontend proxies `/api` requests to the backend. See `frontend/proxy.ts` for configuration.

---

## Configuration

### Backend `.env`

Copy `backend/.env.example` and configure:

```env
APP_URL=http://localhost:8000
DB_CONNECTION=sqlite
SESSION_DRIVER=database
QUEUE_CONNECTION=database
```

### KHQR / Bakong Configuration

To enable KHQR payments, set your merchant credentials in `.env`:

```env
BAKONG_TOKEN='your_bakong_token'
BAKONG_ACCOUNT_ID='user_name@bank'
BAKONG_MERCHANT_NAME='your_name'
BAKONG_MERCHANT_CITY='your_city'
BAKONG_CURRENCY='KHR'
# Optional
store_label='your_store_name'
phone_number='your_phone_number'
bill_number='your_bill_number'
static=True
```

> You can find your `user_name@bank` under your Bakong profile in the mobile app.

---

## Usage

1. Start the **backend** API server (`php artisan serve` in `backend/`)
2. Start the **frontend** dev server (`pnpm dev` in `frontend/`)
3. Open `http://localhost:3000` in your browser
4. Log in or register a new account
5. Begin selling: add products to the cart, process orders, and accept payments via cash or KHQR

---

## Testing

### Backend

```bash
cd backend
php artisan test
```

### Frontend

Run linting and type checking:

```bash
cd frontend
pnpm lint
pnpm build
```

---

## License

This project is licensed under the **MIT License**. See the [backend/LICENSE](backend/LICENSE) file for details.