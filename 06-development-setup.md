# 06 – Development Setup

## Prerequisites

- Node.js (v18+)
- npm or yarn
- PostgreSQL (local for development)
- Expo CLI (for mobile)
- Git

## Backend Setup

1. Clone the repository.
2. Navigate to `backend/`.
3. Install dependencies:
   ```bash
   npm install
Create .env file:

text
PORT=3000
DATABASE_URL=postgresql://user:pass@localhost:5432/receipt_app
API_KEY=your-secret-api-key
STORAGE_BUCKET=your-bucket
STORAGE_ACCESS_KEY=...
STORAGE_SECRET_KEY=...
CURRENCY_API_URL=https://api.exchangerate.host
Run migrations:

bash
npm run migrate:up
Start the server:

bash
npm run dev
Mobile App Setup
Navigate to mobile/.

Install dependencies:

bash
npm install
Create .env file:

text
API_BASE_URL=http://localhost:3000/v1
API_KEY=your-secret-api-key
Start Expo:

bash
expo start
Run on device/emulator using Expo Go.

Environment Variables
Backend
Variable	Description
PORT	Server port (default 3000)
DATABASE_URL	PostgreSQL connection string
API_KEY	Secret key for API authentication
STORAGE_BUCKET	Cloud storage bucket name
STORAGE_ACCESS_KEY	Access key for cloud storage
STORAGE_SECRET_KEY	Secret key for cloud storage
CURRENCY_API_URL	Base URL for exchange rate API
Mobile
Variable	Description
API_BASE_URL	Backend API URL
API_KEY	Same as backend's API_KEY
Database Migrations
We use Knex.js for migrations. Example command:

bash
knex migrate:make create_receipts_table
Edit the generated file, then run:

bash
knex migrate:latest
For rollback: knex migrate:rollback.

Running Tests
Backend:

bash
npm test
Mobile:

bash
npm test  # if using Jest
Local File Storage
For development, you can use a local file system instead of cloud storage. Configure STORAGE_TYPE=local in .env and set STORAGE_LOCAL_PATH=./uploads. This allows testing without cloud credentials.

Troubleshooting
CORS errors: Ensure backend has CORS enabled for http://localhost:19006 (Expo default).

Database connection: Check PostgreSQL is running and credentials are correct.

API key mismatch: Verify the same key is used in both mobile and backend.
