# VoltLift Equipment Website
### Complete, deployable industrial equipment website with built-in Admin CMS

---

## 📁 Project Structure

```
voltlift/
├── index.html              ← Main website (homepage)
├── vl-console-8x2/   (renamed — see Security section)
│   └── index.html          ← Admin CMS panel
├── assets/
│   └── data.js             ← Default seed data (products + settings)
├── data/
│   ├── products.json       ← Reference product data (not directly read by browser)
│   └── settings.json       ← Reference settings data
└── README.md               ← This file
```

---

## 🚀 Quick Start — Run Locally

### Option A: Open Directly in Browser (Simplest — No Server Needed)

1. Unzip the downloaded folder
2. Double-click `index.html` to open the website
3. Double-click `vl-console-8x2/index.html` to open the Admin Panel
4. That's it! All data is stored in your browser's localStorage.

> **Note:** Image uploads work best when served via a local server (Option B below).
> If you open via `file://`, some browsers may block base64 image reading from file uploads.

---

### Option B: Local Server with Python (Recommended for Development)

Python is pre-installed on macOS and most Linux systems.

```bash
# Navigate into the project folder
cd voltlift

# Python 3 (macOS, Linux, Windows with Python 3)
python3 -m http.server 3000

# Python 2 (older systems)
python -m SimpleHTTPServer 3000
```

Then open:
- **Website:** http://localhost:3000
- **Admin Panel:** http://localhost:3000/vl-console-8x2/

---

### Option C: Local Server with Node.js

If you have Node.js installed:

```bash
# Install a simple static server globally (one-time)
npm install -g serve

# Navigate to the project folder and serve it
cd voltlift
serve -p 3000

# OR use npx (no global install needed)
npx serve -p 3000
```

Then open:
- **Website:** http://localhost:3000
- **Admin Panel:** http://localhost:3000/vl-console-8x2/

---

### Option D: Local Server with VS Code Live Server Extension

1. Open the `voltlift` folder in VS Code
2. Install the **Live Server** extension (by Ritwick Dey)
3. Right-click `index.html` → **Open with Live Server**
4. Admin panel: navigate to `vl-console-8x2/index.html` in the browser

---

## 🌐 Production Deployment

### Option 1: Netlify (Free, Recommended — 2 Minutes)

1. Go to [netlify.com](https://netlify.com) and sign up (free)
2. Drag and drop the entire `voltlift` folder onto the Netlify dashboard
3. Your site is live instantly at a `*.netlify.app` URL
4. To use a custom domain: Site Settings → Domain Management → Add custom domain

**To update the site later:** Simply drag the updated folder again or connect to a Git repo for auto-deployment.

---

### Option 2: Vercel (Free)

```bash
# Install Vercel CLI
npm install -g vercel

# Navigate to project folder
cd voltlift

# Deploy (follow prompts — choose "no framework")
vercel

# For production deployment
vercel --prod
```

---

### Option 3: GitHub Pages (Free)

1. Create a new GitHub repository
2. Upload all files in the `voltlift` folder to the repo root
3. Go to **Settings → Pages → Source → main branch / root**
4. Site will be live at `https://yourusername.github.io/repo-name`

---

### Option 4: Traditional Web Hosting (cPanel, Hostinger, GoDaddy, etc.)

1. Log in to your hosting control panel
2. Open **File Manager** → navigate to `public_html`
3. Upload all files from the `voltlift` folder
4. That's it — your site is live at your domain

---

### Option 5: VPS / Linux Server (Nginx)

```bash
# Copy files to server
scp -r voltlift/ user@your-server:/var/www/voltlift/

# Nginx config (create /etc/nginx/sites-available/voltlift)
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    root /var/www/voltlift;
    index index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|webp)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
}

# Enable site
sudo ln -s /etc/nginx/sites-available/voltlift /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx

# Add HTTPS with Let's Encrypt
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

---

## 🔧 Admin Panel Guide

Access the admin panel at `yoursite.com/vl-console-8x2/` or `vl-console-8x2/index.html` locally.

### Managing Products

| Action | Steps |
|--------|-------|
| **Add Product** | Click "Add Product" button → Fill form → Save |
| **Edit Product** | Click "Edit" on any product row |
| **Delete Product** | Click "Delete" → Confirm |
| **Add Image** | Drag & drop or click to upload (max 2MB), or paste an image URL |
| **Filter Products** | Use the search box or category dropdown |

### Product Fields

- **Name** *(required)* — Full product name
- **Category** *(required)* — Used for filter tabs on the website
- **Category Label** — Display name shown on cards and filter buttons
- **Spec Line** — Short technical spec shown on the card
- **Description** — Full description shown in the product popup
- **Badge** — Optional label (Bestseller, New, Popular, etc.)
- **Image** — Upload a file (stored as base64) or paste a URL
- **Key Features** — Bullet-point features shown in the product popup

### Site Settings

Update business info, hero text, contact details and stats in the **Site Settings** page. Click **Save Settings** — changes appear on the live site after a page refresh.

---

## 💾 Data Storage Explained

### Default Behaviour (localStorage)
All data is stored in the browser's `localStorage`. This means:
- ✅ Zero backend setup required
- ✅ Works on any static host
- ⚠️ Data is per-browser — different devices/browsers won't share data
- ⚠️ Clearing browser data will erase products

### Making Data Permanent (Recommended for Production)

**Method 1 — Update `assets/data.js` (simplest)**

1. Go to Admin → Import/Export → **Export Everything**
2. Open the downloaded `voltlift-all-data.json`
3. Open `assets/data.js` in a text editor
4. Replace the `VL_DEFAULT_PRODUCTS` array with the `products` array from your export
5. Replace the `VL_DEFAULT_SETTINGS` object with the `settings` object from your export
6. Re-deploy the updated files

This way, any new visitor will see your latest data even before they've used the admin panel.

**Method 2 — Node.js / Express Backend (for teams)**

A minimal starter server that reads/writes JSON files:

```js
// server.js — run with: node server.js
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();

app.use(express.json({ limit: '10mb' }));
app.use(express.static('.'));  // serve all static files

const PRODUCTS_FILE = path.join(__dirname, 'data/products.json');
const SETTINGS_FILE = path.join(__dirname, 'data/settings.json');

// GET products
app.get('/api/products', (req, res) => {
  try { res.json(JSON.parse(fs.readFileSync(PRODUCTS_FILE, 'utf8'))); }
  catch(e) { res.json([]); }
});

// PUT products (full replace)
app.put('/api/products', (req, res) => {
  fs.writeFileSync(PRODUCTS_FILE, JSON.stringify(req.body, null, 2));
  res.json({ ok: true });
});

// GET settings
app.get('/api/settings', (req, res) => {
  try { res.json(JSON.parse(fs.readFileSync(SETTINGS_FILE, 'utf8'))); }
  catch(e) { res.json({}); }
});

// PUT settings
app.put('/api/settings', (req, res) => {
  fs.writeFileSync(SETTINGS_FILE, JSON.stringify(req.body, null, 2));
  res.json({ ok: true });
});

app.listen(3000, () => console.log('Server running at http://localhost:3000'));
```

Install & run:
```bash
npm init -y
npm install express
node server.js
```

Then update `index.html` and `vl-console-8x2/index.html` to fetch from `/api/products` and `/api/settings` instead of localStorage.

---

## 🛡️ Protecting the Admin Panel

**Two layers of protection are already applied in this build:**

1. **Renamed & unlinked** — the admin panel now lives at `vl-console-8x2/` instead of `admin/`, and every public-facing link to it (nav bar, mobile menu, footer, empty-state message) has been removed from `index.html`. It's no longer discoverable by browsing the site — only by knowing the direct URL.
2. **Login-gated on Azure Static Web Apps** — `staticwebapp.config.json` restricts the `/vl-console-8x2/*` route to `authenticated` users only. Anyone hitting that path without being signed in is redirected to Azure AD login. This only takes effect if you deploy via **Azure Static Web Apps** (see deployment section above) — it has no effect on Blob Storage, Netlify, or plain Nginx hosting.

**If you're not using Azure Static Web Apps**, add one more layer:

**Option A — Netlify Password Protection**
Enable in Netlify dashboard: Site Settings → Access Control → Basic Authentication

**Option B — .htaccess (Apache hosting)**
Create `vl-console-8x2/.htaccess`:
```
AuthType Basic
AuthName "Admin"
AuthUserFile /path/to/.htpasswd
Require valid-user
```
Create `.htpasswd`:
```bash
htpasswd -c .htpasswd admin
```

**Rotate the console name periodically** — treat `vl-console-8x2` as a shared secret. If it ever leaks (browser history, server logs, a screenshot), rename the folder again and update `staticwebapp.config.json` to match.

---

## 🖼️ Adding Product Images

### From a URL
Paste any public image URL in the "Image URL" field in the admin panel.

### By Uploading
Upload directly in the admin (max 2MB). The image is stored as base64 inside localStorage.
> For production: if you have many large images, serve them from a CDN (Cloudinary, ImgBB, Bunny CDN) and use URLs instead of uploads.

### Free Image Hosting Options
- [Cloudinary](https://cloudinary.com) — free tier, great for product photos
- [ImgBB](https://imgbb.com) — free, no account needed
- [Uploadcare](https://uploadcare.com) — free tier available

---

## 📞 Customising Contact & WhatsApp

1. Go to **Admin → Site Settings**
2. Update Phone, Email, Address and WhatsApp number
3. The WhatsApp field should be digits only, e.g. `919876543210` (country code + number)
4. Click **Save Settings**

---

## 🎨 Customising Colours

Open `index.html` in a text editor. Find the `:root` CSS block:

```css
:root {
  --orange: #f5550a;      /* Primary accent — change to your brand colour */
  --orange-light: #ff7a3d;
  --steel: #1a1e25;       /* Card backgrounds */
  --steel-mid: #242933;
  --steel-light: #2f3542;
  /* ... */
}
```

Change `--orange` to any hex colour (e.g. `#0066cc` for blue) to re-theme the entire site.

---

## ❓ FAQ

**Q: Will my products disappear if I clear browser cache?**
A: Clearing cache/localStorage will reset to defaults defined in `assets/data.js`. Export your data regularly or update `data.js` with your latest content.

**Q: Can multiple people manage the site at the same time?**
A: Not with localStorage. For team use, implement the Node.js backend (see above) or use a headless CMS like Contentful.

**Q: How do I add a new category not in the dropdown?**
A: Type a custom slug in the category field and a label in "Category Display Label". Both new filter buttons and nav dropdown will auto-generate from your products.

**Q: Can I have product detail pages?**
A: Currently products open in a popup modal. For dedicated pages, each product would need its own HTML file — this would require a build step (like 11ty or Next.js).

---

## 📄 Licence

This codebase is provided for your personal or commercial use. No attribution required.
