<div align="center">
  <h1>📋 Order Form Manager</h1>
  <p>A modern, fast, and offline-capable Progressive Web App (PWA) for creating, managing, and sharing order forms efficiently. Built for textile trading businesses in India with multi-tenant company isolation, AI-powered input, and a full admin control panel.</p>
  
  <br />
</div>

## ✨ Features

### Core Order Management
- **📱 Progressive Web App (PWA)**: Installable on Desktop, iOS, and Android. Works perfectly offline.
- **⚡ Fast & Responsive**: Built with React 19 and optimized for all screen sizes (mobile, tablet, desktop).
- **🏭 Multi-Firm Support**: Supports three firm types — `agency`, `aadhat`, and `manufacturer` — with automatic order series segregation by firm and financial year.
- **🔍 Smart Autocomplete**: Fast supplier, customer, and destination searching using indexed master data with fuzzy matching.
- **📤 Easy Sharing**: Generate crisp A5 invoice images and share directly via the native Web Share API (WhatsApp, Email, etc.).
- **🖼️ Visual History**: Browse past orders using a thumbnail sidebar with share-count tracking and offline creation flags.
- **☁️ Cloud Sync**: Powered by Firebase Firestore for real-time data sync across all your devices.

### AI & Smart Input
- **📷 AI Label Scanner**: Camera-based OCR using Google Cloud Vision API with bilingual support (Hindi + English handwriting) to auto-fill supplier name and rate from physical labels.
- **🎙️ Voice Input**: Hindi voice-to-order using Sarvam AI (Indic speech-to-text, `saaras:v3`) with Gemini AI for structured field extraction — supports both global order-level and per-item voice entry.

### Authentication & Multi-Tenancy
- **🔐 Google OAuth Login**: Firebase Authentication with Google Sign-In (persistent sessions via `browserLocalPersistence`).
- **🏢 Multi-Tenant Architecture**: Full company isolation — each company has its own Firestore data namespace, configuration, and user roster.
- **👥 Role-Based Access Control**: Three roles — `staff`, `company_admin`, and `super_admin` — with route-level guards enforced in the UI.
- **🛡️ Offline Identity Cache**: User company/role resolved from `localStorage` cache when Firebase is unreachable, enabling seamless offline use.

### License & Admin
- **📜 License Lifecycle Management**: Tiered license states — `ACTIVE`, `PRE_EXPIRY_WARNING`, `SOFT_GRACE`, `READ_ONLY`, `OFFLINE_BLOCKED` — with configurable grace periods and offline allowance days.
- **🔧 Super Admin Dashboard**: Dedicated `/admin` route for managing companies, provisioning users, configuring feature flags, setting preview templates, and managing license expiry dates.
- **🚧 Maintenance Mode**: Admin-controlled global read-only lock with a custom message shown to users.

### Database & Export
- **🗃️ Supplier & Customer Database**: Full CRUD for suppliers and customers with soft-delete, category tagging (A+/A/B/C/D for customers; textile product categories for suppliers), and search.
- **📊 Excel Import/Export**: Bulk import suppliers and customers from `.xlsx` files using the `xlsx` library; export database to PDF using `jsPDF` + `jspdf-autotable`.
- **📄 PDF Generation**: Tabular PDF exports of supplier/customer lists.

### Customization
- **🎨 6 Print Preview Templates**: Six pre-built A5 order preview layouts (`OrderPreview1`–`OrderPreview6`) plus a `DynamicOrderPreview` driven by a configurable `TemplateBlock` builder.
- **⚙️ Per-Company Feature Flags**: Toggleable form fields per company — remarks, transport, C/D terms, within-days, send-LR-by, haste, and quantity.
- **✍️ Digital Signature**: Company admins can upload a signature image (stored as base64 in Firestore) that appears on printed order previews.
- **🏷️ Custom App Branding**: Per-company app name, logo URL, and theme color configurable from the admin dashboard.

---

## 🛠️ Tech Stack

| Layer | Technologies |
|---|---|
| **Frontend** | React 19, TypeScript, Vite 7 |
| **Routing** | React Router DOM v7 |
| **Forms** | React Hook Form |
| **Styling** | Tailwind CSS v4, Lucide React icons, `clsx`, `tailwind-merge` |
| **Backend & DB** | Firebase (Firestore, Auth, Hosting) |
| **Offline & PWA** | Vite PWA Plugin, Workbox |
| **AI / ML** | Google Cloud Vision API (OCR), Sarvam AI (Hindi STT), Gemini AI (NLU) |
| **Export** | `html-to-image`, `html2canvas`, `jsPDF`, `jspdf-autotable` |
| **Data Import** | `xlsx` (Excel parsing) |

---

## 🔐 Authentication Flow

1. User visits the app and is redirected to `/login` if unauthenticated.
2. Google Sign-In via Firebase Auth (`signInWithPopup`).
3. On sign-in, the app resolves the user's `companyId` and `role` via:
   - Firebase custom token claims (fastest path), or
   - A `userIndex` Firestore document lookup, or
   - A `collectionGroup` query across all company user lists.
4. Company license, config, and maintenance flags are fetched and cached in `localStorage` with a configurable offline allowance window.
5. `super_admin` users are routed to `/admin`; all others go to the main order form.

---

## 📂 Project Structure

```
src/
├── components/
│   ├── AutocompleteInput.tsx   # Fuzzy search input for suppliers/customers
│   ├── Layout.tsx              # App shell with navigation
│   ├── OrderPreview.tsx        # Preview dispatcher
│   ├── previews/               # OrderPreview1–6 + DynamicOrderPreview
│   └── scanner/
│       ├── ScannerWidget.tsx   # Camera OCR widget (Google Vision API)
│       └── VoiceRecorderWidget.tsx  # Voice input widget (Sarvam + Gemini)
├── contexts/
│   └── AuthCompanyContext.tsx  # Auth, company resolution, license state
├── hooks/
│   ├── useOrderForm.ts         # Order form state, submission, offline queue
│   └── useMasterData.ts        # Supplier/customer data loading & caching
├── lib/
│   ├── company/                # License logic, Firestore paths, runtime state, types
│   ├── scanner/                # Vision API worker, voice worker, text parser
│   ├── firebaseUtils.ts        # Order CRUD helpers
│   ├── supplierUtils.ts        # Supplier CRUD + Excel import
│   └── customerUtils.ts        # Customer CRUD helpers
├── pages/
│   ├── CreateOrder.tsx         # Main order form (multi-item, AI input)
│   ├── OrderHistory.tsx        # Paginated order history with thumbnail sidebar
│   ├── Database.tsx            # Supplier & customer database management
│   ├── Settings.tsx            # Firm config, signature upload, PWA install
│   ├── AdminDashboard.tsx      # Super admin — companies, users, licenses, config
│   └── Login.tsx               # Google OAuth login
├── types.ts                    # Shared TypeScript interfaces (Order, Supplier, Customer, Firm…)
├── firebase.ts                 # Firebase app initialization
└── App.tsx                     # Route definitions + role-based guards
```

---

## 📝 Note

This project is private and confidential. Source code is not publicly available.
