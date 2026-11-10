# Inventory Manager.

> **Offline-first Point-of-Sale & Inventory Management System for Kenyan retail businesses**

---

## Table of Contents

1. [Application Overview](#1-application-overview)
2. [Features and Functionality](#2-features-and-functionality)
3. [User Interface and Navigation](#3-user-interface-and-navigation)
4. [Important User Workflows](#4-important-user-workflows)
5. [Technical Architecture](#5-technical-architecture)
6. [Important Implementation Details](#6-important-implementation-details)
7. [Application Demonstration Reference](#7-application-demonstration-reference)

---

## 1. Application Overview

### Application Name and Identity

| Field | Value |
|---|---|
| **Display Name** | Inventory Manager. *(note the trailing period)* |
| **Internal Package Name** | `vineyard-electronicstauri` |
| **Bundle Identifier** | `com.maina.vineyard-electronicstauri` |
| **Current Version** | `0.10.2` |
| **Target Platform** | Windows (NSIS installer) |

### Purpose

Inventory Manager. is a fully offline, desktop Point-of-Sale (POS) and inventory management system built for small-to-medium retail businesses operating in Kenya. It enables shop owners and cashiers to manage their product catalogue, process sales, generate receipts, and analyse business performance — entirely on a local machine with no internet connection required after installation.

### Target Users

| Role | Description |
|---|---|
| **Shop Owner / Admin** | Has full access to all features, including reports, settings, user management, licence management, and sensitive data operations |
| **Cashier / Staff** | Has access to checkout, inventory browsing, and sales history; restricted from admin-only sections |

### Core Problems Solved

- Eliminates dependence on cloud connectivity for daily retail operations
- Provides structured product and inventory management with stock tracking
- Automates receipt generation (on-screen, thermal printer, or PDF)
- Offers sales analytics and exportable reports for business decision-making
- Supports multi-user access with role-based restrictions from a single Windows PC

### Technology Stack

| Layer | Technology |
|---|---|
| **Desktop Framework** | [Tauri 2](https://tauri.app/) (Rust + WebView) |
| **Frontend UI** | React 19 + TypeScript |
| **Styling** | TailwindCSS 4 |
| **Build Tool** | Vite 8 |
| **Routing** | React Router 7 |
| **Backend / Commands** | Rust (Tauri IPC command handlers) |
| **Database** | SQLite via `rusqlite` (WAL mode, foreign keys enforced) |
| **PDF Generation** | `jsPDF` |
| **Excel Export** | `rust_xlsxwriter` (Rust backend) |
| **Excel Import** | `calamine` (Rust backend) |
| **Password Hashing** | `argon2` (Argon2id variant) |
| **Icons** | `lucide-react` |
| **Animations** | `motion` (Framer Motion) |
| **Installer** | NSIS (Windows) |
| **Auto-Updater** | Tauri Updater plugin (GitHub Releases endpoint) |

---

## 2. Features and Functionality

### ✅ Checkout / Point of Sale

The core transaction workflow. Users process customer sales by selecting products, applying discounts, choosing a payment method, and completing the sale.

**Key capabilities:**
- Product search by name or SKU
- Barcode scanning support (keyboard-triggered scan event)
- Cart management (add, update quantity, remove items)
- Discount application (percentage or fixed amount, with configurable caps)
- Payment methods: **Cash**, **Card**, **Mobile Money (M-Pesa)**
- Cash tendered and change calculation
- Payment reference field (for card/M-Pesa transactions)
- On-screen receipt preview immediately after sale completion
- Auto-deduction of sold units from inventory stock

### ✅ Product / Inventory Management

A searchable, filterable list of all products in the catalogue.

**Key capabilities:**
- View all products with name, SKU, category, selling price, cost price, and current stock
- Search by product name or SKU
- Filter by category and stock status (In Stock / Low Stock / Out of Stock)
- Paginated display (7 products per page)
- **Product Details Drawer** (slide-in panel): view full product details and edit product fields
- Bulk delete of selected products
- Low stock threshold configurable (stored in `localStorage`)

### ✅ Product Import (Excel Bulk Upload)

Allows importing a large product catalogue at once using an Excel spreadsheet.

**Key capabilities:**
- Drag-and-drop `.xlsx` file upload
- Preview of parsed rows before confirming import
- Row-level validation with error display
- Downloadable Excel import template
- Duplicate SKU detection and resolution
- Route: `/product-import`

### ✅ Sales History

A chronological log of all completed transactions.

**Key capabilities:**
- List of all sales with invoice number, date/time, cashier, payment method, and total
- Search by invoice number or cashier name
- Filter by date range (Today, Yesterday, This Week, This Month, Custom Range)
- Filter by payment method (All, Cash, Card, Mobile Money)
- Pagination (15 sales per page)
- **Sale Details Drawer**: tap a sale to view full line items and payment info
- Re-print any past receipt (triggers Receipt modal)
- Export filtered sales to Excel

### ✅ Reports & Analytics *(Admin only)*

Business performance dashboard with KPIs and visual charts.

**Key capabilities:**
- Date range selection: Today, Yesterday, This Week, Last 7 Days, This Month, Last 30 Days, This Year, Custom Range
- **Summary stat cards**: Total Revenue, Total Transactions, Average Transaction Value, Total Discount Given
- **Daily sales bar chart** (revenue by day over selected period)
- **Payment method breakdown** (Cash / Card / M-Pesa split)
- **Top-selling products table** (by quantity and revenue)
- **Inventory valuation summary** (total stock value, total cost, estimated profit margin)
- Export button opens the sales export dialog

### ✅ Receipt Generation

A unified receipt system that ensures consistent output across all delivery methods.

**Key capabilities:**
- **On-screen preview modal**: styled receipt with shop branding
- **Thermal printing**: sends receipt to OS native print dialog via hidden iframe; supports **58 mm** and **80 mm** paper widths
- **PDF export**: generates a dynamically sized PDF using `jsPDF`, saved via native OS save dialog
- Paper width preference persisted in `localStorage` (`pos_receipt_paper_width`)
- Receipt content: shop name, tagline, address, phone, invoice number, date/time, cashier name, customer name ("Walk-in Customer" if unspecified), itemised list, subtotal, discount, tax, total, payment method, amount tendered (cash), change (cash), payment reference, footer note

### ✅ Excel Export (Sales Data)

Exports sales records to a structured Excel file for accounting and record-keeping.

**Key capabilities:**
- Triggered from Sales History or Reports pages
- Date range presets: Today, Yesterday, Last 7 Days, Last 30 Days, This Month, Last Month, Custom Range
- Generated in Rust using `rust_xlsxwriter`
- Output file has **two sheets**:
  - **Transactions** — one row per sale with KPI summary block at top (total sales count, gross revenue, total discount, tax collected, net revenue), autofilter enabled, freeze panes on header row
  - **Sale Items** — one row per line item across all exported sales, with foreign key to transaction
- Timestamps formatted in Kenya Standard Time (EAT, UTC+3)

### ✅ Multi-User Authentication

Role-based access control with secure credential management.

**Key capabilities:**
- Two roles: `ADMIN` (full access) and `USER` (cashier access)
- Maximum **5 users** per installation (1 admin + 4 normal users), enforced by database trigger
- Passwords hashed with **Argon2id** with per-user random salt
- Sessions stored as **SHA-256 tokens** in the `sessions` table; 30-day session lifetime
- Login screen shown when no active session exists
- Each sale records the cashier's user ID and full name

### ✅ First-Time Setup Wizard

A 3-step onboarding flow shown on the very first launch (before any user or settings exist).

**Steps:**
1. **Welcome screen** — introduces the application
2. **Shop Details** — collects shop name, tagline, address, phone number
3. **Admin Credentials** — sets the first admin username and password

Setup completion is recorded in the `shop_settings` table (`setup_complete = 1`).

### ✅ Settings *(Admin only)*

A multi-section configuration page at `/settings`.

| Section | Description |
|---|---|
| **Store Profile** | Shop name, tagline, address, phone, email |
| **Receipt Settings** | Receipt header text, receipt footer text, paper width (58 mm / 80 mm) |
| **Checkout Defaults** | Default payment method, default discount, command delay |
| **Account Security** | Change own password |
| **User Management** | View all users, create new users, deactivate/reactivate users |
| **System Architecture** | Displays SQLite WAL mode status, Argon2id encryption note, offline architecture info |
| **License** | View current licence type, expiry, device binding, licence status details |
| **Software Updates** | Check for updates, download and install with progress indicator |
| **Danger Zone** | Permanently delete all products (requires typing `DELETE ALL PRODUCTS` to confirm); products with past sales are archived rather than deleted; categories are preserved |

### ✅ Software Updates

Automatic update checking integrated into the application lifecycle.

**Key capabilities:**
- Checks GitHub Releases endpoint silently on startup (3-second delay after launch)
- If an update is available, a **badge appears on the Settings sidebar icon**
- Update can be downloaded and installed from the Software Updates section in Settings
- Real-time download progress displayed
- Application relaunches automatically after installation

### ✅ License System

An Ed25519 signed software licensing system with device binding.

**Licence types:**

| Type | Duration |
|---|---|
| `Demo1d` | 1 day |
| `Trial7d` | 7 days |
| `Monthly30d` | 30 days |
| `Quarterly90d` | 90 days |
| `Yearly365d` | 365 days |
| `Lifetime` | No expiry |
| `Custom` | Configurable dates |

**Licence states:** `Active`, `Expired`, `Missing`, `DeviceMismatch`, `InvalidSignature`, `ClockRollback`, `Corrupted`

Licences include anti-rollback clock checking to prevent users from winding back system time to extend expired licences.

### ✅ M-Pesa Payments Page *(Admin only)*

A log of all Mobile Money payments recorded during sales.

**Key capabilities:**
- Lists all sales made with payment method `MOBILE_MONEY`
- Clicking a row opens a payment details modal
- Route: `/payments`

> **Note:** This page displays stored payment records only. No live M-Pesa STK Push integration is visible in the frontend source code.

### ⚠️ Dashboard — NOT IMPLEMENTED (Stub)

The route `/dashboard` exists in the router and the sidebar is configured to include it. However, `src/pages/DashBoard.tsx` renders only the text `"DashBoard"` with no functionality. It is a placeholder.

### ⚠️ Customers Page — NOT IMPLEMENTED (Stub)

`src/pages/Customers.tsx` renders only the text `"Customers"` with no functionality. It is a placeholder with no route currently linked in the sidebar.

### ⚠️ Tax — Stored But Not Applied

The `shop_settings` table has a `tax_rate` column and the sale record includes a `tax_amount` field. However, inspection of the receipt and checkout source shows tax is always rendered as **0%** in the current UI. The column exists for future use.

---

## 3. User Interface and Navigation

### Overall Layout

The application uses a **two-column layout**:

```
┌────────────────────────────────────────────────────────┐
│  Sidebar (left, collapsible)  │  Main Content Area     │
│  - Logo / App name            │  (page-specific UI)    │
│  - Navigation items           │                        │
│  - User info (bottom)         │                        │
└────────────────────────────────────────────────────────┘
```

The sidebar can be **expanded** (shows icons + labels) or **collapsed** (shows icons only). Collapse state is toggled with a button on the sidebar.

---

### Sidebar Navigation

| Icon | Label | Route | Access |
|---|---|---|---|
| ShoppingCart | Check Out | `/` | All users |
| BarChart2 | Reports | `/reports` | Admin only |
| Package | Inventory | `/inventory` | All users |
| History | History | `/history` | All users |
| Wallet | Payments | `/payments` | Admin only |
| Settings | Settings | `/settings` | Admin only |

Admin-only items are hidden from USER-role accounts.

A **badge (dot)** appears on the Settings icon when a software update is available.

The bottom of the sidebar shows the logged-in user's name and a **Logout** button.

---

### Pages and Their Purposes

| Route | Component | Purpose |
|---|---|---|
| `/` | `Checkout.tsx` | POS terminal — process sales |
| `/inventory` | `Inventory.tsx` | Browse and manage product catalogue |
| `/product-import` | `ProductImport.tsx` | Bulk import products from Excel |
| `/history` | `History.tsx` | View and filter past sales |
| `/reports` | `Reports.tsx` | Sales analytics and KPIs (admin) |
| `/payments` | `Payments.tsx` | M-Pesa payment records (admin) |
| `/settings` | `Settings.tsx` | Application configuration (admin) |
| `/dashboard` | `DashBoard.tsx` | *(Stub — no content)* |

**Standalone / conditional screens (not in sidebar):**

| Screen | Trigger |
|---|---|
| **Login Screen** | No active session found on launch |
| **Setup Wizard** | First launch (`setup_complete = 0` in database) |
| **Launch Screen** | Shown briefly while app initialises |

---

### Dialogs, Drawers, and Modals

| Component | Location | Trigger |
|---|---|---|
| **Receipt Modal** | Checkout, History | After completing a sale; or "Reprint" in History |
| **Product Details Drawer** | Inventory | Clicking a product row |
| **Sale Details Drawer** | History | Clicking a sale row |
| **Payment Details Modal** | Payments | Clicking a payment record |
| **Export Sales Dialog** | History, Reports | Clicking Export button |
| **Bulk Delete Confirmation** | Inventory | Selecting products + clicking Delete |
| **Change Password Modal** | Settings → Account Security | Clicking Change Password |
| **Create User Modal** | Settings → User Management | Clicking Add User |
| **Danger Zone Confirmation** | Settings → Danger Zone | Clicking Delete All Products |
| **Update Progress Modal** | Settings → Software Updates | Clicking Download & Install |

---

### Toast Notification System

A global toast notification bar appears at the top/bottom of the screen to confirm actions (success) or report errors. Toast messages are dismissible and auto-expire.

---

## 4. Important User Workflows

### 4.1 First Launch and Setup

1. Launch the application for the first time
2. The **Setup Wizard** appears automatically (3 steps)
3. **Step 1 — Welcome**: Read the welcome message, click **Get Started**
4. **Step 2 — Shop Details**: Fill in shop name, tagline, address, phone number; click **Next**
5. **Step 3 — Admin Credentials**: Enter admin username and password; click **Complete Setup**
6. The application creates the admin user, saves settings, and marks setup as complete
7. The **Login Screen** appears; log in with the credentials just created

---

### 4.2 Logging In

1. On the login screen, enter your username and password
2. Click **Sign In**
3. If credentials are correct, an authenticated session is created (valid for 30 days)
4. The main application loads; the sidebar reflects your role (admin or cashier)

---

### 4.3 Importing Products via Excel

1. Navigate to **Inventory** (`/inventory`)
2. Click **Import Products** (or navigate directly to `/product-import`)
3. Download the **Excel template** if needed (button on the page)
4. Fill in the template: product name, SKU, category, selling price, cost price, initial stock
5. Drag-and-drop the completed `.xlsx` file onto the upload dropzone, or click to browse
6. A preview table shows all parsed rows with any validation errors highlighted
7. Fix any errors (edit the Excel file and re-upload) or proceed if all rows are valid
8. Click **Confirm Import**
9. Products are created and inventory records initialised; a success toast appears

---

### 4.4 Editing a Product

1. Navigate to **Inventory** (`/inventory`)
2. Find the product using search or filters
3. Click the product row to open the **Product Details Drawer**
4. Edit the desired fields (name, price, cost, stock level, category, etc.)
5. Click **Save**; a success toast confirms the update

---

### 4.5 Managing Stock / Inventory

- **Viewing stock levels:** Inventory page shows current quantity for each product
- **Filtering by stock status:** Use the "Stock" filter dropdown to view In Stock / Low Stock / Out of Stock items
- **Low stock threshold:** Configurable in Settings → Checkout Defaults (stored in `localStorage`)
- **Stock adjustment:** Edit a product's stock quantity via the Product Details Drawer
- **Automatic deduction:** Completing a sale at checkout automatically deducts sold quantities from inventory

---

### 4.6 Completing a Sale (Checkout)

1. Navigate to **Check Out** (`/`)
2. Find products using the search bar (search by name or SKU) or scan a barcode
3. Click a product card or press Enter after scanning to add it to the **cart**
4. Adjust quantities in the cart if needed
5. Apply a discount (optional): enter percentage or fixed amount in the discount field
6. Select **Payment Method**: Cash, Card, or Mobile Money
7. For cash: enter the **Amount Tendered**; the change is calculated automatically
8. For card/M-Pesa: optionally enter a **Payment Reference** (transaction ID)
9. Click **Complete Sale** (or equivalent confirm button)
10. The sale is recorded, inventory is updated, and the **Receipt Modal** appears

---

### 4.7 Printing a Receipt (Thermal Printer)

1. After completing a sale, the Receipt Modal appears automatically
2. Select paper width: **58 mm** or **80 mm** (matches your thermal printer roll)
3. Click **Print**
4. The OS native print dialog opens; select your thermal printer and confirm
5. The receipt prints using the hidden iframe method

---

### 4.8 Exporting a Receipt as PDF

1. After completing a sale, the Receipt Modal appears automatically
2. Click **Export PDF**
3. The OS native save dialog opens; choose a save location and file name
4. The PDF is saved with page dimensions matched to the selected paper width (58 mm or 80 mm)

---

### 4.9 Finding a Previous Sale and Reprinting

1. Navigate to **History** (`/history`)
2. Use filters to narrow results:
   - **Date filter**: Today, Yesterday, This Week, This Month, Custom Range
   - **Search**: type invoice number or cashier name
   - **Payment method**: All, Cash, Card, Mobile Money
3. Click a sale row to open the **Sale Details Drawer**
4. Review line items and payment info
5. Click **Reprint Receipt** in the drawer
6. The Receipt Modal opens with the original sale data; use Print or Export PDF as needed

---

### 4.10 Exporting Sales History to Excel

1. Navigate to **History** (`/history`) or **Reports** (`/reports`)
2. Click **Export** (opens the Export Sales Dialog)
3. Select a date range preset or choose **Custom Range** and enter specific dates
4. Click **Export to Excel**
5. The OS native save dialog opens; choose a save location
6. The file is generated by the Rust backend and saved; it contains two sheets:
   - **Transactions** (one row per sale, with KPI summary at top)
   - **Sale Items** (one row per item across all exported sales)

---

### 4.11 Generating a Sales Report

1. Navigate to **Reports** (`/reports`) *(Admin only)*
2. Use the date range selector at the top to choose a reporting period
3. The page automatically recalculates and displays:
   - Stat cards (Total Revenue, Transactions, Avg Value, Total Discounts)
   - Daily revenue bar chart
   - Payment method breakdown
   - Top-selling products table
   - Inventory valuation summary

---

### 4.12 Configuring Application Settings

1. Navigate to **Settings** (`/settings`) *(Admin only)*
2. Navigate between sections using the section navigation at the top of the page
3. **Store Profile**: Update shop name, tagline, address, phone, email → click **Save**
4. **Receipt Settings**: Update header/footer text, choose paper width → changes apply to future receipts
5. **Checkout Defaults**: Set default payment method, discount, low stock threshold
6. **Account Security**: Click **Change Password** → enter current password, new password, confirm → submit
7. **User Management**: View all users, click **Add User** to create a new cashier account, toggle user active/inactive status
8. **License**: View licence type, device binding, and expiry date
9. **Software Updates**: Click **Check for Updates**; if available, click **Download & Install**

---

### 4.13 Updating the Application

1. On startup, the app silently checks for updates (3-second delay)
2. If an update is found, a **badge** appears on the Settings sidebar icon
3. Navigate to **Settings** → **Software Updates**
4. The available version is displayed; click **Download & Install**
5. A download progress bar shows installation progress
6. After installation, the app automatically relaunches to the new version

---

## 5. Technical Architecture

### Project Structure

```
anwin-shop.tauri/
├── src/                          # Frontend (React + TypeScript)
│   ├── App.tsx                   # Root component, router, providers, launch logic
│   ├── main.tsx                  # Vite entry point
│   ├── update.ts                 # Tauri updater plugin integration
│   ├── pages/
│   │   ├── Checkout.tsx          # POS terminal page
│   │   ├── Inventory.tsx         # Product catalogue page
│   │   ├── History.tsx           # Sales history page
│   │   ├── Reports.tsx           # Analytics/reports page (admin)
│   │   ├── Settings.tsx          # Settings page (admin)
│   │   ├── Payments.tsx          # M-Pesa records page (admin)
│   │   ├── ProductImport.tsx     # Excel bulk import page
│   │   ├── Setup.tsx             # First-time setup wizard
│   │   ├── DashBoard.tsx         # STUB — not implemented
│   │   └── Customers.tsx         # STUB — not implemented
│   ├── components/
│   │   ├── global/
│   │   │   ├── Sidebar.tsx       # Collapsible sidebar navigation
│   │   │   ├── Receipt.tsx       # Receipt modal and standalone receipt component
│   │   │   └── Toast.tsx         # Toast notification system
│   │   ├── settings/
│   │   │   ├── StoreProfileSection.tsx
│   │   │   ├── ReceiptSettingsSection.tsx
│   │   │   ├── CheckoutDefaultsSection.tsx
│   │   │   ├── AccountSecuritySection.tsx
│   │   │   ├── UserManagementSection.tsx
│   │   │   ├── LicenseSettingsSection.tsx
│   │   │   ├── SoftwareUpdatesSection.tsx
│   │   │   ├── DangerZoneSection.tsx
│   │   │   ├── SystemArchitectureSection.tsx
│   │   │   ├── ChangePasswordModal.tsx
│   │   │   └── CreateUserModal.tsx
│   │   └── inventory/
│   │       └── ProductDetailsDrawer.tsx
│   └── lib/
│       ├── currency.ts            # formatCurrency() — KES formatting
│       ├── date.ts                # Kenya timezone utilities (EAT, UTC+3)
│       ├── receipt/
│       │   ├── receipt-data.ts    # buildReceiptData() — unified data model
│       │   ├── receipt-pdf.ts     # jsPDF-based PDF generation
│       │   ├── receipt-printer.ts # Thermal print via hidden iframe
│       │   └── receipt-types.ts   # Shared TypeScript types
│       ├── services/              # Service layer (Tauri invoke wrappers)
│       │   └── product.service.ts # (and others)
│       └── tauri/
│           └── interceptor.ts     # Tauri IPC error interceptor
│
├── src-tauri/                     # Rust backend (Tauri)
│   ├── src/
│   │   ├── lib.rs                 # Command registration (invoke_handler)
│   │   ├── database.rs            # SQLite connection, migrations (6 migrations)
│   │   ├── auth.rs                # Authentication, sessions, user management
│   │   ├── products.rs            # Product CRUD, categories, inventory, bulk import
│   │   ├── checkout.rs            # Sale creation, payment recording, stock deduction
│   │   ├── report.rs              # Analytics queries, inventory valuation
│   │   ├── export.rs              # Excel export (rust_xlsxwriter)
│   │   ├── settings.rs            # Shop settings read/write
│   │   └── license/
│   │       ├── mod.rs             # License commands
│   │       └── types.rs           # License type/state enums
│   └── tauri.conf.json            # Tauri config (productName, version, updater, bundle)
│
├── package.json                   # Frontend dependencies
├── vite.config.ts                 # Vite build configuration
└── README.md                      # This file
```

---

### Frontend Architecture

- **React 19** with TypeScript, using functional components and hooks throughout
- **React Router 7** handles client-side navigation between pages
- **Context Providers** wrap the app: authentication context, toast context
- **TailwindCSS 4** for utility-first styling with dark mode support
- **Service layer** (`src/lib/services/`) abstracts all Tauri IPC `invoke()` calls into typed service functions
- **Tauri IPC interceptor** (`src/lib/tauri/interceptor.ts`) provides centralised error handling for all backend calls

---

### Rust Backend Modules

| Module | File | Responsibility |
|---|---|---|
| `database` | `database.rs` | Opens/creates SQLite DB, runs migrations, WAL + FK setup |
| `auth` | `auth.rs` | Login, logout, session validation, user CRUD, password change |
| `products` | `products.rs` | Product CRUD, category CRUD, inventory, barcode, bulk Excel import |
| `checkout` | `checkout.rs` | Create sale, record payment, deduct stock, generate invoice numbers |
| `report` | `report.rs` | Sales summaries, top products, payment breakdown, inventory valuation |
| `export` | `export.rs` | Excel export using `rust_xlsxwriter` |
| `settings` | `settings.rs` | Read/write `shop_settings`, `setup_complete` |
| `license` | `license/mod.rs` | Validate Ed25519-signed licence files, device fingerprint check |

---

### Tauri Commands (IPC)

All commands are registered in `lib.rs` via `invoke_handler`. The frontend calls them using `invoke("<command_name>", args)`.

**Authentication:**
- `login` — validate credentials, create session, return user info
- `logout` — invalidate session token
- `get_current_user` — retrieve currently authenticated user from session token
- `get_all_users` — list all users (admin)
- `create_user` — create a new user (admin)
- `update_user_status` — activate/deactivate a user (admin)
- `change_password` — change authenticated user's password

**Products & Inventory:**
- `get_products` — paginated product list with filters
- `get_product_by_id` — single product details
- `create_product` — create new product
- `update_product` — update product fields
- `delete_product` — delete a single product
- `delete_all_products` — bulk delete entire catalogue (with archiving logic)
- `get_product_count` — count of active products
- `get_categories` — list all categories
- `create_category` — create a new category
- `bulk_import_products` — import products from parsed Excel data
- `get_inventory` — inventory records

**Checkout & Sales:**
- `create_sale` — create a new sale with items and payment record

**Reports:**
- `get_sales_report` — analytics data for a date range (revenue, top products, payment breakdown, daily breakdown, inventory valuation)

**Sales History:**
- `get_sales_history` — paginated and filtered sales list
- `get_sale_by_id` — single sale with all line items and payment info

**Export:**
- `export_sales_to_excel` — generate Excel file for a date range, return bytes for save dialog

**Settings:**
- `get_shop_settings` — retrieve shop settings
- `update_shop_settings` — save shop settings
- `setup_complete` — mark first-time setup as done

**License:**
- `get_license_info` — read and validate licence file, return status and details

---

### Database Schema

The database file is named `anwin-shop.sqlite3` and stored in the Tauri application data directory (Windows: `%APPDATA%\com.maina.vineyard-electronicstauri\`).

SQLite runs in **WAL mode** with **foreign keys enforced**.

Applied in 6 migrations:

#### Migration 1 — Users and Sessions

```sql
users (
  id INTEGER PRIMARY KEY,
  username TEXT UNIQUE NOT NULL,
  full_name TEXT NOT NULL,
  password_hash TEXT NOT NULL,           -- Argon2id hash
  salt TEXT NOT NULL,                    -- per-user random salt
  role TEXT NOT NULL DEFAULT 'USER',     -- 'ADMIN' or 'USER'
  status TEXT NOT NULL DEFAULT 'active', -- added in migration 6
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
)

sessions (
  id INTEGER PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  token TEXT UNIQUE NOT NULL,            -- SHA-256 hex token
  expires_at DATETIME NOT NULL,          -- 30-day sessions
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
)
```

#### Migration 2 — Products and Inventory

```sql
categories (
  id INTEGER PRIMARY KEY,
  name TEXT UNIQUE NOT NULL,
  normalized_name TEXT UNIQUE            -- added in migration 5 for deduplication
)

products (
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL,
  sku TEXT UNIQUE,
  category_id INTEGER REFERENCES categories(id),
  selling_price REAL NOT NULL,
  cost_price REAL,
  description TEXT,
  status TEXT DEFAULT 'active',          -- 'active' or 'archived'
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
)

barcodes (
  id INTEGER PRIMARY KEY,
  product_id INTEGER REFERENCES products(id),
  barcode TEXT UNIQUE NOT NULL
)

inventory (
  id INTEGER PRIMARY KEY,
  product_id INTEGER UNIQUE REFERENCES products(id),
  quantity INTEGER NOT NULL DEFAULT 0,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
)
```

#### Migration 3 — Sales and Transactions

```sql
sales (
  id INTEGER PRIMARY KEY,
  invoice_number TEXT UNIQUE NOT NULL,   -- Format: INV-YYYYMMDD-XXXX
  subtotal REAL NOT NULL,
  discount_amount REAL DEFAULT 0,
  tax_amount REAL DEFAULT 0,
  total_amount REAL NOT NULL,
  payment_method TEXT NOT NULL,          -- 'CASH', 'CARD', 'MOBILE_MONEY'
  status TEXT DEFAULT 'completed',
  user_id INTEGER REFERENCES users(id),  -- added in migration 6
  user_full_name TEXT,                   -- denormalised cashier name
  customer_name TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
)

sale_items (
  id INTEGER PRIMARY KEY,
  sale_id INTEGER REFERENCES sales(id),
  product_id INTEGER REFERENCES products(id),
  product_name TEXT NOT NULL,            -- denormalised (protects historical receipts)
  sku TEXT,
  quantity INTEGER NOT NULL,
  unit_price REAL NOT NULL,
  line_total REAL NOT NULL
)

payments (
  id INTEGER PRIMARY KEY,
  sale_id INTEGER REFERENCES sales(id),
  payment_method TEXT NOT NULL,
  amount REAL NOT NULL,
  reference TEXT,                        -- card/M-Pesa transaction reference
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
)

stock_movements (
  id INTEGER PRIMARY KEY,
  product_id INTEGER REFERENCES products(id),
  movement_type TEXT NOT NULL,           -- 'SALE', 'ADJUSTMENT', 'IMPORT', etc.
  quantity_change INTEGER NOT NULL,
  reference_id INTEGER,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
)
```

#### Migration 4 — Shop Settings

```sql
shop_settings (
  id INTEGER PRIMARY KEY DEFAULT 1,
  shop_name TEXT,
  tagline TEXT,
  address TEXT,
  phone TEXT,
  email TEXT,
  currency TEXT DEFAULT 'KES',
  tax_rate REAL DEFAULT 0,
  receipt_header TEXT,
  receipt_footer TEXT,
  setup_complete INTEGER DEFAULT 0
)
```

#### Migration 5 — Category Normalisation

Adds `normalized_name` to `categories` for case-insensitive deduplication.

#### Migration 6 — User Account Limits

Adds `status` column to `users`. Adds a database-level trigger enforcing the maximum of 5 total users (1 admin + 4 normal users).

---

### Frontend–Backend Communication

All frontend→backend calls use the **Tauri IPC bridge**:

```typescript
import { invoke } from "@tauri-apps/api/core";

// Example: get sales report
const report = await invoke("get_sales_report", { from: "2026-01-01", to: "2026-11-10" });
```

The service layer (`src/lib/services/`) wraps every `invoke()` call with typed input/output interfaces and passes errors up to the UI for toast display.

---

### Offline-First Architecture

- **No server required**: the application functions entirely without internet access after installation
- **All data local**: SQLite database stored on the user's machine in the Tauri app data directory
- **Internet only needed for**: software updates (GitHub Releases) and licence activation (if applicable)
- **Data persistence**: sales, products, settings, and sessions persist indefinitely on disk
- **No sync**: there is no cloud synchronisation; each installation is independent

---

## 6. Important Implementation Details

### Currency Handling

Currency is formatted using `Intl.NumberFormat` hardcoded to **KES (Kenyan Shillings)**:

```typescript
// src/lib/currency.ts
new Intl.NumberFormat("en-KE", { style: "currency", currency: "KES" })
```

> **Limitation:** Although the `shop_settings` table has a `currency` column (configurable), the `formatCurrency()` function ignores it and always produces KES-formatted output. The currency setting has no effect on the UI.

---

### Date and Time Handling

All dates and times are displayed and stored relative to **Kenya Standard Time (EAT, UTC+3, Africa/Nairobi)**. This is hardcoded throughout the application to guarantee consistent behaviour regardless of the Windows system clock timezone.

**Frontend (`src/lib/date.ts`):**
- `parseToUtcDate()` — normalises SQLite UTC strings (`YYYY-MM-DD HH:MM:SS`) to proper UTC Date instances
- `formatKenyaDateTime()` — formats timestamps using `Intl.DateTimeFormat` with `timeZone: "Africa/Nairobi"`
- `formatKenyaDate()` — formats date portion only (e.g. `10/11/2026`)
- `formatKenyaTime()` — formats time portion only (e.g. `08:40 am`)
- `getKenyaDateParts()` — extracts year/month/day/hour in EAT
- `getKenyaReportDateRange()` — computes "Today", "This Week", "This Month" etc. in Kenyan calendar time
- `getKenyaHistoryDateBounds()` — computes date filter bounds for History page
- `getKenyaExportPresetDateRange()` — computes date bounds for Excel export presets
- `validateDateRange()` — validates custom date range inputs

**Backend (Rust):**
- Timestamps stored as UTC in SQLite using `CURRENT_TIMESTAMP`
- Report and export queries adjust with `+3 hours` SQL expressions to group/format in EAT
- Invoice date component uses Kenya date at the time of sale

---

### Invoice Number Format

Invoice numbers are generated in the Rust checkout module:

```
INV-{YYYYMMDD}-{4-digit-random}
```

- `YYYYMMDD` is the Kenya date at the time of sale
- 4-digit suffix is randomly generated (not sequential)
- Stored as `UNIQUE` in the `sales` table; collision retry logic is in place

---

### Printing Implementation

Thermal receipt printing uses a **hidden HTML iframe** technique:

1. `buildReceiptData()` assembles a typed receipt data object
2. The receipt HTML is rendered into a hidden `<iframe>` in the DOM
3. `iframe.contentWindow.print()` is called, triggering the OS native print dialog
4. The user selects their thermal printer in the OS dialog and confirms
5. The iframe approach preserves browser CSS and allows precise styling for 58 mm / 80 mm paper

Paper width preference is stored in `localStorage` under key `pos_receipt_paper_width`.

---

### PDF Generation

Receipt PDFs are generated using **jsPDF** in the browser:

- `src/lib/receipt/receipt-pdf.ts` builds the PDF
- Page width matches the selected paper width (58 mm or 80 mm)
- Page height is calculated dynamically based on content length (not fixed)
- Font: Courier (monospace) for thermal-style appearance
- After generation, `jsPDF.save()` triggers the OS native file save dialog

---

### Excel Export Implementation

Excel files are generated entirely in the **Rust backend** using `rust_xlsxwriter`:

- Two worksheets per file: **Transactions** and **Sale Items**
- **Transactions sheet** features:
  - KPI summary block at the top (total sales, gross revenue, total discount, tax, net revenue)
  - Autofilter on the header row
  - Freeze panes: first row frozen for scrolling
  - One row per completed sale
- **Sale Items sheet**: one row per line item, with sale ID as foreign key reference
- Timestamps formatted in EAT (+3 hours offset applied in SQL)
- File bytes returned to frontend, which triggers OS save dialog

---

### Password Security

- All passwords hashed with **Argon2id** (memory-hard, resistant to brute force and GPU attacks)
- Each user has a **unique random salt** stored separately in the `users` table
- Session tokens are **SHA-256 hashes** of a random 32-byte value
- Sessions expire after **30 days** (checked server-side on each authenticated request)

---

### License System

- Licence files are signed with **Ed25519** (elliptic curve digital signature)
- Each licence is **device-bound**: a device fingerprint is computed at runtime and compared to the licence's target device field
- **Clock rollback detection**: the system checks if the current time is suspiciously earlier than previously recorded timestamps, invalidating the licence if rollback is detected
- Licence validation states include: `Active`, `Expired`, `Missing`, `DeviceMismatch`, `InvalidSignature`, `ClockRollback`, `Corrupted`

---

### Application Update Mechanism

- Tauri Updater plugin checks the GitHub Releases endpoint: `https://github.com/mainadavid-cmyk/offline-pos-releases`
- Check runs **silently in the background** 3 seconds after app launch
- If `tauri-plugin-updater` finds a newer version, `updateAvailable` state is set to `true`
- A visual badge appears on the Settings sidebar icon
- From Settings → Software Updates, the user can initiate download; progress is shown in real time
- After installation, `relaunch()` is called automatically

---

### Local Storage Keys

The following data is stored in browser `localStorage` (not SQLite):

| Key | Purpose |
|---|---|
| `pos_receipt_paper_width` | Receipt paper width (`"58mm"` or `"80mm"`) |
| *(checkout defaults)* | Default payment method preference |
| *(low stock threshold)* | Low stock quantity threshold for Inventory filter |
| *(command delay)* | Debounce delay for search inputs |

---

### Receipt Data Model

The `buildReceiptData()` function in `src/lib/receipt/receipt-data.ts` assembles a unified `ReceiptData` object from a sale record and shop settings. This single model is consumed by all three output paths (on-screen preview, thermal print, PDF export), guaranteeing identical data across all delivery methods.

**Defaults applied when data is missing:**
- Cashier name → `"Staff"`
- Customer name → `"Walk-in Customer"`
- Shop name → `"Inventory Manager"`
- Receipt footer → `"Thank you for shopping with us!"`
- Paper width → reads from `localStorage`, defaults to `"80mm"`

---

### Known Limitations

| Limitation | Details |
|---|---|
| **Windows only** | NSIS installer target; no macOS or Linux builds configured |
| **Currency hardcoded to KES** | `formatCurrency()` always uses KES regardless of `currency` setting |
| **Tax always 0%** | `tax_rate` stored in settings but checkout and receipts always compute 0 tax |
| **Dashboard is a stub** | The `/dashboard` route renders no content |
| **Customers page is a stub** | `Customers.tsx` renders no content and has no sidebar link |
| **No M-Pesa STK Push** | M-Pesa payments are manually recorded; no live payment gateway |
| **Single-machine only** | No multi-machine sync; each installation is independent |
| **No backup/restore UI** | No dedicated backup or restore feature is implemented |

---

## 7. Application Demonstration Reference

This section provides a complete guide for demonstrating the application, in a recommended sequence, with all navigation paths, key interactions, and prerequisites.

---

### Prerequisites Before Demonstration

1. The application must be installed on a Windows machine
2. Have a valid licence activated (or use a Trial/Demo licence)
3. Complete the first-time setup wizard (admin account + shop details)
4. Prepare sample product data: an Excel file using the app's import template, with 10–20 products across 3–5 categories
5. *(Optional)* Have a thermal receipt printer (58 mm or 80 mm) connected and set as a Windows printer
6. *(Optional)* Prepare a folder on the desktop for saving exported files (PDF receipts, Excel exports)

---

### Recommended Demonstration Sequence

| Step | Screen | Key Interactions |
|---|---|---|
| 1 | First Launch / Setup Wizard | Show the 3-step onboarding flow |
| 2 | Login Screen | Enter admin credentials and log in |
| 3 | Settings — Store Profile | Show shop name, address, receipt footer configuration |
| 4 | Product Import | Drag-drop Excel file, preview rows, confirm import |
| 5 | Inventory | Browse, search, filter by category and stock status, open product drawer |
| 6 | Checkout — Cash Sale | Search product, add to cart, apply discount, select Cash, enter amount, complete sale |
| 7 | Receipt Modal | Show on-screen receipt, print (thermal), export PDF |
| 8 | Checkout — M-Pesa Sale | Complete another sale using Mobile Money payment method |
| 9 | Sales History | Filter by date, search, open sale drawer, reprint receipt |
| 10 | Export Sales | Trigger export dialog, select date range, export to Excel, show saved file |
| 11 | Reports | Show stat cards, daily chart, payment breakdown, top products |
| 12 | Settings — User Management | Show user list, create a new cashier account |
| 13 | Settings — License | Show licence status, type, expiry |
| 14 | Settings — Software Updates | Show update check, badge on sidebar |
| 15 | Settings — Danger Zone | Show Delete All Products button (do NOT confirm in live demo) |

---

### Screen-by-Screen Navigation and Interactions

#### Screen 1: Setup Wizard
- **How to reach**: Only appears on first launch or when `setup_complete = 0`
- **To simulate in demo**: Use a fresh installation or a database without `setup_complete = 1`
- **Key interactions**:
  - Step 1: Welcome screen with Get Started button
  - Step 2: Shop name, tagline, address, phone input fields
  - Step 3: Admin username and password creation
  - Completion triggers navigation to Login

#### Screen 2: Login Screen
- **How to reach**: Shown when no active session exists (fresh launch or after logout)
- **Key interactions**:
  - Username and password fields
  - Sign In button
  - Incorrect credentials show an error message

#### Screen 3: Checkout (`/`)
- **How to reach**: Default route after login; click **Check Out** in sidebar
- **Key interactions**:
  - Type in search bar to find products by name or SKU
  - Scan a barcode (keyboard input ending in Enter)
  - Click a product card to add it to cart
  - Change quantity by editing the quantity field in the cart
  - Remove an item from the cart using the remove button
  - Open the discount panel; enter percentage or fixed amount
  - Choose payment method: Cash, Card, Mobile Money
  - For Cash: enter tendered amount; observe change calculation
  - Click Complete Sale; Receipt Modal opens

#### Screen 4: Receipt Modal
- **How to reach**: Automatically after completing a sale; or Reprint from History
- **Key interactions**:
  - View fully formatted receipt with shop branding, items, totals
  - Toggle paper width: 58 mm / 80 mm
  - Click **Print** → OS print dialog opens (select thermal printer)
  - Click **Export PDF** → OS save dialog opens; file is saved

#### Screen 5: Inventory (`/inventory`)
- **How to reach**: Click **Inventory** in sidebar
- **Key interactions**:
  - Browse paginated product list (7 per page)
  - Type in search bar (name or SKU)
  - Filter by category using dropdown
  - Filter by stock status (In Stock / Low Stock / Out of Stock)
  - Click a product row → Product Details Drawer slides in
  - In drawer: view all fields; edit and save
  - Select multiple products using checkboxes → Bulk Delete button appears
  - Click **Import Products** → navigates to `/product-import`

#### Screen 6: Product Import (`/product-import`)
- **How to reach**: Button on Inventory page or navigate directly
- **Key interactions**:
  - Download Excel template button
  - Drag-and-drop zone for `.xlsx` file
  - Preview table shows parsed rows and validation errors
  - Confirm Import button (enabled only when no errors)

#### Screen 7: Sales History (`/history`)
- **How to reach**: Click **History** in sidebar
- **Key interactions**:
  - Paginated list of sales (15 per page)
  - Search by invoice number or cashier name
  - Filter by date range (Today, Yesterday, This Week, This Month, Custom)
  - Filter by payment method
  - Click a sale row → Sale Details Drawer slides in with line items
  - Reprint Receipt button in drawer
  - **Export** button → opens Export Sales Dialog

#### Screen 8: Export Sales Dialog
- **How to reach**: Export button on History or Reports page
- **Key interactions**:
  - Select preset date range or Custom Range
  - For Custom: enter start and end dates
  - Click Export to Excel → OS save dialog opens
  - Open saved Excel file to show two worksheets

#### Screen 9: Reports (`/reports`) — *Admin only*
- **How to reach**: Click **Reports** in sidebar (visible to admin only)
- **Key interactions**:
  - Date range selector at top (presets + custom)
  - Stat cards: Total Revenue, Total Transactions, Avg Transaction Value, Total Discounts
  - Daily revenue bar chart
  - Payment method breakdown (Cash/Card/M-Pesa split)
  - Top-selling products table (sorted by revenue)
  - Inventory valuation section (stock value, cost, margin)
  - Export button

#### Screen 10: Payments (`/payments`) — *Admin only*
- **How to reach**: Click **Payments** in sidebar (admin only)
- **Key interactions**:
  - List of all M-Pesa/Mobile Money transactions
  - Click a row → Payment Details Modal with reference number, amount, timestamp

#### Screen 11: Settings (`/settings`) — *Admin only*
- **How to reach**: Click **Settings** in sidebar (admin only)
- **Sub-sections and interactions**:
  - **Store Profile**: edit all fields, save
  - **Receipt Settings**: edit header/footer, choose 58/80 mm
  - **Checkout Defaults**: set defaults
  - **Account Security**: Change Password modal
  - **User Management**: list users, Add User modal, toggle active/inactive
  - **System Architecture**: display-only info cards (SQLite WAL, Argon2id, offline)
  - **License**: show licence type, expiry, device binding
  - **Software Updates**: show version info, update badge if available
  - **Danger Zone**: show the Delete All Products section (do NOT confirm in demo)

---

### Sample Data Requirements

| Data Type | Recommended Amount | Notes |
|---|---|---|
| Product categories | 3–5 | e.g., Electronics, Stationery, Beverages |
| Products | 15–25 | Mix of in-stock, low-stock, and out-of-stock items |
| Initial sales | 5–10 (pre-made) | To demonstrate history, reports, and export |
| Admin user | 1 | Created during setup |
| Cashier user | 1 | Created via User Management in Settings |

---

### Features Requiring Special Hardware or Configuration

| Feature | Requirement |
|---|---|
| **Thermal receipt printing** | A thermal printer (58 mm or 80 mm roll) installed as a Windows printer. Without it, the OS print dialog opens but output goes to a standard printer or PDF driver. |
| **Barcode scanning** | A USB or Bluetooth barcode scanner configured in HID keyboard emulation mode (standard for most retail scanners). |
| **Software update demo** | Internet connection to reach the GitHub Releases endpoint. |

---

### Notes for Demo Facilitators

- The **Dashboard (`/dashboard`)** route and **Customers** route exist in the codebase but render no content. Do not navigate to them in a demonstration.
- **Tax is always 0%** in receipts and reports; this is by design in the current version.
- The **currency is always KES**; changing the currency field in settings has no visual effect.
- The **Danger Zone** confirmation requires typing `DELETE ALL PRODUCTS` exactly. Show the UI; do not confirm.
- When reprinting a historical receipt, the receipt data is fully reconstructed from the database — no data loss risk.
- The app uses a **monospace (Courier) font** for PDF receipts to match thermal printer output style.
- All **dates and times** in the UI are in EAT (Kenya Standard Time), regardless of the system clock timezone of the demo machine.
- The sidebar shows **admin-only items** (Reports, Payments, Settings) only when logged in as an ADMIN user. Log in as admin for a full demo.
- The **update badge** on the Settings icon only appears if a newer version exists on the GitHub Releases endpoint. If demonstrating this, ensure internet access and that the installed version is not the latest.

---

*End of README — Generated from source code inspection of version 0.10.2*
