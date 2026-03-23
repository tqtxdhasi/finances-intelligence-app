# 05 – Core Business Logic

This document details the business rules that drive the application.

## Currency Conversion

- **Primary currency** is stored in user settings (default PLN).
- On receipt creation, the user selects the **original currency**.
- The backend fetches the exchange rate from exchangerate.host for the receipt's date (if provided) or real‑time.
- Rate is stored as `exchange_rate` (number).
- Computations:
  - `converted_total = original_total * exchange_rate`
  - For each item: `converted_price = original_price * exchange_rate`
- If the exchange rate API fails, the app shows an error and does not save the receipt.

## Product Normalization

- When an item is added:
  1. Trim and lowercase the entered name.
  2. Look for an exact match in `product_aliases.alias` (case‑insensitive).
  3. If found, use the associated product's `canonical_name` and `category_id`.
  4. If not found, create a new product:
     - `canonical_name` = cleaned name (e.g., remove punctuation, lowercase)
     - `category_id` = NULL (user can assign later)
     - Insert product and also insert an alias for the entered name.
  5. Store `normalized_name` = product.canonical_name, and optionally `category_id` (denormalized) on the item.

**Future enhancement**: Fuzzy matching (Levenshtein) for typos.

## Category Hierarchy

- Categories have a `parent_id` (self‑reference). Root categories have `parent_id = NULL`.
- When filtering by category, include all descendants (recursive query).
- Example SQL to get all descendant IDs:
  ```sql
  WITH RECURSIVE cat_tree AS (
    SELECT id FROM categories WHERE id = ?
    UNION ALL
    SELECT c.id FROM categories c
    INNER JOIN cat_tree ct ON c.parent_id = ct.id
  )
  SELECT id FROM cat_tree;
When a category is deleted, all referencing items/products have category_id set to NULL (no cascade).

Total Consistency
The original_total on a receipt must equal the sum of original_price of its items.

Backend validates this on receipt creation/update. If mismatch, returns 400 error.

This ensures accurate accounting and analytics.

Search & Filtering (Backend)
Search: merchant ILIKE '%term%' OR EXISTS (SELECT 1 FROM items WHERE receipt_id = receipts.id AND name ILIKE '%term%')

Category filter: Use recursive CTE to get category IDs + descendants.

Price range: Filter converted_total between min and max.

Currency filter: Filter original_currency = ?.

Pagination & Caching
Use OFFSET and LIMIT for page‑based pagination.

For infinite scroll, use page parameter.

Cache analytics results in memory (backend) for 5 minutes to reduce database load. Use a simple map with expiry.

Import Validation & Rollback
On bulk import, all operations are wrapped in a database transaction.

If any row fails validation (e.g., missing merchant, negative price), the transaction is rolled back.

Error details are returned to the client, and no data is persisted.

This ensures data integrity even if a single record is invalid.

Export Formats
CSV: Flat file with columns: receipt_id, merchant, date, total, item_name, quantity, unit, price, category.

JSON: Structured array of receipts with nested items.

XML: Similar to JSON but in XML format.

Excel (XLSX): Multiple sheets: "Receipts" and "Items", with proper headers.

Settings Persistence
Settings are stored in the database table user_settings.

Mobile app fetches settings on launch and caches them locally.

When settings are changed, both local cache and remote are updated.
