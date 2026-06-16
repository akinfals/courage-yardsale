# Courage Events — Yard Sale Site

Live site: **yardsales.courageevents.com**
Fly.io app: `courage-yardsale.fly.dev`

A static single-page site that displays clearance inventory from a Google Sheet. No backend required — inventory updates happen by editing the sheet.

---

## How It Works

```
Google Sheet (inventory) → fetched as CSV on page load → rendered as item cards
Google Drive folders     → linked per item for photos
```

The site fetches `Sheet1` from the Google Sheet every time a visitor opens it. No database, no server-side code — just nginx serving a single HTML file.

---

## Project Structure

```
yardsale-flyio/
├── index.html       # Entire site — HTML, CSS, and JS in one file
├── Dockerfile       # nginx:alpine container
├── nginx.conf       # nginx server config (port 80, gzip)
├── fly.toml         # Fly.io deployment config
└── README.md        # This file
```

---

## Inventory — Google Sheet

**Sheet URL:** https://docs.google.com/spreadsheets/d/181foEO-8VxSYDapAOcm7ZGhb98t90AkWeQVoVtb25Dg/edit

**Sheet ID (in code):** `181foEO-8VxSYDapAOcm7ZGhb98t90AkWeQVoVtb25Dg`

**Tab used:** `Sheet1`

### Column structure

| Column | Description |
|--------|-------------|
| S/N | Row number (must be a digit — used to detect data rows) |
| Item Name | Display name on the card |
| Link to Folder | Google Drive folder URL for photos |
| Price per piece | e.g. `10,000` — displayed as ₦ |
| Qty | Stock count. `0` = sold out. ≤3 = "low stock" warning |

### To update inventory
Just edit the sheet. Changes appear on the site within seconds (no redeployment).

> ⚠️ The sheet must remain set to **"Anyone with the link → Viewer"** or the CSV fetch will fail with an auth error.

---

## Contact

**Christina:** 09024613968
Every item card has a "Buy Now" button that dials this number directly.

---

## Local Development

No build step needed — open `index.html` directly in a browser.

```bash
open index.html
# or
python3 -m http.server 8080
# then visit http://localhost:8080
```

> Note: The Google Sheet CSV fetch may be blocked by CORS in some browsers when opening the file directly. Use the `python3` server method to avoid this.

---

## Deployment — Fly.io

### First-time setup

```bash
# 1. Install flyctl
brew install flyctl          # macOS
# curl -L https://fly.io/install.sh | sh   # Linux/Mac without brew

# 2. Login
flyctl auth login

# 3. From this folder — create the app (first time only)
flyctl launch --name courage-yardsale --no-deploy

# 4. Deploy
flyctl deploy
```

App will be live at: `https://courage-yardsale.fly.dev`

### Subsequent deployments (after editing index.html)

```bash
flyctl deploy
```

That's it. Fly.io rebuilds the Docker image and redeploys in ~30 seconds.

### Fly.io config summary (`fly.toml`)

| Setting | Value | Reason |
|---------|-------|--------|
| `primary_region` | `jnb` | Johannesburg — closest to Nigeria |
| `auto_stop_machines` | `stop` | Saves cost; machine starts on first request |
| `min_machines_running` | `0` | Free tier friendly |
| `memory` | `256mb` | More than enough for static HTML |

---

## Custom Domain — yardsales.courageevents.com

### Step 1 — Add cert on Fly.io
```bash
flyctl certs add yardsales.courageevents.com
```

### Step 2 — Add DNS record
In your DNS provider (wherever `courageevents.com` is managed), add:

| Type | Name | Value |
|------|------|-------|
| CNAME | `yardsales` | `courage-yardsale.fly.dev` |

Fly.io will auto-provision an SSL certificate via Let's Encrypt within a few minutes.

---

## GitHub — Keeping It Updated

```bash
# Clone
git clone https://github.com/YOUR_USERNAME/courage-yardsale.git
cd courage-yardsale

# Make changes to index.html, then:
git add .
git commit -m "describe what you changed"
git push

# Then redeploy to Fly.io
flyctl deploy
```

---

## Known Limitations & Future Work

| Limitation | Fix |
|------------|-----|
| Photos are in Drive folders, not embedded | Replace Drive folder links with direct image URLs, or integrate Google Drive API to pull thumbnails |
| No cart/checkout | Purchase is call-only by design. Could add WhatsApp deep link as alternative |
| Sheet must be public | Could swap to Google Sheets API with a service account key for a private sheet |
| No search/filter | Could add client-side filtering by category or price range |

---

## Troubleshooting

**Site shows "Could not load inventory"**
- Check the Google Sheet is set to "Anyone with the link → Viewer"
- Open browser DevTools → Network tab → look for a failed request to `docs.google.com/spreadsheets`

**Fly.io deploy fails**
- Run `flyctl logs` to see what went wrong
- Make sure you're in the `yardsale-flyio` folder when running `flyctl deploy`

**Items not updating after sheet edit**
- The site fetches fresh data on each page load. Hard-refresh the browser (Cmd+Shift+R / Ctrl+Shift+R) to bypass cache.

---

*Built June 2026 · Courage Events*
