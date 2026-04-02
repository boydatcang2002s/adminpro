# MIYU Nails & Flow Spa — Booking System

Full-stack booking system with customer-facing booking page and admin dashboard.

## Stack

- **Backend:** Node.js + Express
- **Database:** SQLite via `better-sqlite3` (zero-config, file-based, no external DB needed)
- **Auth:** JWT (JSON Web Tokens) with bcrypt password hashing
- **Frontend:** Vanilla HTML/CSS/JS (no framework dependencies)

## Project Structure

```
miyu-backend/
├── server.js           # Main Express server
├── db.js               # SQLite schema, seeding, database instance
├── package.json
├── .env.example        # Copy to .env before running
├── routes/
│   ├── auth.js         # Login, logout, change password
│   ├── bookings.js     # Create/read/update bookings
│   ├── nailers.js      # Nail artist management
│   ├── services.js     # Treatment/service management
│   ├── slots.js        # Availability slot management
│   └── customers.js    # Customer CRM
├── middleware/
│   └── auth.js         # JWT middleware
└── public/             # Served as static files
    ├── index.html      # Customer booking page
    └── admin.html      # Admin dashboard
```

## Quick Start

### 1. Install dependencies

```bash
npm install
```

### 2. Configure environment

```bash
cp .env.example .env
# Edit .env and set your JWT_SECRET and admin credentials
```

### 3. Run

```bash
# Development (auto-restart)
npm run dev

# Production
npm start
```

The server starts on http://localhost:3000 (or `PORT` from .env).

---

## Pages

| URL | Description |
|-----|-------------|
| `/` | Customer booking page |
| `/admin.html` | Admin dashboard (requires login) |
| `/api/health` | API health check |

---

## Default Admin Login

```
Username: admin
Password: miyu2024secure
```

**Change this immediately** in `.env` before deploying!

---

## API Endpoints

### Public (no auth required)
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/services` | List active services |
| GET | `/api/nailers` | List active nail artists |
| GET | `/api/slots?nailer_id=&date=` | Get slots for a nailer/date |
| GET | `/api/slots/available-dates?nailer_id=` | Dates with free slots |
| POST | `/api/bookings` | Create a booking |
| GET | `/api/bookings/ref/:ref` | Look up booking by reference |

### Admin (requires JWT token)
| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/auth/login` | Login |
| GET | `/api/auth/me` | Verify token |
| GET | `/api/bookings` | List all bookings |
| GET | `/api/bookings/today` | Today's bookings |
| GET | `/api/bookings/stats` | Dashboard statistics |
| PATCH | `/api/bookings/:id/status` | Update booking status |
| GET/POST/PUT/DELETE | `/api/nailers/...` | Manage nail artists |
| GET/POST/PUT/DELETE | `/api/services/...` | Manage services |
| GET/POST/DELETE | `/api/slots/...` | Manage availability slots |
| GET/POST/PUT/DELETE | `/api/customers/...` | Customer CRM |

---

## Deployment Options

### Option A: VPS / Linux server

```bash
# Install Node.js 18+
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Clone/upload your project
cd /var/www/miyu
npm install --production

# Use PM2 for process management
npm install -g pm2
pm2 start server.js --name miyu
pm2 save
pm2 startup

# Set up Nginx as reverse proxy (recommended)
# Point domain to port 3000
```

### Option B: Railway.app (Recommended — free tier available)

1. Push code to GitHub
2. Connect repo to [Railway.app](https://railway.app)
3. Set environment variables in Railway dashboard
4. Deploy — Railway auto-detects Node.js

### Option C: Render.com

1. Create a new Web Service
2. Connect GitHub repo
3. Build command: `npm install`
4. Start command: `node server.js`
5. Add environment variables

### Option D: Heroku

```bash
heroku create miyu-studio
heroku config:set JWT_SECRET=your_long_secret NODE_ENV=production
git push heroku main
```

---

## Nginx Config (for VPS)

```nginx
server {
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Then: `sudo certbot --nginx -d yourdomain.com` for HTTPS.

---

## Database

SQLite database is stored as `./miyu.db` (or path from `DB_PATH` env var).

The database is **auto-created and seeded** on first run with:
- 4 nail artists (Miyu, Linh, Sophie, Hana)
- 6 services (Klassische Maniküre, Gel-Maniküre, Russische Maniküre, etc.)
- Available slots for the next 30 days

**Backup:** Simply copy `miyu.db` — it's a single file.

---

## Security Notes

1. Change `JWT_SECRET` to a long random string (32+ chars)
2. Change default admin password
3. Use HTTPS in production (Nginx + Let's Encrypt)
4. The admin panel is at `/admin.html` — consider adding IP-based access control
