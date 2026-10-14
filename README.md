# FreshFold Laundry – Laundry Services Web App

Tutedude Assignment 4. A responsive React + Vite website where visitors can browse laundry services, build a cart, book a pickup, and have the booking emailed through **EmailJS**.

## What is frontend-only vs. external

| Feature | How it works |
| --- | --- |
| Services, cart, totals, validation, booking reference | 100% frontend (React). Cart is saved in `localStorage`. |
| Booking email | **EmailJS** (external service, called from the browser). Needs your free EmailJS account. |
| Contact form | Optional second EmailJS template. Without it, the form runs in a clearly-labelled **demo mode** (nothing is sent). |
| Backend server | **Not needed.** There is no database; bookings exist only in the email. |

## Tech stack
React 18, Vite 5, plain CSS, `@emailjs/browser`.

## Folder structure
```
freshfold-laundry/
├── index.html                 # page title, meta description, fonts
├── package.json
├── vite.config.js
├── .env.example               # copy to .env and fill in
├── .gitignore
├── README.md
└── src/
    ├── main.jsx               # entry point
    ├── App.jsx                # page layout + shared handlers
    ├── components/            # Header, Hero, Stats, Services, ServiceCard, Cart,
    │                          # BookingForm, Field, About, Contact, Footer, BackToTop, Toast
    ├── config/emailConfig.js  # EmailJS configuration (reads .env)
    ├── services/emailService.js  # sends emails via EmailJS
    ├── hooks/useCart.js, useToast.js
    ├── data/services.js, siteContent.js   # edit prices & text here
    ├── utils/format.js, validation.js, booking.js
    └── styles/main.css
```

## 1. Install and run locally
Requirements: [Node.js](https://nodejs.org) 18 or newer.

```bash
cd freshfold-laundry
npm install
cp .env.example .env      # Windows: copy .env.example .env
npm run dev
```
Open the address shown (usually http://localhost:5173). Without EmailJS values the site still runs; booking shows a clear "email not configured" message.

## 2. Configure EmailJS (step by step)
1. Create a free account at https://www.emailjs.com and sign in.
2. **Email Services → Add New Service.** Pick Gmail (or another provider), connect it, and copy the **Service ID**.
3. **Email Templates → Create New Template.** Set:
   - **To Email:** your business/test email address (where bookings should arrive).
   - **Reply To:** `{{reply_to}}`
   - **Subject:** `New laundry booking {{booking_ref}} from {{customer_name}}`
   - **Content:** paste the body below (in the template's code editor, wrap the order details in a `<pre style="white-space:pre-wrap">` so line breaks show).
   ```
   Booking reference: {{booking_ref}}

   Customer: {{customer_name}}
   Email: {{customer_email}}
   Phone: {{customer_phone}}

   Pickup address: {{pickup_address}}
   Pickup date: {{pickup_date}}
   Pickup time: {{pickup_time}}
   Instructions: {{instructions}}

   Order details:
   {{order_details}}

   Total: {{order_total}}
   ```
   Save and copy the **Template ID**.
4. **Account → General:** copy your **Public Key**. (Never use the Private Key in this project.)
5. Edit `.env`:
   ```
   VITE_EMAILJS_PUBLIC_KEY=<your public key>
   VITE_EMAILJS_SERVICE_ID=<your service id>
   VITE_EMAILJS_BOOKING_TEMPLATE_ID=<your template id>
   ```
6. **Restart** `npm run dev` (Vite reads `.env` only at startup).

### Optional: confirmation copy to the customer
Create a second template whose **To Email** is `{{to_email}}`, using the same variables, and set `VITE_EMAILJS_CUSTOMER_TEMPLATE_ID`. If it is empty, no customer copy is attempted and the UI never claims one was sent. (Free EmailJS plans limit the number of templates and monthly emails; check their pricing page.)

### Optional: contact form
Create a template using `{{from_name}}`, `{{from_email}}`, `{{message}}`, `{{reply_to}}` and set `VITE_EMAILJS_CONTACT_TEMPLATE_ID`.

### Security notes
- The Public Key, Service ID and Template IDs are meant to be public. Do **not** add a Private Key or password anywhere in the frontend.
- In the EmailJS dashboard you can restrict allowed domains and enable reCAPTCHA to reduce abuse once the site is deployed.

## 3. Test the application
**Without EmailJS configured**
1. Add a service → cart count, totals and toast update. Refresh the page → cart persists.
2. Change quantities with +/−, remove an item, confirm the empty-cart message.
3. Click **Book Now** with an empty form → field errors appear, focus jumps to the first invalid field.
4. Pick a past date (type it manually) → "Pickup date cannot be in the past."
5. Fill the form correctly with items in the cart → a yellow "Email not configured" notice appears with your details; **no** success message.

**With EmailJS configured**
1. Add services, fill the form with valid data (e.g. phone `98765 43210`), click **Book Now**.
2. A green confirmation with a reference like `FF-YYYYMMDD-XXXXX` appears and the cart clears.
3. Check the inbox of the template's "To Email" address (also check spam).
4. Break a template ID in `.env`, restart, book again → a red "Booking not sent" error appears instead of success.

**Responsive:** use browser DevTools device mode (≈375 px) and check the hamburger menu, stacked hero and single-column cards.

## 4. Build and deploy for free
```bash
npm run build      # creates the dist/ folder
npm run preview    # optional local check of the production build
```
**Netlify:** push the project to GitHub → "Add new site → Import from Git" → build command `npm run build`, publish directory `dist` → under *Site configuration → Environment variables* add the three `VITE_EMAILJS_*` values → redeploy.
**Vercel:** import the GitHub repo (Vite is auto-detected) → add the same environment variables → deploy.
**GitHub Pages:** `base: "./"` is already set; build locally with your `.env` in place and publish the `dist` folder (e.g. with the `gh-pages` package). Note `.env` is git-ignored, so values must be present at build time.

After deploying, remember to add your site's domain to EmailJS allowed origins if you enabled restrictions.

## Customising
- Services & prices: `src/data/services.js`
- Statistics, About and Contact text, business hours: `src/data/siteContent.js`
- Colours: CSS variables at the top of `src/styles/main.css`
