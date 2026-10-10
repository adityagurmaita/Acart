# Acart
### Everyday, upgraded.

A complete MERN portfolio store with a marketplace-style blue search header, responsive product cards, secure accounts, checkout, order tracking and a store management dashboard. Original Acart branding; no affiliation with Flipkart.

## What's included

- Product catalogue, search, categories, price sorting, detail pages and image galleries
- Reviews and computed ratings, with one review per account per product
- Persistent cart, quantity controls and server-verified checkout prices
- Signup/login with bcrypt password hashing and JWT in an HTTP-only session cookie
- Customer order history and placed → shipped → delivered timeline
- Razorpay Orders API, checkout, server-side signature/capture checks and signed webhooks
- Clearly labelled no-charge demo checkout for portfolio testing
- Admin product add/edit/archive, inventory and order status management
- Mobile-first layouts, keyboard focus states, loading/error/empty states
- Database transactions to prevent partial inventory updates and overselling
- Automated API tests and real browser end-to-end tests

**This is working application source, not a static UI mock. It has not been publicly deployed.** The bundled catalogue contains fictional products/specifications/prices and illustrative stock photography. It is a portfolio project, not a ready-to-trade business.

## Start here (Windows / laptop)

You need Node.js 22+, internet access, and a MongoDB Atlas connection string. Atlas is simpler than installing MongoDB locally. See `docs/DEPLOY.md` for database setup.

1. Extract the ZIP. Open the extracted **Acart** folder in VS Code: File → Open Folder.
2. In VS Code, choose **Terminal → New Terminal**. This opens a terminal inside the right folder.
3. Type this, then press Enter:
   ```text
   npm install
   ```
4. In VS Code's file list, copy `server/.env.example` and rename the copy to `server/.env` (not `.env.txt`).
5. Edit that new file:
   - `MONGODB_URI`: paste your Atlas connection string. Include `/acart` as the database name.
   - `JWT_SECRET`: enter a long random secret, at least 32 characters. You can generate one in the terminal with:
     ```text
     node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
     ```
   - `ADMIN_EMAIL`: choose your admin email.
   - `ADMIN_PASSWORD`: choose a password with at least 10 characters. This creates the admin only when seeded.
   - Keep `ENABLE_DEMO_PAYMENTS=true` for no-charge testing.
   - Leave Razorpay keys blank until you have your own **test-mode** keys.
6. Save the file. Then type:
   ```text
   npm run seed
   npm run dev
   ```
7. Open **http://localhost:5173** in your browser. Leave the terminal running.
8. Create a customer account using Login → Create account. Browse, add an item, open Cart, and place a **demo** order.
9. Sign out, then sign in with the admin email/password from step 5. Click **Admin** to manage products and orders.
10. To stop the app, click the terminal and press **Ctrl+C**.

The seed script preserves existing records. It will not reset an existing admin's password. The admin page is at `/admin`, and the backend enforces its role even if a customer opens that URL manually.

### Using a phone

Once deployed, open the deployment's HTTPS link on your phone. You can shop, create accounts, review products and manage the admin dashboard there. A ZIP alone does not run a Node/MongoDB app on Android. Initial upload/deployment needs help or a computer/browser-based development workspace; see the deploy guide. No paid service is required by the code itself, but hosting provider availability/pricing must be checked when deploying.

### Local MongoDB alternative

MongoDB **must run as a replica set** because checkout uses transactions. A standalone local database will not work for checkout. Use Atlas or run the included `docker-compose.yml` with Docker Desktop:

```text
docker compose up -d
```

Then keep the sample URI `mongodb://127.0.0.1:27017/acart?replicaSet=rs0`. The database remains in a named Docker volume. Do not use this unauthenticated, localhost-only database as public production infrastructure.

## Razorpay setup

1. Generate **test-mode** API keys in your own Razorpay dashboard.
2. Add `RAZORPAY_KEY_ID` and `RAZORPAY_KEY_SECRET` to `server/.env`. Never put the secret in the frontend or GitHub.
3. Enable automatic payment capture in Razorpay. The server checks `captured` status, order ID, INR currency and amount before confirming the order.
4. For a deployed site, set a webhook endpoint to `https://YOUR-REAL-HOST/api/payments/webhook`, choose `payment.captured`, and use the same webhook secret in `RAZORPAY_WEBHOOK_SECRET`.
5. Restart the server and run a complete test-mode payment, including failure/cancellation paths.
6. Real payments require your own activated merchant account, live keys, proper business policies and a verified live deployment. Set `ENABLE_DEMO_PAYMENTS=false` before live keys. The server refuses live keys with demo payments enabled.

**No real or Razorpay-hosted test payment was made during delivery.** Payment logic was checked with controlled gateway responses, signature validation and webhook tests, but the real gateway must be tested with your own keys. No credentials are included.

### Payment failure safety

Unconfirmed payments appear as pending in order history. Check history and the Razorpay dashboard before paying again. Webhooks can complete orders when the browser callback is missed. Stock is deducted transactionally on confirmation, not reserved while a Razorpay checkout is open. If stock runs out after a captured payment, settlement fails safely and requires merchant support/refund handling. Automatic refunds, payment retries on a pending order, returns and cancellations are not implemented.

## Tests

```text
npm test
npm run build
npx playwright install chromium
npm run test:ui
```

The tests start a temporary MongoDB replica set. Its first run downloads MongoDB; internet is required. Browser tests use port 5000, so stop the regular server first. Screenshots are written to `test-results/`. Test accounts only exist in the temporary test database; there is no permanent default admin password.

Verified at delivery: 13 backend tests passed, production build passed, complete browser shopping/admin flow passed. Responsive route screenshots were captured at 320, 390 and 1280 pixels with no horizontal overflow or browser runtime errors.

## Structure

```text
client/src/pages/       Home, product, cart, auth, checkout, orders, admin
client/src/components/  Shared UI and access gates
client/src/store.jsx    Account and cart context
client/src/api.js       API client and price/rating helpers
client/public/images/  Bundled illustrative product photographs
server/src/app.js       Validated API routes, auth, checkout, payments, admin
server/src/models.js    Mongoose user/product/order schemas
server/src/catalog.js   Seed catalogue
server/src/seed.js      Non-destructive database seed
server/test/           API integration tests
scripts/ui-test.mjs     Browser flow and responsive checks
screenshots/           Delivery screenshots
```

Production Express serves the built React app and API from one origin, avoiding cross-site cookie complexity. `VITE_API_URL` is optional for separate deployments, but separate cross-site auth needs an explicit cookie/CORS design change. Recommended deployment is same-origin.

## Security and limitations

- Backend validates input, hashes passwords, rejects role injection and untrusted origins, rate-limits requests, and uses secure cookies in production.
- Prices and totals are computed from database products, never trusted from the browser cart.
- Admin deletion archives products while preserving existing order line items.
- Cart storage is local to your browser. Prices can change; final server checkout totals are authoritative.
- Shipping is a portfolio rule: ₹49 below ₹1,999, free at or above ₹1,999. All catalogue prices include taxes; no tax invoices are generated.
- Reviews come from signed-in accounts but are not verified-purchase reviews.
- Admin order list shows the latest 200 orders. Catalogue is capped at 100 results. Pagination is a future improvement.
- Order statuses are admin updates, not courier integration. No emails, SMS, seller marketplace, coupons, returns, password reset, email verification, or automatic fulfilment.
- Arbitrary HTTPS product image links can expose viewers' IP addresses to their hosts. Prefer your own trusted image storage; upload handling is not included.
- Before commercial use: add stock reservations/refunds, payment reconciliation/support, monitoring/backups, business policies, privacy/terms, tax compliance, account recovery and a separate security review.

## References

- Razorpay: https://razorpay.com/docs/payments/server-integration/nodejs/integration-steps/
- Render: https://render.com/docs/deploy-node-express-app
- Atlas: https://www.mongodb.com/docs/atlas/tutorial/deploy-free-tier-cluster/
- Image source list: `docs/IMAGE_CREDITS.md`
