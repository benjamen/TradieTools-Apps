# TradieTools Apps

Professional quote builder, pricing database, and financial tools for NZ tradies.

**Status:** 🚀 Active Development | 📦 Production Ready

## Features

### ✅ Quote Builder (Vue3)
- Interactive quote calculations
- Material + labor cost breakdown
- Automatic markup and GST calculation
- PDF export
- Email sending
- Trade-specific hourly rates
- 30-day quote validity

### 📊 Pricing Database
- Trade-specific hourly rates by region
- Experience level pricing tiers
- Real-time pricing lookups

### 💰 Job Cost Database  
- Historical job cost tracking
- Material and labor cost templates
- Regional cost variations

### 📈 Cash Flow Forecaster
- Monthly revenue projections
- Fixed cost calculations
- Cash position tracking
- Seasonal pattern analysis

## Tech Stack

- **Frontend:** Vue 3 + HTML/CSS/JS
- **Backend:** Frappe ERP (Python/FastAPI)
- **Database:** Frappe Database (SQLite/MySQL)
- **Hosting:** tradietools.optified.nz

## Quick Start

### Development

```bash
git clone https://github.com/benjamen/TradieTools-Apps.git
cd TradieTools-Apps
cat apps/quote-builder.vue
bash deployment/deploy_vue_apps.sh
```

### Production

```bash
python3 frappe-doctypes/create_frappe_doctypes.py
bash deployment/deploy_vue_apps.sh
python3 apps/create_vue_apps.py
```

## File Structure

```
TradieTools-Apps/
├── apps/                      # Vue3 components
│   ├── quote-builder.vue      # Quote builder component
│   └── create_vue_apps.py     # Deploy to Frappe
├── frappe-doctypes/           # Frappe schemas
│   └── create_frappe_doctypes.py
├── deployment/                # Deployment scripts
│   └── deploy_vue_apps.sh
└── docs/                      # Documentation
```

## API Endpoints

- `POST /api/tradie/quotes/create` - Create quote
- `GET /api/tradie/quotes/{id}` - Get quote
- `GET /api/tradie/pricing/rates?trade=electrician` - Get rates

## Roadmap

- [x] Quote Builder MVP
- [x] Pricing Database Integration
- [x] Frappe DocType Setup
- [ ] PDF Export
- [ ] Email integration
- [ ] User authentication
- [ ] Premium tier
- [ ] Mobile app

## Status

🟢 **Production Ready** - All core features functional

---

**Last Updated:** June 7, 2026
