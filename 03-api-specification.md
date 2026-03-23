# 03 – API Specification

Base URL: `https://your-backend.com/v1`  
Authentication: API key in header `X-API-Key: <key>`  
All responses are JSON.

## Common Response Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 400 | Bad request (validation error) |
| 401 | Unauthorized (invalid/missing API key) |
| 404 | Not found |
| 500 | Server error |

## Endpoints

### Receipts

#### `GET /receipts`
List receipts with search, filter, sort, pagination.

**Query Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| search | string | Search merchant and item names |
| date_from | date (ISO) | Filter receipts with date >= |
| date_to | date | Filter receipts with date <= |
| category_ids | string | Comma‑separated category UUIDs (include child categories) |
| min_total | number | Minimum converted total |
| max_total | number | Maximum converted total |
| currency | string | Filter by original currency |
| sort_by | string | `date`, `total`, `merchant` (default `date`) |
| sort_order | string | `asc` or `desc` (default `desc`) |
| page | int | Page number (default 1) |
| limit | int | Items per page (default 20) |

**Response**:
```json
{
  "data": [
    {
      "id": "uuid",
      "merchant": "Supermarket",
      "date": "2025-03-23",
      "time": "14:30:00",
      "original_currency": "EUR",
      "original_total": 45.50,
      "converted_total": 195.20,
      "exchange_rate": 4.29,
      "image_url": "https://...",
      "created_at": "2025-03-23T12:00:00Z",
      "updated_at": "..."
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "total_pages": 5
  }
}
GET /receipts/:id
Get single receipt with its items.

Response:

json
{
  "id": "uuid",
  "merchant": "Supermarket",
  "date": "2025-03-23",
  "time": "14:30:00",
  "original_currency": "EUR",
  "original_total": 45.50,
  "converted_total": 195.20,
  "exchange_rate": 4.29,
  "image_url": "https://...",
  "items": [
    {
      "id": "uuid",
      "name": "Milk",
      "normalized_name": "milk",
      "quantity": 2,
      "unit": "l",
      "original_price": 2.50,
      "converted_price": 10.73,
      "category_id": "cat-uuid"
    }
  ]
}
POST /receipts
Create a new receipt. Multipart form‑data.

Fields:

merchant (string, required)

date (YYYY-MM-DD, required)

time (HH:MM:SS, optional)

original_currency (string, required, e.g., 'EUR')

original_total (number, required) – will be validated against sum of items

items (JSON string) – array of:

json
{
  "name": "Milk",
  "quantity": 2,
  "unit": "l",
  "original_price": 2.50,
  "category_id": "optional"
}
file (file, optional) – image or PDF

Response (201):

json
{
  "id": "new-receipt-uuid"
}
PUT /receipts/:id
Update an existing receipt (full update). Similar payload as POST.

DELETE /receipts/:id
Delete receipt and all its items.

Categories
GET /categories
Returns hierarchical list.

Response:

json
[
  {
    "id": "uuid",
    "name": "Dairy",
    "children": [
      { "id": "...", "name": "Cheese", "children": [] }
    ]
  }
]
POST /categories
json
{ "name": "New Category", "parent_id": "optional-uuid" }
PUT /categories/:id
json
{ "name": "Updated" }
DELETE /categories/:id
Sets category_id to NULL on referencing items/products.

Products
GET /products
Search products by name.

Query:

search (string) – search canonical_name and aliases

Response:

json
[
  {
    "id": "uuid",
    "canonical_name": "milk",
    "aliases": ["mléko", "milch"],
    "category_id": "cat-uuid"
  }
]
POST /products
json
{
  "canonical_name": "milk",
  "category_id": "optional",
  "aliases": ["mléko", "milch"]
}
PUT /products/:id
Update name, category, aliases.

DELETE /products/:id
Deletes product and its aliases. Items referencing it keep the normalized_name but lose category association.

Analytics
GET /analytics/expenses
Returns total expenses over time, grouped dynamically.

Query:

from (date, required)

to (date, required)

group_by (string: day, week, month, quarter, year) – optional, backend chooses based on range if omitted

Response:

json
{
  "data": [
    { "date": "2025-03-01", "total": 120.50 },
    { "date": "2025-03-02", "total": 80.00 }
  ],
  "total": 200.50
}
GET /analytics/top/categories
Top categories by total spent.

Query:

from (date)

to (date)

limit (int, default 10)

Response:

json
[
  {
    "category_id": "uuid",
    "name": "Dairy",
    "total_spent": 450.00,
    "percentage": 45
  }
]
GET /analytics/top/products
Similar to top categories, returns product details.

GET /analytics/product/:id
Detailed stats for a product.

Response:

json
{
  "id": "uuid",
  "canonical_name": "milk",
  "total_spent": 150.00,
  "total_quantity": 30,
  "average_price": 5.00,
  "average_frequency_days": 3.2,
  "last_purchase_date": "2025-03-22",
  "history": [
    {
      "date": "2025-03-20",
      "quantity": 2,
      "price": 10.00,
      "receipt_id": "receipt-uuid"
    }
  ]
}
GET /analytics/category/:id
Similar to product detail but includes top products and history.

Export & Import
GET /export
Generates and returns an export file.

Query:

format (csv, json, xml, xlsx)

from (date)

to (date)

Response: File download (content‑disposition).

POST /import
Accepts a file (multipart) for bulk import. The file format can be CSV, JSON, XML, or Excel. The backend validates and inserts in a single transaction; on any error, the entire transaction is rolled back.

Fields:

file (file, required)

Response:

json
{
  "success": true,
  "receipts_imported": 10,
  "items_imported": 45
}
Settings
GET /settings
Returns current user settings.

Response:

json
{
  "primary_currency": "PLN",
  "default_view": "list",
  "date_format": "DD/MM/YYYY",
  "theme": "dark"
}
PUT /settings
Updates settings. Partial updates allowed.

Payload:

json
{
  "primary_currency": "USD"
}
Authentication
For V1, we use a simple API key. The backend expects the key in the X-API-Key header. The key can be generated once (e.g., a UUID) and stored in the mobile app securely (Expo SecureStore). In a future version, we may add JWT for multi‑user.

Error Handling
All errors return a JSON object:

json
{
  "error": "Validation failed",
  "details": [
    { "field": "merchant", "message": "Merchant is required" }
  ]
}
