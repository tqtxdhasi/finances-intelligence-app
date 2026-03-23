# 01 – System Architecture (Refined)

## Overview

The Receipt Intelligence App follows a **cloud‑first** architecture. All data is stored remotely, and the mobile client communicates with a backend API that handles persistence, file storage, and business logic. No local database is used in V1.

## High‑Level Diagram

```mermaid
graph TD
    A[Mobile App<br/>React Native] -->|HTTPS| B[Backend API<br/>Node.js + Express]
    B --> C[(PostgreSQL)]
    B --> D[File Storage<br/>S3 / R2 / Supabase]
    B --> E[Currency API<br/>exchangerate.host]
    C --> F[(receipts, items,<br/>products, categories)]
    D --> G[Images / PDFs]
```

## Component Details

### Mobile App

| Area | Technology / Approach |
|------|----------------------|
| Framework | React Native (Expo) |
| State Management | Zustand (or Redux Toolkit) |
| Navigation | React Navigation (bottom tabs + stack) |
| API Client | Axios (configured with base URL & API key) |
| File Handling | `expo-image-picker`, `expo-document-picker`; upload via `multipart/form-data` |
| Charts | Victory Native (or react-native-chart-kit) |
| Secure Storage | Expo SecureStore (for API key) |

### Backend API

| Area | Technology / Approach |
|------|----------------------|
| Runtime | Node.js + Express |
| Authentication | API key (sent in `X-API-Key` header). Single key per user. |
| Validation | Joi or Zod |
| Currency Conversion | Fetch rates from `exchangerate.host`; cache results for 1 hour |
| File Upload | Multer; store in cloud storage (S3, R2, Supabase) |
| Export | Generate CSV, JSON, XML, Excel on the fly; serve as downloadable file |
| Error Handling | Structured JSON responses with appropriate HTTP status codes |

### Database

| Aspect | Details |
|--------|---------|
| Type | PostgreSQL (or any SQL database) |
| Migrations | Knex or Prisma |
| Indexes | On `receipts.date`, `items.receipt_id`, `product_aliases.alias`, etc. |

### File Storage

| Aspect | Details |
|--------|---------|
| Service | Supabase Storage, Cloudflare R2, or AWS S3 |
| Naming | UUID‑based filenames to avoid collisions |
| URLs | Public URLs (for V1) – data is private to the user but no authentication required to view files. If needed, signed URLs can be used later. |

### Currency API

| Aspect | Details |
|--------|---------|
| Provider | [exchangerate.host](https://exchangerate.host) (free, no API key required) |
| Endpoint | `https://api.exchangerate.host/convert?from={original_currency}&to={primary_currency}&date={date}` |
| Fallback | If date is not provided, use latest rate. If API fails, return error and do not save receipt. |

## Data Flow

### 1. Add Receipt

1. User fills form, selects file (image/PDF).
2. App uploads file to backend (multipart).
3. Backend stores file → receives URL.
4. Backend fetches exchange rate (based on receipt date or real‑time).
5. Backend validates that `original_total` equals sum of item `original_price`.
6. Backend inserts receipt and items into database.
7. Backend returns receipt ID to app.

### 2. List Receipts

- App sends search/filter/sort parameters (page, limit).
- Backend builds SQL query with `WHERE` clauses and `OFFSET`/`LIMIT`.
- Returns paginated results.

### 3. Analytics

- App requests aggregated data (e.g., top categories, product details).
- Backend executes SQL aggregations with optional caching (5‑minute TTL).
- Returns JSON.

### 4. Export

- App requests export with format and date range.
- Backend queries all matching receipts + items, formats file (CSV, JSON, XML, Excel).
- Streams file as response with `Content-Disposition: attachment`.

### 5. Import

- App sends file (multipart) to `/import`.
- Backend parses file, validates each row.
- Wraps entire operation in a transaction. If any row fails, rollback and return error details.

## Security

| Area | Measure |
|------|---------|
| API Key | Stored in Expo SecureStore; sent in `X-API-Key` header for every request. |
| CORS | Restrict to mobile app domain (or specific origins). |
| File Upload | Validate file type (image/jpeg, image/png, application/pdf) and size (e.g., ≤10 MB). |
| No User Accounts | Single user per backend instance; no registration/login. The API key acts as the sole authentication. |

## Extensibility

- The architecture is modular: replace any component (e.g., file storage, currency API) without affecting other parts.
- Future V2 may add local SQLite for offline support; repositories will abstract the data source, allowing seamless sync.

## Key Design Decisions

- **Cloud‑first** – simplifies V1 development and ensures data consistency.
- **Single user** – no need for complex user management; one API key per installation.
- **No local database** – reduces complexity; sync not required.
- **Manual entry first** – OCR is a future enhancement.
- **Precision over convenience** – strict validation (total = sum of items) and transaction‑based imports.
