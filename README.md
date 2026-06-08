# EXPECTATIONS — Full Stack Website

A complete e-commerce platform for the EXPECTATIONS streetwear brand.

---

## Project Structure

```
expectations-website/
├── index.html          # Storefront (Sprint 1)
├── tryon.html          # AI Virtual Try-On (Sprint 3)
├── checkout.html       # Cart + Checkout (Sprint 4)
├── manufacturer.html   # Manufacturer Portal (Sprint 2)
└── backend/
    ├── server.js       # Express API
    ├── package.json
    ├── .env.example    # Environment variables template
    └── expectations.db # SQLite database (auto-created)
```

---

## Quick Start (Local)

### 1. Backend

```bash
cd backend
npm install
cp .env.example .env
# Fill in your keys in .env (see below)
npm run dev
```

The API runs on `http://localhost:3001`

### 2. Frontend

Serve the HTML files from the root folder:

```bash
# Option A: VS Code Live Server (easiest)
# Right-click index.html → Open with Live Server

# Option B: Python (no install needed)
cd expectations-website
python3 -m http.server 3000

# Option C: Node static server
npx serve . -p 3000
```

Open `http://localhost:3000`

---

## Setting Up Keys

### Stripe (Payments)

1. Go to https://dashboard.stripe.com/apikeys
2. Copy your **Secret key** (`sk_test_...`) → `STRIPE_SECRET_KEY`
3. Copy your **Publishable key** (`pk_test_...`) → `STRIPE_PUBLISHABLE_KEY`
4. Add your publishable key to `checkout.html`:

```html
<!-- In checkout.html, add before </body> -->
<script src="https://js.stripe.com/v3/"></script>
<script>
  const stripe = Stripe('pk_test_YOUR_PUBLISHABLE_KEY');
  // Stripe Elements mount here
</script>
```

5. For webhooks (local testing):
```bash
npm install -g stripe
stripe listen --forward-to localhost:3001/api/webhook/stripe
# Copy the webhook secret → STRIPE_WEBHOOK_SECRET
```

### Resend (Email)

1. Sign up free at https://resend.com (3,000 emails/month free)
2. Add and verify your domain (`expectations.co`)
3. Create an API key → `RESEND_API_KEY`
4. Set `MANUFACTURER_EMAIL` to your manufacturer's email address

---

## API Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET    | /api/health | — | Health check |
| POST   | /api/create-payment-intent | — | Create Stripe payment intent |
| POST   | /api/confirm-order | — | Save order + send emails |
| GET    | /api/orders | Mfg key | List all orders |
| GET    | /api/orders/:id | Mfg key | Single order + events |
| PATCH  | /api/orders/:id/status | Mfg key | Update status / tracking |
| GET    | /api/inventory | Mfg key | Stock levels |
| PATCH  | /api/inventory | Mfg key | Update stock |
| GET    | /api/analytics | Mfg key | Order analytics |
| POST   | /api/webhook/stripe | Stripe sig | Payment webhooks |

**Manufacturer auth:** Add header `x-manufacturer-key: YOUR_KEY` to all `/api/orders` and `/api/inventory` requests.

---

## Connecting Frontend to Backend

In `checkout.html`, replace the demo `processPayment()` with:

```javascript
async function processPayment() {
  // 1. Create payment intent
  const intentRes = await fetch('http://localhost:3001/api/create-payment-intent', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ amount: grandTotal, email: customerEmail })
  });
  const { clientSecret } = await intentRes.json();

  // 2. Confirm with Stripe.js
  const { error, paymentIntent } = await stripe.confirmCardPayment(clientSecret, {
    payment_method: {
      card: cardElement, // Stripe Elements card
      billing_details: { name: cardName, email: customerEmail }
    }
  });

  if (error) { toast(error.message, 'err'); return; }

  // 3. Confirm order in your backend
  const orderRes = await fetch('http://localhost:3001/api/confirm-order', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      firstName, lastName, email, phone,
      addr1, addr2, city, state, zip, country,
      items: cart, subtotal, shippingCost, discount, total,
      shippingMethod: shipping.method,
      stripePaymentIntentId: paymentIntent.id
    })
  });
  const { orderId } = await orderRes.json();
  showConfirmation({ orderId, ... });
}
```

---

## Connecting the Manufacturer Dashboard

In `manufacturer.html`, update the API calls to use your backend:

```javascript
// Replace mock data fetch with:
const res = await fetch('http://localhost:3001/api/orders', {
  headers: { 'x-manufacturer-key': 'YOUR_MANUFACTURER_KEY' }
});
const { orders } = await res.json();
renderOrders(orders);

// To ship an order:
await fetch(`http://localhost:3001/api/orders/${orderId}/status`, {
  method: 'PATCH',
  headers: {
    'Content-Type': 'application/json',
    'x-manufacturer-key': 'YOUR_MANUFACTURER_KEY'
  },
  body: JSON.stringify({ status: 'shipped', trackingNumber: 'DHL-123', carrier: 'DHL' })
});
```

---

## Production Deployment

### Option A: Vercel (Recommended — Free)

```bash
# Install Vercel CLI
npm install -g vercel

# Deploy frontend (from root)
vercel --prod

# Deploy backend (from backend/)
cd backend
vercel --prod
```

Add environment variables in Vercel dashboard → Settings → Environment Variables.

### Option B: Railway (Backend) + Vercel (Frontend)

Railway is great for Node backends with persistent SQLite or PostgreSQL.

```bash
# Install Railway CLI
npm install -g @railway/cli
railway login
cd backend
railway init
railway up
```

### Option C: VPS (DigitalOcean / Hetzner)

```bash
# On your server:
git clone YOUR_REPO
cd expectations-website/backend
npm install --production
cp .env.example .env && nano .env  # fill in keys
npm install -g pm2
pm2 start server.js --name expectations-api
pm2 save && pm2 startup

# Nginx reverse proxy
# /etc/nginx/sites-available/expectations
server {
    listen 80;
    server_name api.expectations.co;
    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}
```

---

## Upgrading to PostgreSQL (Production)

For a live store, replace SQLite with PostgreSQL:

```bash
npm install pg
```

```javascript
// Replace better-sqlite3 with:
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// Async queries:
const { rows } = await pool.query('SELECT * FROM orders WHERE id = $1', [id]);
```

Free PostgreSQL options: Supabase, Neon, Railway.

---

## Order Flow

```
Customer → Checkout → Stripe → /api/confirm-order
                                    ↓
                            Save to database
                                    ↓
                    ┌───────────────┴───────────────┐
                    ↓                               ↓
          Email customer                Email manufacturer
          (order confirmation)          (ship now notification)
                                                    ↓
                                        Manufacturer Portal
                                        (mark shipped + tracking)
                                                    ↓
                                        Email customer
                                        (tracking number)
```

---

## Promo Codes

To add promo codes, edit the `applyPromo()` function in `checkout.html` and optionally create a `promo_codes` table in the database:

```sql
CREATE TABLE promo_codes (
  code TEXT PRIMARY KEY,
  discount_pct INTEGER,
  active INTEGER DEFAULT 1,
  uses_remaining INTEGER DEFAULT -1
);
INSERT INTO promo_codes VALUES ('EXP10', 10, 1, -1);
INSERT INTO promo_codes VALUES ('LAUNCH20', 20, 1, 100);
```

---

Built with ❤ for EXPECTATIONS — SS26
