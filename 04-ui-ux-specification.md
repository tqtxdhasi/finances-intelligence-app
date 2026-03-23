# 04 – UI/UX Specification

## Design Principles

- **iOS‑like**: Rounded corners, system fonts, subtle shadows, smooth animations.
- **Dark theme first**: All screens use dark colors; light theme will be added later.
- **Responsive**: Works on various screen sizes (phone/tablet).
- **Accessible**: Sufficient contrast, touch targets at least 44x44 points.

## Navigation

Bottom tab bar with three items:
- **Receipts** (list icon)
- **Add** (plus icon)
- **Analytics** (chart icon)

On tap, the content changes with a slide/fade animation (using React Navigation’s default transitions).

## Receipts Tab

### Header
- Title: "Receipts"
- Toggle button (list/grid) – changes view mode.
- Search icon – opens search bar inline.
- Filter icon – opens filter modal.

### List View
Each row:
- Left: thumbnail (if image) or placeholder.
- Center: merchant name, date (formatted), total (primary currency).
- Right: optional star (favourite – future).
- Swipe actions: none (V1). Buttons in detail screen.

### Grid View
- Two columns.
- Each card: thumbnail, merchant, total.

### Search Bar
- Expands from header when search icon tapped.
- Live filtering (debounced).
- Cancel button to dismiss.

### Filter Modal
- Bottom sheet (or modal) with:
  - Date range (two date pickers)
  - Category picker (tree selector, multi‑select)
  - Price range (min/max numeric inputs)
  - Currency dropdown (optional)
  - Sort dropdown (date, total, merchant, asc/desc)
- Buttons: Apply, Reset.
- Apply closes modal and triggers new API call.

### Pagination
- **Infinite scroll**: Load more as user scrolls (20 items per page).
- **Page navigation** (optional): For large datasets, may add page buttons in future.

### Receipt Detail Screen
- Navigation: back button (stack).
- Header: merchant, date, total (editable with pencil icon).
- File preview: if image, display with pinch‑zoom; if PDF, use a PDF viewer component (react‑native‑pdf).
- Items section: table with columns: Item, Quantity, Price.
- Actions: Edit (pencil) and Delete (trash) at bottom.

**Edit Mode**: Same as Add screen but pre‑filled; updates via PUT request.

## Add Receipt Tab

### Form
- Scrollable view with sections:
  1. **Basic Info**
     - Merchant (text input)
     - Date (date picker)
     - Time (optional time picker)
     - Original currency (dropdown)
     - Total (numeric input) – optionally auto‑calculated from items.
  2. **File Upload**
     - Button: "Add Image/PDF"
     - Shows preview if selected.
  3. **Items**
     - List of items with summary (name, price).
     - Button: "Add Item" (opens modal).
  4. **Save** button at bottom.

### Add Item Modal
- Modal with form:
  - Name (text input, autocomplete from products)
  - Quantity (numeric)
  - Unit (dropdown or text)
  - Price (numeric)
  - Category (optional, select from tree)
- Buttons: Save, Cancel.

### Validation
- Show inline errors (red text) if required fields missing.
- On save, if total does not match sum of items, show alert and prevent save.

### Save Flow
1. Validate.
2. Show loading indicator.
3. Upload file (if any) → get URL.
4. Call POST /receipts.
5. On success, navigate to Receipts tab and show success toast.
6. On error, show alert with error message; keep form data as draft (state not cleared).

## Analytics Tab

### Time Filter
- Horizontal scroll of chips: Day, Week, Month, Quarter, Year, Custom.
- Custom opens date picker modal.

### Chart
- Line chart using Victory Native.
- X‑axis: dates; Y‑axis: total spent.
- Animated drawing on filter change.
- If range is large (e.g., 2 years), group by month automatically.

### Top Expenses
- Two segmented buttons: "By Category", "By Product".
- List items:
  - Left: category/product name.
  - Right: total spent (primary currency).
  - Progress bar indicating percentage.
- Tap on item → push detail screen.

### Category Detail Screen
- Header: category name, total spent.
- Top products in this category (list with totals).
- History list: receipts (date, total, merchant) – tap to open receipt.

### Product Detail Screen
- Header: product name, total spent, total quantity, average price, average frequency (days), last purchase date.
- Chart: consumption over time (quantity vs date).
- History list: occurrences (date, quantity, price, receipt reference) – tap to open receipt.

## Settings Screen

Accessed via an icon in the header of any screen (or a dedicated tab in future). For V1, we'll use a gear icon in the top‑right of Receipts tab.

Settings options:
- Primary currency (dropdown)
- Default view (list/grid)
- Theme (dark/light) – light placeholder
- Export data (opens export options)
- Import data (file picker)
- API key (display only, not editable)

**Export Options**:
- Format (segmented: CSV, JSON, XML, Excel)
- Date range (from/to)
- Export button → triggers download and share.

## Animations

- Tab transitions: fade/scale (0.2s).
- List item entrance: fade up (0.15s) with stagger.
- Chart drawing: animated stroke.
- Modal presentations: slide up with dim background.
- Button taps: subtle scale down.

## Theming

**Dark theme**:
- Background: #121212
- Surface: #1E1E1E
- Primary text: #FFFFFF
- Secondary text: #AAAAAA
- Accent: #FF9800

**Light theme** (future):
- Background: #F5F5F5
- Surface: #FFFFFF
- Primary text: #212121
- Secondary text: #757575
- Accent: #FF9800

Use React Native's Appearance API to detect system preference but override with user setting.

## Responsive Layout

- Use flexbox.
- For grid view, use `numColumns={2}` in FlatList.
- All text scales with system font size (allow dynamic type).
