# 10 – Glossary

| Term | Definition |
|------|------------|
| **API Key** | A token used to authenticate requests to the backend. For V1, it's a fixed secret stored on the device. |
| **Canonical Name** | The standardized, normalized name of a product (e.g., "milk" for "MLEKO 2%"). |
| **Cloud‑First** | The application stores all data in the cloud and requires an internet connection to function. |
| **Currency Conversion** | The process of converting amounts from the receipt's original currency to the user's primary currency using an exchange rate. |
| **Denormalization** | Storing a copy of a foreign key (e.g., category_id on items) to speed up queries, even though it could be derived via a join. |
| **Fuzzy Matching** | Approximate string matching (e.g., Levenshtein distance) to find similar product names. |
| **Hierarchical Category** | A category that can have parent and child categories (e.g., "Dairy" > "Cheese"). |
| **Normalization (Product)** | Mapping raw item names to a canonical product entity. |
| **OCR** | Optical Character Recognition – extracting text from images. |
| **Pagination** | Splitting a large result set into pages (or infinite scroll). |
| **Primary Currency** | The currency in which all analytics and totals are displayed. |
| **Rollback** | Undoing a transaction if any part fails, ensuring data consistency. |
| **SQL Injection** | A security vulnerability where malicious SQL is injected via user input. We prevent it by using parameterized queries. |
