<div align="center">
  <h1>🍽️ InnovraDINE</h1>
  <p>A smart QR-based restaurant ordering and growth platform. Customers scan a QR code at their table to instantly access a digital menu, place orders, and pay from their phone. After the meal, they are prompted to leave a Google review or follow the restaurant on social media — turning every dining experience into an opportunity for growth.</p>

  **Live Site:** [innovra-dine.com](https://innovra-dine.com)
</div>

---

## Overview

InnovraDINE solves a core problem for restaurants: happy customers eat, enjoy, and leave — without leaving a review, following the brand, or returning. The platform addresses this by embedding a post-order engagement loop directly into the ordering flow.

The system has two distinct interfaces:

- **Customer App** — QR scan → Welcome screen → Menu → Cart → Order → Post-order engagement
- **Manager Dashboard** — Real-time order management, menu editing, analytics, QR generation, and outlet configuration

---

## ✨ Features

### Customer-Facing

| Feature | Description |
|---|---|
| QR Code Scanning | Scan table QR to instantly open the digital menu — no app download required |
| Dine-In & Takeaway | Supports both dine-in (table number) and takeaway (kiosk) order modes |
| Menu Browsing | Category-filtered menu with veg/non-veg/egg dietary indicators, variants, and images |
| Cart & Checkout | Add items, apply coupons, add cooking notes, and place orders |
| Payment Selection | Pay in Cash (UPI/card via Cashfree, Razorpay, PhonePe — integrated, configurable) |
| Order Confirmation | Real-time order status updates with sequential financial-year order numbers |
| Order History | View past orders across all outlets, with order status tracking |
| Google Review Prompt | After every order, customers are encouraged to leave a Google review |
| Social Follow Prompt | Prompt to follow the restaurant on Instagram/Facebook after dining |
| Coupon/Offers System | Percentage, flat-rate, and BOGO discount coupons with usage limits |
| Customer Auth | Google OAuth + OTP phone number verification |
| Customer Profile | Manage name, phone number, and view account details |
| Favourites | Save favourite menu items per outlet |

### Manager Dashboard

| Feature | Description |
|---|---|
| Real-Time Orders | Live order feed with accept / prepare / ready / deliver / reject actions |
| Menu Management | Add, edit, delete menu items with drag-and-drop category reordering |
| Category Management | Create and reorder menu categories with drag-and-drop |
| Offers Management | Create and manage discount offers with expiry dates and usage limits |
| Sales Analytics | Revenue charts, order counts, and sales breakdowns by category and item |
| QR Code Generator | Generate and download printable QR codes per outlet and table |
| Print KOT | Print Kitchen Order Tickets directly from the dashboard |
| Print Invoice | Generate and print customer invoices (PDF) |
| Printer Settings | Configure thermal printer settings |
| Settlement Reports | Export settlement data as CSV |
| Outlet Configuration | Manage outlet name, address, GSTIN, FSSAI, images, social links, payment config |
| User Management | Create and manage manager accounts (admin only) |
| PDF Menu Import | Import menu items from a PDF using Google Gemini AI |

---

## 🛠️ Tech Stack

### Frontend

| Layer | Technology |
|---|---|
| **Framework** | React 18 + TypeScript |
| **Build Tool** | Vite (with SWC) |
| **Routing** | React Router v6 |
| **Styling** | TailwindCSS + shadcn/ui (Radix UI primitives) |
| **State Management** | React Context (Cart, CustomerAuth, ManagerAuth) |
| **Data Fetching** | TanStack React Query v5 |
| **Charts** | Recharts |
| **Drag & Drop** | @dnd-kit |
| **Forms** | React Hook Form |
| **PDF Generation** | jsPDF + html2canvas |
| **QR Code** | qrcode, qrcode.react, html5-qrcode |
| **File Export** | file-saver, jszip, PapaParse (CSV) |
| **Toast Notifications** | Sonner |
| **Error Tracking** | Sentry |
| **AI Integration** | Google Gemini API (PDF menu import) |

### Backend

| Layer | Technology |
|---|---|
| **Database** | Firebase Firestore |
| **Authentication** | Firebase Auth |
| **File Storage** | Firebase Storage |
| **Hosting** | Firebase Hosting |
| **Cloud Functions** | Python 3.13 (Firebase Functions v2) |
| **Region** | `asia-south2` |

---

## 🔐 Security Architecture

- **Orders are write-only via Cloud Function** — Direct client writes to orders are blocked (`allow create: if false`). All orders go through `place_order`, which enforces server-side price validation — clients cannot tamper with prices.
- **Rate Limiting** — `place_order` enforces a 1-order-per-minute limit per user via Firestore transactions.
- **Coupon Security** — Coupon usage is tracked and validated entirely server-side; clients cannot self-apply or reuse coupons.
- **Role-Based Firestore Rules** — Three roles (`admin`, `manager`, `customer`) with outlet-scoped access enforced in security rules.
- **Private Secrets Isolation** — Payment gateway credentials stored at `outlets/{outletId}/private/secrets`, readable only by admins.
- **Customer Data Isolation** — Customers can only read their own orders and update only the `reviewed` flag.

---

## ☁️ Cloud Functions

All functions deployed to `asia-south2`. Source: `functions/main.py` (Python 3.13).

| Function | Type | Description |
|---|---|---|
| `place_order` | HTTPS | Secure order placement. Validates pricing server-side, applies coupons, enforces rate limits, generates sequential financial-year order numbers atomically via Firestore transaction. |
| `validate_order` | Firestore Trigger | Fires on new order creation. Validates item availability and flags price mismatches. |
| `create_payment_session` | HTTPS | Creates payment sessions supporting Cashfree, Razorpay, and PhonePe via a strategy pattern. |
| `create_admin` | HTTPS | Securely creates an admin user in Firebase Auth with role assignment. Requires `X-Admin-Secret` header. |

### Financial-Year Order Numbers
Order numbers are sequential and reset each April 1. Counter stored at `outlets/{outletId}/counters/orders_{fyStart}_{fyEnd}` and incremented via Firestore transaction for atomicity.

---

## 🗃️ Data Model

All outlet data is scoped under `outlets/{outletId}`.

```
Firestore
│
├── outlets/{outletId}
│   ├── menuItems/{itemId}              # Items (name, price, category, variants, image)
│   ├── categories/{categoryId}         # Categories with sort order
│   ├── orders/{orderId}                # Customer orders
│   ├── offers/{offerId}                # Discount coupons (%, flat, BOGO)
│   ├── reviews/{reviewId}              # Customer star ratings
│   ├── counters/{fy_start_fy_end}      # Sequential order number counter per financial year
│   └── private/secrets                 # Payment gateway credentials (admin only)
│
├── user_roles/{userId}                 # Role: 'admin' | 'manager', outlet scope
├── profiles/{userId}                   # Customer profile (name, phone)
├── user_rate_limits/{userId}           # Rate limiting (Cloud Function managed)
├── coupon_usages/{usageId}             # Per-user coupon usage (Cloud Function managed)
└── coupon_daily_stats/{statsId}        # Daily coupon stats (Cloud Function managed)
```

---

## 📂 Project Structure

```
src/
├── pages/
│   ├── WelcomeScreen.tsx       # Per-outlet QR landing screen
│   ├── Menu.tsx                # Customer menu browsing
│   ├── Cart.tsx                # Cart and checkout
│   ├── OrderConfirmation.tsx   # Real-time order status
│   ├── OrderHistory.tsx        # Customer order history
│   ├── ManagerDashboard.tsx    # Manager order + analytics hub
│   ├── ManagerMenuEditor.tsx   # Menu editing
│   ├── OffersManagement.tsx    # Coupon management
│   ├── OutletManagement.tsx    # Outlet configuration (admin)
│   ├── UserManagement.tsx      # User management (admin)
│   └── ImportMenu.tsx          # AI-powered PDF menu import
├── components/
│   ├── SalesAnalytics.tsx      # Revenue charts and breakdowns
│   ├── AdminQRGenerator.tsx    # QR code generator
│   ├── InvoiceTemplate.tsx     # Invoice PDF template
│   ├── KOTTemplate.tsx         # Kitchen Order Ticket template
│   ├── ReviewDialog.tsx        # Post-order review prompt
│   ├── CouponDialog.tsx        # Coupon selection UI
│   └── ui/                     # 48 shadcn/ui primitive components
├── contexts/
│   ├── CartContext.tsx          # Cart state (items, totals, offers)
│   ├── CustomerAuthContext.tsx  # Customer auth state
│   └── ManagerAuthContext.tsx   # Manager auth state
├── lib/
│   ├── orderSync.ts             # Order creation + real-time sync
│   ├── bogoEngine.ts            # BOGO offer calculation logic
│   └── importMenuJSON.ts        # AI menu import utilities
├── services/
│   ├── auth.service.ts          # Google OAuth + OTP auth
│   ├── menu.service.ts          # Menu CRUD
│   ├── offer.service.ts         # Coupon/offer service
│   ├── settlement.service.ts    # CSV settlement export
│   └── printer/                 # Thermal printer integration
└── types/
    └── outlet.ts                # OutletConfig interface
functions/
├── main.py                      # All Python Cloud Functions
└── requirements.txt
firestore.rules                  # Firestore security rules
firestore.indexes.json           # Composite index definitions
storage.rules                    # Firebase Storage security rules
```

---

## 👥 Role System

| Role | Access |
|---|---|
| `admin` | Full access: all outlets, user management, outlet config, payment secrets |
| `manager` | Outlet-scoped: orders, menu, offers, categories for their assigned outlet |
| `customer` | Read own orders, place orders via Cloud Function, update `reviewed` flag only |

---

## 📝 Note

This project is private and confidential. Source code is not publicly available.

**Contact:** innovratechnologies@gmail.com | [@innovra_dine](https://www.instagram.com/innovra_dine/)
