# SUKA Project

SUKA is a POS project for Indonesian businesses. The idea is simple: give business owners a cashier system they can use for free, even when the internet is down. When they need more features, they can add them through optional plugins.

This repository is the home for the product direction, technical decisions, and development progress. As the project grows, this README will be updated so the team has one place to understand what SUKA is and what still needs to be built.

> **Current stage:** Product planning and architecture. Items described as planned are not yet implemented unless they are marked as completed in the update log.

## What we're building

SUKA starts with a free POS and is planned to grow into a set of connected tools for running a business.

The core product should cover everyday shop operations:

- Register and sign in as an owner.
- Create and manage stores and outlets.
- Add employees and assign roles.
- Manage products, categories, and basic stock.
- Build a cart and complete sales.
- Keep transaction history.
- Show basic sales summaries and reports.
- Keep working when the internet is unavailable.

Extra capabilities will be available as plugins. Owners can choose which ones they need instead of paying for a large package full of features they may never use.

## Product rules

These are the decisions we want development to follow:

1. **The core POS is free.** Basic cashier operations should not require a paid plan.
2. **No per-outlet or per-employee fee for the core POS.** Owners can add outlets and employees without buying a separate subscription for each one.
3. **Plugins are optional and paid monthly.** An owner only subscribes to the plugins they want.
4. **Subscriptions belong to the owner account.** One plugin subscription applies to all outlets owned by that account.
5. **Offline sales must work.** A cashier should not be blocked from completing a sale because the internet is unavailable.
6. **Save locally first.** A transaction is written to the device before the app tries to send it to the server.
7. **Keep business data safe.** A failed sync or expired plugin must not erase core POS data.
8. **Permissions matter.** Owners decide what employees can see and do.

## What's included in the free POS

The current plan is to keep these core features free:

- Owner registration and login.
- Store and outlet management.
- Employee management.
- Roles and permissions.
- Products and categories.
- Cart and checkout.
- Sales transactions.
- Basic stock management.
- Transaction history.
- Cashier settings.
- Basic sales summaries and local reports.
- Local transaction storage.
- Cloud sync when an internet connection is available.

The free core is intended to have no per-outlet, per-employee, per-device, or per-product charge. If infrastructure limits or fair-use rules are needed later, they must be clear to users.

## How offline and online work together

The local database is the first place a transaction is saved. The server is used for account services, cloud copies of data, and remote monitoring.

### When the cashier is online

1. The cashier completes a sale.
2. SUKA saves the transaction to the device.
3. SUKA sends it to the backend.
4. The backend checks the request and stores the transaction.
5. Once the server accepts it, the owner's dashboard can show the sale.

### When the cashier is offline

1. The cashier completes a sale as usual.
2. SUKA saves it to the device.
3. The transaction waits in a local sync queue.
4. The cashier can continue serving customers.
5. When the connection returns, SUKA retries the pending upload.
6. The owner can see the sale remotely after it reaches the server.

A remote owner cannot see a transaction that is still only on an offline cashier device. That is an unavoidable part of this design.

### Sync behavior we need

- Give every transaction a unique, stable ID.
- Keep pending transactions on the device until the server confirms receipt.
- Make uploads idempotent, so retrying a request cannot create a duplicate sale.
- Track states such as `pending`, `syncing`, `synced`, and `failed`.
- Retry temporary failures safely.
- Validate the owner, outlet, employee, and device on the backend.
- Define how shared data such as products and stock is reconciled when multiple devices make changes.

Local network sync between devices in the same shop can be considered later. The first planned approach is local storage plus cloud sync when internet is available.

## Accounts, outlets, employees, and device pairing

Owner accounts and ownership records live on the backend. This allows an owner to sign in on a replacement phone and regain access to their account.

Employees should not need to create a full owner account. The planned pairing flow is:

1. The owner chooses an outlet, employee role, and permissions.
2. SUKA creates a short-lived QR pairing code.
3. The employee scans the code using the SUKA app.
4. The backend validates the pairing request.
5. The device is linked to the correct owner and outlet.
6. The employee receives only the access granted by the owner.

QR codes must use short-lived pairing tokens. They must not contain permanent passwords or unrestricted credentials.

Signing in on a new phone restores account access, but it does not magically recover unsynchronized data from a lost phone. Data recovery requires a previous cloud sync or a separate backup process.

## Plugin Store

SUKA will have a Plugin Store for features that are not part of the free POS core. Plugin subscriptions are monthly and belong to the owner account. If an owner has several outlets, one subscription covers those outlets.

### Planned plugins

| Plugin | What it does | Plan |
|---|---|---|
| **SUKA WA Assistant** | Connects one WhatsApp Business number chosen by the owner and adds AI-assisted customer replies and workflows. | Paid monthly |
| **SUKA Warehouse** | Adds advanced warehouse, multi-location stock, and stock transfer tools. | Paid monthly |
| **SUKA Calculator** | Adds business calculations such as cost, margin, and selling price. | Paid monthly |
| **SUKA Analytics** | Adds deeper sales and business analysis. | Paid monthly |
| **SUKA Member** | Adds customer membership and loyalty features. | Proposed |
| **SUKA Order** | Adds extra ordering workflows. | Proposed |

Plugin names and details may change as we design and test them.

### Plugin rules

- The core POS stays free.
- Plugins are subscribed to individually.
- A subscription is attached to `owner_id`, not `outlet_id`.
- One active subscription covers all outlets belonging to that owner.
- Employees can use a plugin only when their permissions allow it.
- Expired plugins can be disabled, but core POS data must remain available.
- The backend must verify subscription access. Hiding a button in the frontend is not enough.

Plugins need a controlled way to access POS features. Each plugin should declare its version, permissions, data access, and connectivity needs. Third-party plugin code must never receive unrestricted access to the POS database.

## SUKA WA Assistant

SUKA WA Assistant is the planned WhatsApp Business and AI assistant plugin.

The owner can choose which supported WhatsApp Business number to connect. The connection belongs to the owner account, not to a particular outlet.

Planned capabilities:

- AI auto-replies to customer messages.
- Answers based on the owner's approved product, price, and stock information.
- Helps customers ask questions and place orders.
- Sends order updates and digital receipts where supported.
- Routes conversations or orders to the right outlet using owner-defined rules.
- Lets the owner configure the assistant and decide which employees can manage it.

### WhatsApp connection rules

- One active WhatsApp connection/session per owner account in the initial plan.
- The owner chooses the number to connect.
- The same connection can serve all outlets belonging to that owner.
- The owner can disconnect the current number and connect a different supported number.
- The subscription is charged per owner account, not per outlet.

The exact integration method and available features must be confirmed during implementation. WhatsApp Business Platform, provider requirements, message charges, and Meta policies must be followed. Do not assume that every WhatsApp feature is available through every connection method.

## What the frontend needs to build

The frontend is responsible for the screens and the experience people use every day. It must also handle local data safely and make the online/offline state clear.

### Owner experience

- Register, sign in, and recover account access.
- First-time setup and store creation.
- Owner dashboard with sales summaries.
- Store and outlet selector.
- Outlet creation and settings.
- Employee list, invitations/pairing, roles, and permissions.
- QR code generation for employee device pairing.
- Device list and device revocation controls.
- Product, category, and basic stock management.
- Transaction history and sales reports.
- Sync status, pending uploads, and retry feedback.
- Plugin Store, plugin details, installation, and subscription status.
- SUKA WA Assistant connection settings and plugin controls.

### Cashier and employee experience

- QR pairing and device setup.
- Employee sign-in or device re-entry flow as defined by the authentication design.
- Cashier screen and product search.
- Cart, discounts if permitted, checkout, and receipt.
- Transaction history based on employee permissions.
- Clear offline and sync status.
- Friendly handling of unavailable server connections.
- Manager screens only when the assigned role allows them.

### Frontend foundations

- Responsive layouts for phone, tablet, and desktop where needed.
- Installable PWA experience and service worker.
- Local database setup using IndexedDB, with Dexie.js as the current proposed helper.
- Local-first transaction creation.
- Durable pending-sync queue and safe retry behavior.
- UI states for loading, empty data, errors, offline, and successful sync.
- Permission-aware navigation and actions.
- Plugin loading and lifecycle UI based on the approved plugin architecture.
- Accessible forms, validation, and useful error messages.

The frontend must never be the only place where access control is enforced. The backend must validate every protected operation too.

## What the backend needs to build

The backend is responsible for account security, shared business data, synchronization, subscriptions, and integrations. It must be the source of truth for ownership and permissions.

### Accounts and business structure

- Owner registration, login, logout, and account recovery.
- Secure password handling or the selected identity provider.
- Owner profile and account settings.
- Store and outlet APIs.
- Employee records and role/permission APIs.
- Device registration, pairing, and revocation.
- Short-lived, single-use QR pairing tokens.
- Ownership checks and tenant isolation on every relevant endpoint.

### POS and synchronization APIs

- Product, category, and stock endpoints.
- Transaction ingestion from local devices.
- Unique transaction ID validation and idempotency.
- Device and employee validation for each transaction.
- Sync acknowledgements so devices know what was safely received.
- Safe handling of retries and partial failures.
- Stock update rules and conflict handling across devices/outlets.
- Transaction history and report queries.
- Backup and restore services for data that has reached the cloud.

### Owner dashboard and real-time updates

- APIs for sales totals, transaction counts, and outlet summaries.
- Real-time event delivery through WebSocket or another suitable server-push method.
- Events sent only to the correct owner and authorized employees.
- Reconnection and missed-event recovery.
- Clear timestamps so the dashboard can distinguish current server data from unsynchronized local data.

### Plugin and subscription services

- Plugin catalog and plugin metadata.
- Plugin version and compatibility information.
- Installation and activation records.
- Owner-based monthly subscriptions.
- Payment status and renewal/expiration handling.
- Entitlement checks for every paid plugin API.
- Plugin permissions and audit logs.
- Safe behavior when a plugin expires or is cancelled.

### SUKA WA Assistant services

- Secure connection management for the WhatsApp number selected by the owner.
- One active WhatsApp session/connection per owner account in the initial plan.
- Webhook handling and message processing.
- AI request orchestration and usage controls.
- Retrieval of approved products, prices, stock, and outlet information.
- Owner-defined outlet routing rules.
- Message logs and delivery status where supported.
- Human handover and controls for sensitive actions.
- Secure storage of provider credentials and tokens.

### Backend foundations

- PostgreSQL schema and migrations.
- API validation and consistent error responses.
- Authentication and authorization middleware.
- Rate limiting and abuse protection.
- Structured logs and monitoring.
- Automated tests for tenant isolation, sync retries, duplicate prevention, and subscription enforcement.
- Backup, restore, and retention policies.

## Suggested technical direction

These are current proposals, not promises that the stack is already implemented.

| Area | Proposed technology or approach |
|---|---|
| Frontend | React / Next.js and TypeScript |
| UI | Responsive web app and installable PWA |
| Local storage | IndexedDB |
| Local database helper | Dexie.js |
| Offline support | Service Worker and durable sync queue |
| Backend | API service; framework to be confirmed |
| Cloud database | PostgreSQL |
| Real-time updates | WebSocket or another server-push method |
| Pairing | Short-lived QR token |
| Plugin system | Versioned manifests, permissions, and controlled APIs |

The exact stack can be adjusted after prototypes and technical testing.

## Security and data rules

- Data belonging to one owner must never be visible to another owner.
- Every protected API must check identity, ownership, role, and permission.
- Never put passwords, permanent access tokens, or provider secrets in QR codes.
- Keep provider credentials on the backend, not in frontend code.
- Use secure transport for network requests.
- Do not remove a local transaction just because an upload failed.
- Keep an audit trail for refunds, stock adjustments, permission changes, and other sensitive actions.
- Define backup and restore behavior before relying on cloud copies.
- Expiring a plugin must not delete core products, transactions, or other POS records.
- AI must use approved business data. Sensitive actions such as refunds or payment changes need explicit authorization and appropriate confirmation.

## Code contribution conventions

Keep code straightforward and easy to maintain.

- Use clear names for files, functions, variables, and components.
- Keep each change focused on the requested feature.
- Do not add code comments or comment-only columns/sections. Prefer clear names and simple structure so the code explains itself.
- Do not leave placeholder implementations, dead code, or unexplained temporary work in committed changes.
- Keep secrets and environment-specific values out of source control.
- Update this README when a product decision or architecture changes.
- Add or update tests when behavior changes.

If a comment is genuinely required to explain a non-obvious constraint, agree on it before adding it. The default is code without comments.

## Development roadmap

This is a working checklist. Mark an item complete only after it has been implemented and tested.

### Product and design

- [ ] Confirm core POS workflows and acceptance criteria.
- [ ] Finalize owner, store, outlet, employee, role, and device relationships.
- [ ] Define the plugin catalog and subscription rules.
- [ ] Define backup, restore, and data retention behavior.
- [ ] Design owner, cashier, manager, and plugin screens.

### Frontend

- [ ] Set up the application structure, routing, design system, and responsive layouts.
- [ ] Build owner registration, login, and account recovery screens.
- [ ] Build store and outlet management.
- [ ] Build employee management, role settings, and QR pairing screens.
- [ ] Build product, category, and basic stock screens.
- [ ] Build cashier, cart, checkout, and receipt flows.
- [ ] Build transaction history and basic reports.
- [ ] Add IndexedDB/Dexie local persistence.
- [ ] Add offline states and a durable transaction sync queue.
- [ ] Build owner dashboard and sync status views.
- [ ] Build Plugin Store and subscription screens.
- [ ] Build SUKA WA Assistant settings and connection screens.
- [ ] Add frontend tests for critical flows and offline behavior.

### Backend

- [ ] Set up the API service, PostgreSQL database, and migrations.
- [ ] Build owner authentication and account recovery.
- [ ] Build store, outlet, employee, role, and permission APIs.
- [ ] Build device registration, QR pairing, and revocation.
- [ ] Build product, category, and stock APIs.
- [ ] Build transaction sync endpoints with idempotency.
- [ ] Build sync acknowledgements and conflict handling.
- [ ] Build sales summary and report APIs.
- [ ] Add real-time owner dashboard events.
- [ ] Build plugin catalog and entitlement services.
- [ ] Build owner-based monthly subscription and payment handling.
- [ ] Build WhatsApp connection, webhook, and message services.
- [ ] Add backend tests for authorization, tenant isolation, sync, and subscriptions.
- [ ] Set up logs, monitoring, and backup procedures.

## Project decisions and progress log

Use this section to record meaningful decisions and verified progress. Add the newest entry first. Keep planned work separate from work that has actually shipped.

### 2026-11-17 — Product direction and initial architecture

**Decisions**
- SUKA is an offline-first POS platform with a free core and optional paid plugins.
- Core POS features include outlets, employees, products, transactions, basic stock, history, and basic summaries.
- Plugin subscriptions are monthly and belong to the owner account.
- One owner subscription applies to all outlets owned by that account.
- Transactions are saved locally first and sent to the backend when internet is available.
- A remote owner sees a sale after it reaches the backend.
- Employee devices are intended to connect through an owner-generated QR pairing flow.
- SUKA WA Assistant is the working name for the WhatsApp Business AI plugin.
- The initial plan allows one active WhatsApp connection per owner account. The owner chooses which supported number to connect.
- The connected WhatsApp number can serve all of the owner's outlets, with outlet-routing rules.

**Planning**
- Frontend and backend responsibilities have been outlined.
- The development checklist has been added.
- Technical choices still need to be validated through implementation.

**Completed**
- Initial product README and decision log.

## Keeping this README current

This file should change along with the project.

Whenever a feature is started, completed, or changed:

1. Update the relevant section above.
2. Update the roadmap checkboxes only when the work is verified.
3. Add a dated note to the top of the progress log.
4. Explain important changes to earlier decisions instead of silently replacing them.
5. Keep detailed implementation notes in separate documents when this README becomes too large.

## Names

- **Project:** SUKA Project
- **Brand:** SUKA
- **Core product:** SUKA POS
- **WhatsApp plugin:** SUKA WA Assistant

## License

A license has not been selected yet. Until one is added, do not assume the repository grants permission to reuse, modify, or distribute its contents.
