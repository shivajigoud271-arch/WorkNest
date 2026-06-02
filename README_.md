# WorkNest — Setup Guide

## What's in this folder

| File | Purpose |
|------|---------|
| `index.html` | The full website (all pages + Stripe + EmailJS) |
| `vercel.json` | Vercel deployment config |
| `README.md` | This setup guide |

---

## Step 1 — Push to GitHub (2 mins)

1. Go to https://github.com/new
2. Create a repo called `worknest` (private or public)
3. Open your terminal and run:

```bash
cd worknest
git init
git add .
git commit -m "WorkNest launch"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/worknest.git
git push -u origin main
```

---

## Step 2 — Deploy on Vercel (2 mins, free)

1. Go to https://vercel.com → Sign up with GitHub
2. Click **"Add New Project"** → Import your `worknest` repo
3. Click **Deploy** — done! You get a free URL like `worknest.vercel.app`

---

## Step 3 — Set up EmailJS (free, 200 emails/month)

EmailJS sends emails directly from the browser — no backend server needed.

1. Go to https://www.emailjs.com → Create free account
2. **Add Email Service:**
   - Click "Email Services" → Add Service → Gmail (use your Gmail)
   - Copy your **Service ID** (looks like `service_abc123`)

3. **Create Template 1 — Admin notification** (you receive this when someone applies)
   - Click "Email Templates" → Create New
   - Template ID: `template_admin_notify`
   - Subject: `New WorkNest Application — {{job_title}}`
   - Body:
     ```
     New application received!

     Job: {{job_title}} — {{job_area}}

     Student: {{student_name}}
     Email: {{student_email}}
     Age: {{student_age}}
     Phone: {{student_phone}}
     Visa expiry: {{visa_expiry}}
     Availability: {{availability}}
     Languages: {{languages}}
     Experience: {{experience}}

     Reply to this email to contact the student.
     ```
   - To email: your Gmail address
   - Save → Copy Template ID

4. **Create Template 2 — Student confirmation** (student receives this after applying)
   - Template ID: `template_student_confirm`
   - Subject: `Your WorkNest application is confirmed! ✅`
   - Body:
     ```
     Hi {{student_name}},

     Your application for {{job_title}} near {{job_area}} has been received!

     What happens next:
     - We will contact the employer with your profile within 24 hours
     - You will receive the exact location, schedule, and trial day details within 24–48 hours
     - If you don't hear back within 30 days, reply to this email for a full €1 refund

     Thank you for using WorkNest!

     — WorkNest Team
     ```
   - To email: `{{to_email}}`
   - Save → Copy Template ID

5. **Get your Public Key:**
   - Click your account name (top right) → "Account"
   - Copy **Public Key**

6. **Update CONFIG in index.html:**
   ```js
   const CONFIG = {
     EMAILJS_PUBLIC_KEY: 'your_public_key_here',
     EMAILJS_SERVICE_ID: 'service_abc123',
     EMAILJS_TEMPLATE_ADMIN: 'template_admin_notify',
     EMAILJS_TEMPLATE_STUDENT: 'template_student_confirm',
     ADMIN_EMAIL: 'your@gmail.com',
     STRIPE_PK: 'pk_test_...', // add Stripe key below
   };
   ```

---

## Step 4 — Set up Stripe (€1 payments)

Stripe takes a small fee (~3.5% + €0.25 per transaction on €1 = about €0.28 fee, you receive ~€0.72).

### Simple way (Stripe Payment Links — no code needed):
1. Go to https://dashboard.stripe.com → Create account
2. Click **"Payment Links"** → Create link
3. Add product: "WorkNest Application Fee" — €1.00
4. Copy the link (e.g. `https://buy.stripe.com/abc123`)
5. In `index.html`, replace the `handlePayment()` function's Stripe block with:
   ```js
   window.open('https://buy.stripe.com/YOUR_LINK', '_blank');
   await sendEmails(...);
   showSuccess();
   ```

### Advanced way (Stripe.js — already in code):
1. Dashboard → Developers → API Keys
2. Copy **Publishable Key** (starts with `pk_test_` for testing)
3. Add to CONFIG: `STRIPE_PK: 'pk_test_your_key_here'`
4. For production, you need a small backend (Vercel serverless function) to create PaymentIntents — see `/api/create-payment-intent.js` template below

---

## Optional — Vercel Serverless Function for Stripe (production)

Create file `/api/create-payment-intent.js`:

```js
const Stripe = require('stripe');

module.exports = async (req, res) => {
  const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);
  const intent = await stripe.paymentIntents.create({
    amount: 100, // €1.00 in cents
    currency: 'eur',
    automatic_payment_methods: { enabled: true },
  });
  res.json({ clientSecret: intent.client_secret });
};
```

Add your secret key in Vercel dashboard → Settings → Environment Variables:
- `STRIPE_SECRET_KEY` = `sk_live_your_secret_key`

---

## Deployment checklist

- [ ] GitHub repo created and code pushed
- [ ] Vercel deployed (get your live URL)
- [ ] EmailJS account created, two templates made
- [ ] CONFIG in index.html updated with all keys
- [ ] Stripe payment link or Stripe.js configured
- [ ] Test a full application end-to-end
- [ ] Share WorkNest URL with your student community!

---

## Promote WorkNest

Once live, share in:
- WhatsApp groups for Indian students in Paris/France
- Facebook: "Étudiants Indiens en France", "Indian Students in Europe"
- Reddit: r/Paris, r/france, r/expats
- Instagram: tag @indiansinparis type accounts
- EDC Paris student networks

---

*Built with ❤️ for Indian students in Europe*
