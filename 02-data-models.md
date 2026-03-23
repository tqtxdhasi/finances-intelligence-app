
---

## 📄 File 3: `02-data-models.md`

```markdown
# 02 – Data Models & Database Schema

All tables use UUID primary keys. Timestamps are stored as UTC.

## Entity‑Relationship Diagram

```mermaid
erDiagram
    receipts ||--o{ items : contains
    items }o--|| categories : belongs_to
    products ||--o{ product_aliases : has
    products }o--|| categories : belongs_to
    categories ||--o{ categories : parent

    receipts {
        uuid id PK
        text merchant
        date date
        time time
        text original_currency
        decimal original_total
        decimal converted_total
        decimal exchange_rate
        text image_url
        timestamp created_at
        timestamp updated_at
    }

    items {
        uuid id PK
        uuid receipt_id FK
        text name
        text normalized_name
        decimal quantity
        text unit
        decimal original_price
        decimal converted_price
        uuid category_id FK
    }

    products {
        uuid id PK
        text canonical_name
        uuid category_id FK
        text preferred_unit
        timestamp created_at
    }

    product_aliases {
        uuid id PK
        uuid product_id FK
        text alias
    }

    categories {
        uuid id PK
        text name
        uuid parent_id FK
        timestamp created_at
    }
Table Definitions
receipts
Column	Type	Constraints	Description
id	UUID	PRIMARY KEY	
merchant	TEXT	NOT NULL	Store name
date	DATE	NOT NULL	Purchase date (YYYY-MM-DD)
time	TIME		Optional time
original_currency	TEXT	NOT NULL, length 3	ISO 4217 currency code (e.g., 'EUR')
original_total	DECIMAL(10,2)	NOT NULL	Total in original currency
converted_total	DECIMAL(10,2)	NOT NULL	Total in user's primary currency
exchange_rate	DECIMAL(10,6)	NOT NULL	Rate used at conversion
image_url	TEXT		Remote URL to file (image/PDF)
created_at	TIMESTAMP	NOT NULL	
updated_at	TIMESTAMP	NOT NULL	
items
Column	Type	Constraints	Description
id	UUID	PRIMARY KEY	
receipt_id	UUID	FOREIGN KEY (receipts.id) ON DELETE CASCADE	
name	TEXT	NOT NULL	Original item name as entered
normalized_name	TEXT	NOT NULL	Canonical product name (from products)
quantity	DECIMAL(10,2)	NOT NULL	
unit	TEXT		e.g., 'kg', 'pcs'
original_price	DECIMAL(10,2)	NOT NULL	Price in receipt's original currency
converted_price	DECIMAL(10,2)	NOT NULL	Price in primary currency
category_id	UUID	FOREIGN KEY (categories.id)	Denormalized for quick analytics
products
Column	Type	Constraints	Description
id	UUID	PRIMARY KEY	
canonical_name	TEXT	UNIQUE NOT NULL	Standardized product name
category_id	UUID	FOREIGN KEY (categories.id)	May be NULL
preferred_unit	TEXT		e.g., 'kg'
created_at	TIMESTAMP	NOT NULL	
product_aliases
Column	Type	Constraints	Description
id	UUID	PRIMARY KEY	
product_id	UUID	FOREIGN KEY (products.id) ON DELETE CASCADE	
alias	TEXT	UNIQUE NOT NULL	Case‑insensitive? We'll store as lowercase
categories
Column	Type	Constraints	Description
id	UUID	PRIMARY KEY	
name	TEXT	NOT NULL	
parent_id	UUID	FOREIGN KEY (categories.id) ON DELETE CASCADE	NULL for root
created_at	TIMESTAMP	NOT NULL	
user_settings
A simple key‑value store (can be a JSON field in a dedicated table or a separate table with key/value columns). For V1, we'll use a table:

Column	Type	Constraints	Description
key	TEXT	PRIMARY KEY	e.g., 'primary_currency'
value	TEXT	NOT NULL	JSON string
Default settings:

json
{
  "primary_currency": "PLN",
  "default_view": "list",
  "date_format": "DD/MM/YYYY",
  "theme": "dark"
}
Indexes
For optimal performance:

sql
CREATE INDEX idx_receipts_date ON receipts(date);
CREATE INDEX idx_receipts_merchant ON receipts(merchant) WHERE merchant IS NOT NULL;
CREATE INDEX idx_items_receipt_id ON items(receipt_id);
CREATE INDEX idx_items_normalized_name ON items(normalized_name);
CREATE INDEX idx_product_aliases_alias ON product_aliases(alias);
CREATE INDEX idx_categories_parent_id ON categories(parent_id);
Migrations
Use a migration tool (Knex, Prisma, or plain SQL scripts). Keep all schema changes versioned.

Example migration (Knex):

javascript
exports.up = function(knex) {
  return knex.schema
    .createTable('receipts', table => {
      table.uuid('id').primary();
      table.string('merchant').notNullable();
      table.date('date').notNullable();
      table.time('time');
      table.string('original_currency', 3).notNullable();
      table.decimal('original_total', 10, 2).notNullable();
      table.decimal('converted_total', 10, 2).notNullable();
      table.decimal('exchange_rate', 10, 6).notNullable();
      table.string('image_url');
      table.timestamps(true, true);
    })
    // ... other tables
};
Data Integrity Rules
Referential integrity: Cascade deletes from receipts to items.

Total consistency: original_total must equal sum of original_price of its items. Backend enforces this on save.

Category deletion: If a category is deleted, items/products referencing it will have category_id set to NULL (instead of cascade). This avoids data loss.
