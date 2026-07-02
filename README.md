# 🛒 Shiv Ganga Super Market

Fresh groceries and daily essentials, delivered fast in Uttam Nagar, Delhi.
A full-featured online storefront for a real neighbourhood supermarket —
built as a single-page web app with a Firebase backend and Cloudinary image
hosting.

🔗 **Live site:** [shivganga-app.vercel.app](https://shivganga-app.vercel.app/)

## About

Shiv Ganga Super Market is a real, operating store located at Plot No 24,
Vipin Garden, Dwarka Mor, Uttam Nagar, Delhi. This app is its online
ordering platform, offering 30-minute delivery to the local area.

## Features

- 🏬 **Storefront** — browse by category, view popular items, search
- 🛒 **Cart & checkout** — standard and express delivery speeds, coupon
  codes, GST calculation, and Cash on Delivery (UPI/card/cash accepted at
  the door)
- 👤 **Customer accounts** — phone number + 4-digit PIN login/registration,
  order history
- 📦 **Order tracking** — track any order by ID, live status updates
- 🧾 **Invoices** — printable/PDF order invoices
- 💬 **WhatsApp integration** — order confirmations and direct WhatsApp
  ordering
- 📱 **Installable** — add-to-home-screen support for a native-app-like
  experience
- ⚙️ **Full admin dashboard**:
  - Overview stats (orders, revenue, customers, products)
  - Order management with status updates
  - Product & inventory management with low-stock alerts
  - Customer list and spending history
  - Coupon management (percentage, flat, or free-delivery discounts)
  - Sales reports by day/week/month
  - Store open/closed toggle (instantly stops customers from ordering)
  - Announcement bar editor
  - Cloudinary image upload per product
  - Manual Firebase data sync tools

## Tech Stack

- **Frontend** — single-page app (vanilla JS), installable as a PWA
- **Backend / database** — [Firebase](https://firebase.google.com)
  Realtime Database, with [Firebase Authentication](https://firebase.google.com/docs/auth)
  gating all admin-level access
- **Images** — [Cloudinary](https://cloudinary.com) (unsigned upload
  preset, auto-optimized to WebP/AVIF, resized to 400×400)
- **Notifications** — [Web3Forms](https://web3forms.com) for order email
  notifications, WhatsApp deep links for order confirmation
- **Hosting** — [Vercel](https://vercel.com)

## Getting Started

1. Clone the repo
2. Create a [Firebase](https://firebase.google.com) project:
   - Add a Web App and copy the config into `FIREBASE_CONFIG` in
     `index.html`
   - Enable **Realtime Database**
   - Enable **Authentication → Email/Password** sign-in
   - Create one admin user under **Authentication → Users**, and copy
     their UID
   - Add that UID to `shivganga/admins` in the database (see
     [Security](#security) below) so it's recognized as an admin
   - Apply the database rules from the [Security](#security) section
3. Set up a Cloudinary account:
   - Go to **Cloudinary Console → Upload Presets**
   - Add an unsigned upload preset named `shivganga_unsigned`, folder
     `shivganga_products`
4. Configure a [Web3Forms](https://web3forms.com) access key for order
   email notifications
5. Deploy with [Vercel](https://vercel.com)

## Security

Firebase config values (`apiKey`, `authDomain`, `projectId`, etc.) are
public identifiers, not secrets, and are safe to have in client-side code —
this is standard for Firebase apps. Real access control is enforced
entirely through **Firebase Authentication + Realtime Database Security
Rules**, not through anything in the client.

**Admin access model:**
- Admin identity is verified through Firebase Authentication
  (email/password), not a password hash stored in the database
- A `shivganga/admins` node lists authorized admin UIDs; only UIDs in this
  list are granted write access by the database rules below
- The `admins` list itself is writable only manually via the Firebase
  Console — never from client code

**Database rules** (Realtime Database → Rules):

```json
{
  "rules": {
    "shivganga": {
      "admins": {
        ".read": "auth != null && root.child('shivganga/admins').child(auth.uid).val() === true",
        ".write": false
      },
      "data": {
        "sg_orders": {
          ".read": "auth != null && root.child('shivganga/admins').child(auth.uid).val() === true",
          ".write": true
        },
        ".read": true,
        ".write": "auth != null && root.child('shivganga/admins').child(auth.uid).val() === true"
      },
      "images": {
        ".read": true,
        ".write": "auth != null && root.child('shivganga/admins').child(auth.uid).val() === true"
      }
    }
  }
}
```

What this enforces:
- Product/catalog/settings **reads** stay public — customers need this to
  browse
- Product/catalog/settings/image/admin-data **writes** require a signed-in
  Firebase user whose UID is in `shivganga/admins`
- Customers can still **place** orders (public write to `sg_orders`) but
  cannot **read** the full order list — this prevents scraping of other
  customers' names, phone numbers, and addresses
- No plaintext or hashed admin password is ever stored in the database

**If you fork or redeploy this project:** don't skip re-applying these
rules and re-creating the admin user — a fresh Firebase project defaults
to fully open or fully closed rules depending on setup mode, neither of
which matches what this app needs.

## Project Structure

```
index.html   # Full app: storefront, cart, checkout, customer accounts,
             # and admin dashboard
```

## Status

This powers a real, operating local business — order data, customer
contact details, and admin access are treated with production-level care,
not as a demo. Cash on Delivery is used; no card/payment data is
processed or stored by this app.

## License

All Rights Reserved — see [LICENSE](LICENSE) for details. This code is
public for portfolio/reference purposes only; it may not be copied,
reused, or redeployed without permission.
