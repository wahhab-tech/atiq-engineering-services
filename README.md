# Atiq Engineering & Services Corporation — Website

A full-stack business website for a generator, electrical and solar services
company: a responsive marketing site (home, about, services, contact) backed
by a Node.js/Express API that stores quote requests in MongoDB, plus a
password-protected admin dashboard to view and manage submissions.

## Folder structure

```
atiq-engineering/
├── public/                     # Front end (served statically by Express)
│   ├── index.html
│   ├── about.html
│   ├── services.html
│   ├── contact.html
│   ├── privacy-policy.html
│   ├── admin-login.html
│   ├── admin-dashboard.html
│   ├── css/
│   │   └── style.css
│   └── js/
│       ├── main.js             # nav, scroll reveal, WhatsApp links, toasts
│       ├── contact-form.js     # quote form validation + submission
│       ├── admin-auth.js       # admin login
│       └── admin-dashboard.js  # admin request table, search, delete
├── server/
│   ├── server.js                # Express app entry point
│   ├── db.js                    # MongoDB connection
│   ├── generate-admin-hash.js   # helper to hash your admin password
│   ├── models/
│   │   └── ContactRequest.js
│   ├── routes/
│   │   ├── contact.js           # POST /api/contact
│   │   └── admin.js             # POST /api/admin/login, GET/DELETE /api/requests
│   └── middleware/
│       └── auth.js              # JWT auth guard for admin routes
├── .env.example
├── .gitignore
├── package.json
└── README.md
```

## Prerequisites

- Node.js 18 or later
- A MongoDB database — either:
  - a local MongoDB server (`mongodb://127.0.0.1:27017`), or
  - a free [MongoDB Atlas](https://www.mongodb.com/atlas) cluster

## Setup

1. **Install dependencies**

   ```bash
   npm install
   ```

2. **Create your environment file**

   ```bash
   cp .env.example .env
   ```

3. **Set your MongoDB connection string** in `.env` (`MONGODB_URI`).

4. **Generate a JWT secret** and put it in `.env` as `JWT_SECRET`:

   ```bash
   node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
   ```

5. **Set your admin login.** Choose a username (`ADMIN_USERNAME`) and generate
   a bcrypt hash for your password:

   ```bash
   node server/generate-admin-hash.js "YourStrongPassword123"
   ```

   Copy the printed hash into `.env` as `ADMIN_PASSWORD_HASH`. The plain
   password is never stored — only the hash.

6. **Run the server**

   ```bash
   npm start
   ```

   or, for auto-restart during development:

   ```bash
   npm run dev
   ```

7. Open **http://localhost:3000** for the website, and
   **http://localhost:3000/admin-login.html** for the admin dashboard.

## How it works

- The **contact/quote form** (`contact.html`) validates required fields in
  the browser, then `POST`s to `/api/contact`. The API re-validates
  everything server-side, rate-limits repeated submissions from the same
  device, and stores the request in MongoDB.
- The **admin dashboard** (`admin-dashboard.html`) requires logging in at
  `admin-login.html`. Login calls `POST /api/admin/login`, which checks the
  username against `ADMIN_USERNAME` and the password against
  `ADMIN_PASSWORD_HASH` using bcrypt, then returns a signed JWT. The
  dashboard stores that token in `localStorage` and sends it as a
  `Authorization: Bearer <token>` header on every request to
  `GET /api/requests` and `DELETE /api/requests/:id`. Tokens expire after 12
  hours; expired or missing tokens redirect back to the login page.
- The **WhatsApp button** (floating on every page, plus the "WhatsApp Us"
  buttons) opens `https://wa.me/923073001398` with a pre-filled message.
- The **"Call Now"** buttons use `tel:03073001398` to open the phone dialer.
- **"Learn More"** buttons on the homepage link to matching sections on
  `services.html`, and each service's **"Request This Service"** button
  pre-fills the quote form's service dropdown via a query string.

## API reference

| Method | Endpoint             | Auth   | Description                          |
|--------|-----------------------|--------|---------------------------------------|
| POST   | `/api/contact`         | Public | Submit a quote/contact request        |
| POST   | `/api/admin/login`     | Public | Admin login, returns a JWT            |
| GET    | `/api/requests`        | Admin  | List all submitted requests           |
| DELETE | `/api/requests/:id`    | Admin  | Delete a request                      |
| GET    | `/api/health`          | Public | Health check                          |

## Security notes

- Secrets (MongoDB URI, JWT secret, admin password hash) live only in `.env`,
  which is git-ignored and never sent to the browser.
- Passwords are never stored in plain text — only a bcrypt hash.
- Both the contact form and the admin login are rate-limited per IP address.
- `helmet` sets standard security headers; input is validated and length-
  capped on both the client and the server.

## Deploying

- Set the same environment variables from `.env` on your host (Render,
  Railway, a VPS, etc.).
- Point `MONGODB_URI` at your production database (e.g. MongoDB Atlas).
- Set `CORS_ORIGIN` to your live domain once the site is public.
- Run `npm install --production` then `npm start`.
