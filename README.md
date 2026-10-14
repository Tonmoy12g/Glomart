# 🌸 GlowMart — E-commerce Website

**GlowMart** হলো বাংলাদেশের একটি স্কিনকেয়ার অনলাইন শপের সম্পূর্ণ ওয়েবসাইট —
মোবাইল-ফ্রেন্ডলি বাংলা স্টোরফ্রন্ট + Flask ব্যাকএন্ড (প্রোডাক্ট API, অর্ডার, অ্যাকাউন্ট, অ্যাডমিন)।

## ⚡ দ্রুত শুরু (Quick Start)

```bash
cd glowmart-website/backend
pip install -r requirements.txt
python app.py
```

তারপর ব্রাউজারে খুলুন: **http://localhost:5000**

প্রথমে অ্যাকাউন্ট খুলুন (নাম + মোবাইল + পাসওয়ার্ড) অথবা লগইন করুন —
লগইন ছাড়া সাইটে ঢোকা যাবে না।

অ্যাডমিন পেজ: **http://localhost:5000/admin**
(ডিফল্ট পাসওয়ার্ড: `changeme123` — নিচে দেখুন কীভাবে বদলাবেন)

---

## English

### What this is

- **Frontend** (`index.html`, `styles.css`, `app.js`, `images/`): Bangla mobile-first
  storefront. Homepage, product catalog with search + category filter, product
  detail modal, cart drawer, checkout form (COD), order confirmation.
- **Backend** (`backend/app.py`, Flask): serves the frontend AND a JSON API.
  One command (`python app.py`) runs the whole site locally.
- **Database**: SQLite (`backend/orders.db`, stdlib `sqlite3` only).
  - `users` — customer accounts. Passwords are **never stored in plaintext**:
    salted PBKDF2 hashes via `werkzeug.security.generate_password_hash`.
  - `sessions` — random login tokens (`secrets.token_urlsafe`), deleted on logout.
  - `orders` — orders linked to `user_id`, status `pending`, IDs like `GM-XXXXXX`.
- **Pathao Courier**: `backend/pathao.py` contains stub functions only.
  **Integration is PREPARED but NOT active and NOT tested.** Nothing in the UI
  or docs claims otherwise. Credentials (when you get them) go in environment
  variables on the server — never in frontend code.

### API endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | `/api/products` | — | Product catalog from `products.json` |
| POST | `/api/auth/signup` | — | `{name, phone, password}` → session token |
| POST | `/api/auth/login` | — | `{phone, password}` → session token |
| GET | `/api/auth/me` | Bearer token | `{name, phone}` of logged-in user |
| POST | `/api/auth/logout` | Bearer token | Deletes the session token |
| POST | `/api/orders` | Bearer token | `{address, notes, items:[{id,qty}], payment:"cod"}` → `{order_id}` |
| GET | `/api/admin/orders` | `?password=` | Orders, newest first |
| POST | `/api/admin/pathao/test` | `?password=` | Returns the "not active" message |
| GET | `/admin` | password in page | Simple admin order viewer |

Phone numbers are validated as Bangladeshi mobile format (`01XXXXXXXXX`).
Order totals are computed server-side from `products.json` — the client total
is never trusted.

### Adding / editing products

1. Edit `products.json` — each product needs `id`, `name`, `size`, `category`,
   `price` (number, in Taka), `image` (path under `images/`), `description`
   and `usage` (Bangla text).
2. Drop the product photo into `images/` (JPG/PNG).
3. No restart needed — `/api/products` reads the file on every request.

### Admin password

Set a strong password with the `ADMIN_PASSWORD` environment variable:

```bash
# local
ADMIN_PASSWORD="আপনার-শক্ত-পাসওয়ার্ড" python app.py
```

On hosting, set it in the dashboard's environment variables. The default
`changeme123` is for local testing only.

### Publishing with free hosting (beginner-friendly)

**Option A — frontend only (quickest, free):**
1. Take these files/folders: `index.html`, `styles.css`, `app.js`, `images/`.
2. Drag-and-drop them onto **Netlify Drop** (app.netlify.com/drop),
   or deploy to **Vercel / GitHub Pages / Cloudflare Pages**.
3. You get a public link instantly.
4. ⚠️ Without the backend, login and checkout **cannot save anything** —
   this is only a visual demo.

**Option B — full site (recommended for real orders):**
1. Put the frontend files on **Netlify** or **Vercel** (free).
2. Deploy `backend/` on **Render** (free tier):
   - New → Web Service → connect your GitHub repo → root directory `backend`.
   - Build command: `pip install -r requirements.txt`
   - Start command: `gunicorn app:app`
   - Add environment variables in the Render dashboard:
     `ADMIN_PASSWORD` = your strong password.
3. Point the frontend's API calls at your Render URL
   (in `app.js`, set the fetch base URL — or serve everything from Flask
   on Render alone and skip the separate frontend host).
4. ⚠️ Honest caveats of free tiers:
   - Render free sleeps after ~15 minutes idle (first visit is slow).
   - Render's filesystem is **ephemeral** — `orders.db` (SQLite) can be lost
     on restart. Fine for testing; use a persistent database
     (e.g. Render Postgres, paid) before taking real customer orders.
5. **Pathao is NOT active.** When you integrate it for real, put
   `PATHAO_CLIENT_ID` / `PATHAO_CLIENT_SECRET` / `PATHAO_BASE_URL` in the
   hosting dashboard's environment variables — never in code — and test
   via `POST /api/admin/pathao/test` before wiring it into checkout.

### Project layout

```
glowmart-website/
├── index.html          # storefront (Bangla UI)
├── styles.css          # pink/white/soft-green theme, mobile-first
├── app.js              # shop + auth gate logic (same-origin /api/* only)
├── products.json       # editable product catalog
├── images/             # product photos
├── .env.example        # env var template (no real secrets)
├── README.md
└── backend/
    ├── app.py          # Flask: static hosting + API + auth + admin
    ├── admin.html      # password-protected order viewer
    ├── pathao.py       # Pathao stubs — NOT active/tested
    └── requirements.txt
```
