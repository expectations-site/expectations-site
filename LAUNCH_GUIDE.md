# EXPECTATIONS — Complete Launch Guide
## Everything you need to go from files to a live business

---

# PART 1: GET THE WEBSITE LIVE (Technical)

## Step 1 — Domain Name
Buy **expectations.co** (or .com) from:
- **Namecheap** → namecheap.com (~$12/year for .co)
- **Cloudflare** → cloudflare.com (cheapest, no markup)

Recommended: `expectations.co` matches the brand

---

## Step 2 — Hosting (Free to start)

### Frontend (HTML files) → Vercel
1. Go to vercel.com → Sign up free
2. Install: `npm install -g vercel`
3. In your project folder: `vercel --prod`
4. Connect your domain in Vercel dashboard → Settings → Domains
5. Add: `expectations.co` and `www.expectations.co`

### Backend (API) → Railway
1. Go to railway.app → Sign up free ($5/month after free tier)
2. `npm install -g @railway/cli`
3. `cd backend && railway init && railway up`
4. Add env variables in Railway dashboard
5. Your API lives at: `https://expectations-api.railway.app`

---

## Step 3 — Stripe (Payments)
1. Go to **stripe.com** → Create account
2. Add your business details + bank account
3. Dashboard → Developers → API Keys
4. Copy Secret Key → paste in Railway env vars
5. Copy Publishable Key → paste in checkout.html
6. Enable: Cards, Apple Pay, Google Pay (Settings → Payment Methods)
7. Set up webhook: Dashboard → Webhooks → Add endpoint
   → URL: `https://your-railway-url.railway.app/api/webhook/stripe`
   → Events: `payment_intent.succeeded`, `payment_intent.payment_failed`

**Stripe fees:** 2.9% + 30¢ per transaction. No monthly fee.

---

## Step 4 — Email (Resend)
1. Go to **resend.com** → Sign up free (3,000 emails/month free)
2. Add your domain: expectations.co
3. Add the DNS records Resend gives you in Namecheap/Cloudflare
4. Create API key → paste in Railway env vars
5. Test: send yourself a test order confirmation

---

## Step 5 — Update checkout.html with your live keys
Find this section in checkout.html and add:
```html
<!-- Add just before </body> -->
<script src="https://js.stripe.com/v3/"></script>
```
Replace `processPayment()` demo logic with the Stripe.js code
from the README — takes about 20 minutes.

---

## Step 6 — Environment Variables (Railway)
Add these in Railway → Variables:
```
STRIPE_SECRET_KEY        = sk_live_...
STRIPE_PUBLISHABLE_KEY   = pk_live_...
STRIPE_WEBHOOK_SECRET    = whsec_...
RESEND_API_KEY           = re_...
MANUFACTURER_EMAIL       = your-manufacturer@email.com
MANUFACTURER_KEY         = choose-a-long-random-string
FRONTEND_URL             = https://expectations.co
NODE_ENV                 = production
```

---

## Step 7 — Go Live Checklist
- [ ] Domain purchased and connected to Vercel
- [ ] Backend deployed on Railway
- [ ] Stripe in LIVE mode (not test)
- [ ] Test order placed end-to-end
- [ ] Manufacturer received the email notification
- [ ] Confirmation email landed in inbox (not spam)
- [ ] SSL certificate active (Vercel does this automatically)
- [ ] Manufacturer portal login working

---

# PART 2: BUSINESS ESSENTIALS

## Legal & Business Registration

### Register your business
- Register as an LLC or Sole Proprietor in your country
- Get a business bank account (Wise Business is great for international)
- Register a business email: hello@expectations.co (Google Workspace ~$6/month)

### Legal pages you need on your website
These are legally required in most countries:
- **Privacy Policy** — covers data collection (use Termly.io to generate free)
- **Terms & Conditions** — covers purchases and disputes
- **Refund/Return Policy** — be specific: 14 or 30 days? Exchange only?
- **Shipping Policy** — processing time, estimated delivery, international fees
- **Cookie Policy** — required for EU customers (GDPR)

Add these as links in your footer. I can write all of them for you.

---

## Tax Setup

### Depending on your country:
- Register for VAT/GST if selling to EU, UK, or Australian customers
- In the US: collect sales tax per state (Stripe Tax handles this automatically — enable it in Stripe dashboard for ~0.5% fee)
- Keep records of every sale — Stripe exports these automatically

### Recommended: Use **Stripe Tax**
- Stripe dashboard → Tax → Enable
- Automatically calculates and collects correct tax per country
- Generates tax reports for your accountant

---

## Pricing & Margins

### Know your numbers before launch:
```
Sale price:          $89  (hoodie example)
Manufacturing cost:  $28  (estimate)
Shipping to customer: $8  (built into free shipping threshold)
Stripe fee:          $2.88 (2.9% + 30¢)
Packaging:           $2
──────────────────────────
Your margin:         ~$48  (54%)
```

Aim for 50–70% gross margin on streetwear.
Price aggressively enough to run sales without going below 30%.

---

## Packaging & Branding

You need physical branded packaging:
- **Poly mailer bags** — black matte with EXPECTATIONS printed
- **Tissue paper** — branded with logo stamp or printed
- **Sticker / thank you card** — "Wear it with EXPECTATIONS" note
- **Hang tags** — attach to every garment with size + care instructions

**Where to order:**
- noissue.co — sustainable branded packaging, minimum 100 units
- packlane.com — custom boxes
- stickermule.com — stickers and labels
- alibaba.com — bulk hang tags and poly mailers cheapest

---

# PART 3: MARKETING TOOLS TO ADD

## 1. Instagram Shopping
- Set up Instagram Business account
- Connect to a Facebook Catalog (Meta Commerce Manager)
- Tag products in posts → customers buy without leaving Instagram
- This alone can drive 30–40% of streetwear sales

## 2. TikTok Shop
- Apply at seller.tiktok.com
- Upload your product catalog
- TikTok videos with products linked = free traffic
- TikTok affiliates can promote your items for commission

## 3. Google Analytics 4
Add this to every HTML page (free):
```html
<!-- Replace G-XXXXXXXX with your Measurement ID -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXX');
</script>
```
Shows you: where visitors come from, what they click, where they drop off.

## 4. Meta Pixel (Facebook/Instagram Ads)
Add to every page for retargeting ads:
```html
<script>
  !function(f,b,e,v,n,t,s){...}  <!-- Meta provides the full snippet -->
  fbq('init', 'YOUR_PIXEL_ID');
  fbq('track', 'PageView');
</script>
```
This lets you run retargeting ads to people who visited but didn't buy.

## 5. Email Marketing — Klaviyo
- klaviyo.com — free up to 250 contacts
- Connect to your backend to sync orders
- Set up automated flows:
  - Welcome email when someone subscribes
  - Abandoned cart email (1 hour after they leave)
  - Post-purchase email (3 days after delivery — ask for a review)
  - Win-back email (30 days inactive)

These automations alone can recover 15–20% of lost sales.

## 6. SMS Marketing — Postscript or Klaviyo SMS
- Streetwear audience responds very well to SMS
- "New drop just landed 👀 — expectations.co" 
- 40–50% open rates vs 20% for email

---

# PART 4: CUSTOMER EXPERIENCE ADD-ONS

## Size Guide Page
Add a size guide page (I can build this) with:
- Measurement instructions (how to measure chest, waist, length)
- Size chart per product category
- "Our models are X cm tall wearing size M" note

## Reviews — Judge.me or Okendo
- Collect reviews after purchase (auto-email 7 days post-delivery)
- Show star ratings on product cards
- Judge.me free plan works well for starting out

## Live Chat — Tidio or Crisp
- Tidio.com — free plan available
- Add a chat widget to your site
- Customers ask sizing questions before buying = fewer returns
- Connect to your phone so you can reply on the go

## Instagram Feed — Elfsight
- Embed your Instagram grid on the homepage
- Shows social proof without building a separate gallery
- elfsight.com ~$6/month

## Loyalty Program — Smile.io
- Reward customers with points for purchases, shares, referrals
- Free tier available
- Great for repeat customers in streetwear

---

# PART 5: OPERATIONS

## Manufacturer Relationship
Your manufacturer portal is built — now set clear agreements:
- **SLA:** Orders must ship within 2 business days of notification
- **Quality control:** Define acceptable/unacceptable defect rate
- **Low stock alerts:** Manufacturer reorders when stock hits threshold
- **Returns handling:** Who processes returns — you or manufacturer?

Document this in a simple contract.

## Returns Process
Decide your policy:
- Free returns or customer pays return shipping?
- Exchange only, or full refund?
- How many days?

Build a returns page on the site (I can add this).

## Customer Service Email
Set up: support@expectations.co
Use **Missive** or **Zendesk** to manage support emails professionally.

Respond to every inquiry within 24 hours — reputation is everything in streetwear.

---

# PART 6: MONTHLY COSTS (Realistic Estimate)

| Service | Cost/month |
|---|---|
| Vercel (hosting) | Free |
| Railway (backend) | $5 |
| Domain renewal | ~$1 |
| Resend (email) | Free (3k/mo) |
| Stripe fees | 2.9% per sale |
| Google Workspace (email) | $6 |
| Klaviyo (email marketing) | Free to start |
| **Total fixed** | **~$12/month** |

You can run the entire operation for under $15/month to start.
Scale up services as revenue grows.

---

# QUICK LAUNCH PRIORITY ORDER

## Week 1 — Get live
1. Buy domain (Namecheap)
2. Deploy frontend to Vercel
3. Deploy backend to Railway
4. Set up Stripe live mode
5. Set up Resend email
6. Test one full order end-to-end

## Week 2 — Legal & social
7. Write Privacy Policy + Returns Policy (I can do this)
8. Set up Instagram Business + TikTok accounts
9. Add Google Analytics to all pages
10. Set up Klaviyo welcome flow

## Week 3 — Marketing
11. Set up Instagram Shopping catalog
12. Add Meta Pixel
13. Film first TikTok content
14. Send launch email to your list

## Week 4 — Refine
15. Add live chat (Tidio)
16. Add reviews widget
17. Monitor first orders end-to-end
18. Adjust based on what you see in Analytics

---

# WHAT I CAN STILL BUILD FOR YOU

- [ ] Size guide page
- [ ] Returns / Refund policy page
- [ ] Privacy Policy + Terms pages
- [ ] Product detail page (individual item view)
- [ ] Search functionality
- [ ] Loyalty points system
- [ ] Admin analytics dashboard (revenue charts)
- [ ] Multi-language support
- [ ] Mobile app (React Native)

---

Built for EXPECTATIONS — SS26 🖤
