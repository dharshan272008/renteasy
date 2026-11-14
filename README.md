# RentEase - Compound Rental & Electricity Management System

RentEase is a rental management system designed for compound owners and landlords in India (managing multiple portions, independent houses, or compound flats). It features real Supabase Authentication, row-level security (RLS), AI-powered electricity meter reading via Google Gemini Vision, rent payment tracking with receipt evidence vault, and automated electricity billing.

---

## 🛠️ Stack Architecture

- **Frontend**: React 19, TypeScript, Vite, Tailwind CSS 4, Lucide Icons.
- **Backend**: Express on Node.js (`server.ts`) mounting Vite middleware in dev.
- **Database & Auth**: Supabase PostgreSQL with strict Row-Level Security (`owner_id = auth.uid()::text`).
- **Storage**: Supabase Storage private bucket `rentease-evidence` (JPEG, PNG, WebP up to 5MB) accessed via 1-hour signed URLs.
- **AI OCR**: Google Gemini Vision (`gemini-3.8-flash`) reading active numerical energy values (including decimal kWh like `20750.7`).

---

## 🚀 Environment Variables

Copy `.env.example` to `.env`:

```env
# Client & Server Supabase
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-supabase-anon-key

# Server-Side Proxies & AI OCR
PORT=3000
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-supabase-anon-key
GEMINI_API_KEY=your-gemini-api-key
```

> **Note**: In production or AI Studio, client variables read from `import.meta.env` while `server.ts` uses `process.env`. Never commit secrets.

---

## 🗄️ Database Setup (Supabase)

1. Navigate to your Supabase project's **SQL Editor**.
2. Open `supabase/schema.sql` from this repository.
3. Paste and run the entire script.

The SQL migration is **safe and idempotent**:
- Creates and validates all 11 core tables:
  `owners`, `houses`, `tenants`, `tenancy_history`, `meters`, `meter_readings`, `meter_evidence`, `rent_payments`, `rent_evidence`, `eb_payments`, `audit_logs`.
- Enables Row Level Security on all tables with strict owner-folder isolation.
- Provisions the private `rentease-evidence` storage bucket with allowed mime types (`image/jpeg`, `image/png`, `image/webp`).
- Adds database triggers ensuring meter readings and rent payments match active tenancies without orphaned records.
- Snapshots electricity `unit_rate` and `eb_amount` at the moment of reading.

---

## 🔐 Supabase Authentication Settings

In your Supabase project dashboard (**Authentication → Providers → Email**):
- **Enable Email Signup**: ON
- **Confirm email**: Works with both ON or OFF. RentEase automatically informs users if a verification email has been dispatched or logs them in immediately if auto-confirm is enabled.

---

## 🌐 Deploy to Render and use on Android

RentEase is a web app that can be installed from Android Chrome's menu after deployment. It still needs an internet connection to use Supabase and the server APIs.

1. Push the project to a GitHub repository. The local `.env` file is git-ignored; never commit API keys.
2. In Render, create a **Blueprint** from that repository and use the included `render.yaml`.
3. Set the environment variables requested by Render. Copy the Supabase project URL to both URL variables, and the Supabase publishable key to both key variables. Set `GEMINI_API_KEY` to your Gemini key. Do not use a Supabase secret/service-role key.
4. Deploy and wait for the `/api/health` check to pass. In Supabase **Authentication → URL Configuration**, set the Site URL to the Render HTTPS URL and add it to the allowed redirect URLs.
5. On Android, open the Render URL in Chrome, then choose **⋮ → Install app** (or **Add to Home screen**).

Render's free service may sleep when idle, so the first request after a period of inactivity can take longer.

---

## 📦 Scripts

- `npm run dev`: Starts the Node.js Express server + Vite dev middleware on port 3000.
- `npm run build`: Compiles TypeScript and builds the production frontend into `/dist`.
- `npm run start`: Starts the production server and serves the built frontend.
- `npm run lint`: Runs TypeScript validation (`tsc --noEmit`).
- `npm run clean`: Cleans the build output directory (`rm -rf dist`).
