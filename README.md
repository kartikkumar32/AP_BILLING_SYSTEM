# Arvind Pandey Food and Catering Services — Billing System

A standalone Flask billing application with customer management, invoices, payments, PDF invoices, Excel reports, expenses, inventory and staff accounts.

## Included

- Owner/staff login and role checks
- Customers and customer ledgers
- Invoice creation with per-plate or item/quantity lines
- Optional tax rate input (enter only the legally applicable rate)
- Payment tracking and outstanding balance
- PDF invoice generation
- Excel export for invoices, payments and expenses
- Expense tracking
- Inventory with stock adjustments and low-stock alerts
- Render deployment configuration

## Local setup

Python 3.10+ recommended.

```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate
pip install -r requirements.txt
```

Set environment variables before first run. Example (macOS/Linux):

```bash
export SECRET_KEY="$(python -c 'import secrets; print(secrets.token_hex(32))')"
export OWNER_USERNAME=owner
export OWNER_PASSWORD='Use-a-unique-password-12+'
export DATABASE_PATH=billing.db
python app.py
```

On Windows PowerShell use `$env:OWNER_PASSWORD="..."` etc. If `OWNER_PASSWORD` is not set on first run, the app prints a randomly generated initial password to the terminal. Save it securely. Do not expose production secrets in source control.

Open `http://127.0.0.1:5000`. On first startup, the database tables and initial owner account are created.

## Render deployment (standalone website)

1. Push this folder to a **private GitHub repository**.
2. In Render, create a new **Web Service** and connect the repository. This is a dynamic Flask application, **not a Static Site**.
3. Set build command to `pip install -r requirements.txt`.
4. Set start command to `gunicorn app:app --workers 2 --threads 4 --timeout 120`.
5. Attach a persistent disk and mount it at `/var/data`.
6. Set environment variables:
   - `DATABASE_PATH=/var/data/billing.db`
   - `SECRET_KEY` to a long random secret
   - `OWNER_USERNAME=owner` (or a different username)
   - `OWNER_PASSWORD` to a unique strong password
   - `BUSINESS_ADDRESS`, `BUSINESS_PHONE`, and `BUSINESS_GSTIN` as appropriate
7. Deploy, then open the service URL and log in. Change secrets using Render's Environment page; never place them in Git.
8. Attach your own billing subdomain (for example `billing.yourdomain.example`) using Render's custom-domain instructions and DNS records. Replace the example with your actual domain.

**Important:** Render persistent disks are available on eligible paid web-service plans. Confirm current plan and disk pricing before deploying. A disk-mounted SQLite database is suitable for a small single-instance service; do not scale this service horizontally. Back up the database off the service disk.

## Backup and restore

The database is `DATABASE_PATH`. Schedule an off-platform backup at least daily. A simple safe manual backup while the app is stopped:

```bash
cp /var/data/billing.db ./billing-$(date +%F).db
```

For live production, use SQLite's backup API rather than copying a database file while writes are happening. Example:

```python
import sqlite3
src = sqlite3.connect("/var/data/billing.db")
dst = sqlite3.connect("billing-backup.db")
src.backup(dst)
dst.close()
src.close()
```

Store backups in encrypted, access-controlled off-site storage and test restore regularly. Keep at least 7 daily and 4 weekly copies. Do not commit database backups to GitHub.

## Business/tax configuration

The invoice template labels invoices “TAX INVOICE / BILL” and displays the configured business GSTIN when provided. This app does **not** determine whether the business must charge GST, the correct rate, place-of-supply rules, HSN/SAC, CGST/SGST/IGST split, e-invoicing eligibility, or statutory invoice fields. Have a qualified accountant confirm these before using it as a compliant tax-invoice system. Tax is configurable per invoice and defaults to 0%. Never add a GSTIN unless it belongs to the business.

## Security and operational notes

- Use HTTPS (Render provides TLS for configured domains).
- Use a strong `SECRET_KEY` and owner password.
- Staff accounts are created by the owner and require 12-character passwords.
- Do not share owner credentials.
- Limit access to trusted staff; the app contains financial/customer information.
- Test backup restore before relying on this system for business records.
- For larger concurrent usage, migrate to PostgreSQL and add automated off-site backups.
- Current starter has no password-reset flow, CSRF library, online payment gateway, WhatsApp API integration, invoice cancellation/credit notes, or automated stock deduction from catering orders. WhatsApp sharing is via a prefilled `wa.me` link on the invoice detail page.

## PDF currency and layout fix
PDF amounts use `Rs.` instead of the Unicode rupee sign because ReportLab's built-in Helvetica font can render the rupee glyph as a black square. The PDF now has a branded header, structured customer/event details, aligned item table, highlighted totals and balance, and a notes/footer section.
