# 🎁 Eid-ul-Adha Salami 2026 — Complete Setup & Deployment Guide

## Overview
Department of International Relations · 22th Batch · University of Chittagong  
Sponsor: **SHADID MAHBUB**

---

## 📁 Project Structure

```
eid-salami-2026/
├── app/
│   ├── layout.tsx              ← Root layout
│   ├── page.tsx                ← Main homepage
│   ├── globals.css             ← Global styles
│   ├── admin/
│   │   ├── page.tsx            ← Admin dashboard
│   │   └── login/page.tsx      ← Admin login
│   └── api/
│       ├── spin/route.ts       ← Main spin endpoint
│       ├── check-spin/route.ts ← Check if already spun
│       └── admin/
│           ├── login/route.ts  ← Admin auth
│           ├── users/route.ts  ← User data + stats
│           └── payment/route.ts ← Mark paid
├── components/
│   ├── SpinWheel.tsx           ← Canvas wheel
│   ├── RegistrationForm.tsx    ← User form
│   ├── ResultCard.tsx          ← Win result display
│   ├── Confetti.tsx            ← Confetti animation
│   └── admin/
│       ├── Dashboard.tsx       ← Full admin panel
│       ├── ParticipantsTable.tsx ← User table
│       └── StatsCards.tsx      ← Stat cards
├── lib/
│   ├── firebase.ts             ← Client Firebase
│   ├── firebase-admin.ts       ← Server Firebase Admin
│   ├── weighted-random.ts      ← Spin algorithm
│   ├── fingerprint.ts          ← Device fingerprint
│   ├── telegram.ts             ← Telegram notifications
│   ├── auth.ts                 ← JWT admin auth
│   └── rate-limit.ts           ← Upstash rate limiting
├── types/index.ts              ← TypeScript types
├── firestore.rules             ← Firestore security
├── .env.local.example          ← Env vars template
└── package.json
```

---

## 🚀 Step-by-Step Deployment Guide

### Step 1: Firebase Setup

1. Go to [Firebase Console](https://console.firebase.google.com)
2. Click **Add Project** → Name: `eid-salami-2026`
3. Disable Google Analytics (optional) → **Create Project**

#### Enable Firestore
1. Left sidebar → **Firestore Database** → **Create database**
2. Choose **Production mode** → Select region (e.g., `asia-south1`)
3. Click **Enable**

#### Get Client Config
1. Project Overview → **Add app** → **Web** (</>) icon
2. Register app → Copy `firebaseConfig` values into `.env.local`

#### Get Admin SDK Key
1. Project Settings (gear icon) → **Service Accounts**
2. Click **Generate new private key** → Download JSON
3. Copy values:
   - `client_email` → `FIREBASE_ADMIN_CLIENT_EMAIL`
   - `private_key` → `FIREBASE_ADMIN_PRIVATE_KEY`

#### Apply Firestore Security Rules
1. Firestore Database → **Rules** tab
2. Replace contents with `firestore.rules` file content
3. Click **Publish**

---

### Step 2: Telegram Bot Setup

1. Open Telegram → search `@BotFather`
2. Send `/newbot` → Follow prompts → Get your bot token
3. Open your bot → Send `/start`
4. Get your Chat ID: Visit `https://api.telegram.org/botYOUR_TOKEN/getUpdates`
5. Look for `"chat":{"id":XXXXXXXXX}` — that's your `TELEGRAM_CHAT_ID`

> **Note:** The token is already pre-configured in the code. Update it in `.env.local` if you create a new bot.

---

### Step 3: Upstash Redis (Rate Limiting)

1. Go to [console.upstash.com](https://console.upstash.com)
2. Create a new Redis database → Choose region
3. Copy **REST URL** and **REST Token** to `.env.local`

> **Optional:** Rate limiting degrades gracefully if not configured. The app still works without it.

---

### Step 4: Admin Password Hash

Generate a bcrypt hash for your admin password:

```bash
node -e "const b=require('bcryptjs'); b.hash('YourPassword123', 12).then(console.log)"
```

Or use an online tool: [bcrypt-generator.com](https://bcrypt-generator.com)

Set in `.env.local`:
```
ADMIN_USERNAME=admin
ADMIN_PASSWORD_HASH=$2a$12$your_hash_here
ADMIN_JWT_SECRET=your_random_64_char_secret
```

Generate JWT secret:
```bash
openssl rand -base64 64
```

---

### Step 5: Local Development

```bash
# Clone / set up project
cd eid-salami-2026

# Install dependencies
npm install

# Copy env template
cp .env.local.example .env.local
# Fill in all values in .env.local

# Run dev server
npm run dev
```

Visit `http://localhost:3000`

---

### Step 6: Vercel Deployment

1. Push code to GitHub repository
2. Go to [vercel.com](https://vercel.com) → **New Project**
3. Import your GitHub repository
4. Configure:
   - Framework: **Next.js** (auto-detected)
   - Root Directory: `./`
5. **Environment Variables** — Add all variables from `.env.local`:
   - `NEXT_PUBLIC_FIREBASE_API_KEY`
   - `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN`
   - `NEXT_PUBLIC_FIREBASE_PROJECT_ID`
   - `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET`
   - `NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID`
   - `NEXT_PUBLIC_FIREBASE_APP_ID`
   - `FIREBASE_ADMIN_PRIVATE_KEY` ← Paste entire key with `\n` preserved
   - `FIREBASE_ADMIN_CLIENT_EMAIL`
   - `TELEGRAM_BOT_TOKEN`
   - `TELEGRAM_CHAT_ID`
   - `ADMIN_USERNAME`
   - `ADMIN_PASSWORD_HASH`
   - `ADMIN_JWT_SECRET`
   - `UPSTASH_REDIS_REST_URL`
   - `UPSTASH_REDIS_REST_TOKEN`

6. Click **Deploy** ✅

> **Important for FIREBASE_ADMIN_PRIVATE_KEY on Vercel:**  
> When pasting the private key, make sure newlines are represented as literal `\n` characters.  
> Or paste the raw key and Vercel handles it automatically.

---

## 🔑 Admin Panel

Access: `https://your-domain.com/admin`  
Login: `https://your-domain.com/admin/login`

Default admin username: `admin` (set via `ADMIN_USERNAME` env var)  
Password: Whatever you hashed into `ADMIN_PASSWORD_HASH`

### Admin Features
- 📊 Live stats dashboard
- 👥 Full participant table with search & filter
- ✅ Mark payment as paid (with bKash Tx ID)
- 📄 Export to CSV / Excel
- 📊 Probability distribution chart (admin-only)

---

## 🎰 Weighted Random Algorithm

```
Number  Weight   Probability
  1       2       2.00%   Very Low
  2       4       4.00%   Low
  3       8       8.00%   Medium
  4      14      14.00%   High
  5      30      30.00%   HIGHEST ← Peak
  6      20      20.00%   High
  7      10      10.00%   Medium
  8       6       6.00%   Low
  9       4       4.00%   Very Low
 10       2       2.00%   JACKPOT
─────────────────────────────
Total: 100 weight = 100%
```

- Generated **server-side only** — never trusted from client
- Client cannot manipulate result
- Result is saved to Firestore before being returned

---

## 🔒 Security Features

| Feature | Implementation |
|---------|----------------|
| One spin per user | IP + Device Fingerprint + bKash dedup |
| Server-side result | `weightedRandomSpin()` in API route |
| Rate limiting | Upstash Redis (3 req/hour/IP) |
| Admin auth | JWT (24h expiry) + bcrypt password |
| Firestore rules | All client writes blocked |
| Input sanitization | Strip HTML, trim, validate regex |
| HTTPS | Vercel enforces HTTPS |

---

## 📱 Telegram Notification Format

```
🎁 New Salami Spin!

Eid-ul-Adha Salami 2026

👤 Name : John Doe
📅 Session : 2025-26
🗺️ District : Dhaka
📱 Bkash : 01XXXXXXXXX
💰 Result : 5 BDT
🌐 IP : xxx.xxx.xxx.xxx

✅ Payment pending
🕐 June 9, 2026, 10:30:00 AM
```

For jackpot (10 BDT):
```
🏆🎊 JACKPOT HIT! 🎊🏆

🎁 New Salami Spin!
...
```

---

## 🐛 Troubleshooting

**Firebase Admin key error:**
```
Error: Failed to parse private key
```
→ Make sure `\n` in the private key are actual newlines. In Vercel, paste the key with `\n` as literal backslash-n.

**Telegram not sending:**
→ Start a conversation with your bot first, then get the chat ID.

**Rate limit not working:**
→ Upstash credentials may be missing. App still functions without rate limiting.

**Admin login fails:**
→ Verify `ADMIN_USERNAME` and `ADMIN_PASSWORD_HASH` are set correctly in Vercel env vars.

---

## 🌙 Eid Mubarak!

*Sponsored by **SHADID MAHBUB***  
*Department of International Relations · University of Chittagong · 22th Batch · 2025-26*
