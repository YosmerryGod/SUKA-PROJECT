# SUKA Project

**SUKA Project** is building a modular, offline-first Point of Sale (POS) platform for businesses in Indonesia. The core POS is designed to remain free, while optional paid plugins add advanced capabilities such as WhatsApp Business AI assistance, warehouse management, and business analytics.

> **Project status:** Planning and architecture definition. Features described as planned or proposed are not yet claims of implementation.

## Vision

Make practical business tools accessible to businesses of different sizes: start with a free POS, then let each owner add only the capabilities their business needs.

SUKA is intended to grow beyond a cashier application into a modular business platform with a plugin ecosystem.

## Product Principles

- **Offline-first:** Core cashier operations should continue when the internet is unavailable.
- **Local-first transactions:** Save a transaction to the device before attempting cloud synchronization.
- **Cloud when available:** Send new transactions to the backend when a server connection is available.
- **Free core:** Essential POS operations remain free.
- **Optional paid plugins:** Advanced features are subscribed to separately.
- **Owner-based subscriptions:** A plugin subscription belongs to the owner account and applies across that owner's outlets.
- **Modular and extensible:** Features can be added through a controlled plugin system.
- **Data safety:** Losing connectivity or allowing a plugin subscription to expire must not erase core POS records.

## Core POS — Free

The current product direction is to keep the core POS free, including:

- Owner registration and login.
- Store and outlet management.
- Employee management.
- Role and permission management.
- Product and category management.
- Cart and checkout.
- Sales transactions.
- Stock management basics.
- Transaction history.
- Cashier settings.
- Basic sales summaries and local reports.
- Offline transaction storage.
- Cloud synchronization when internet is available.

The intended free model does not charge per outlet, employee, device, product, or basic transaction. Usage limits and infrastructure policies, if needed later, must be documented transparently rather than silently changing the free-core promise.

## Offline-First and Cloud Synchronization

The POS should not depend on an internet connection to complete a sale.

### When the device is online

1. The cashier creates a transaction.
2. The application saves it to the local database first.
3. The application sends the transaction to the backend.
4. The backend validates and stores it in the cloud database.
5. The owner's dashboard can show the transaction after the server accepts it.

### When the device is offline

1. The cashier creates a transaction.
2. The application saves it locally.
3. The transaction is marked as waiting for synchronization.
4. Cashier operations continue without waiting for the server.
5. When connectivity returns, the application retries pending transactions.
6. The owner can see the transaction remotely after it has reached the backend.

### Synchronization requirements

- Every transaction must have a stable unique ID.
- Transactions must be safe to retry without creating duplicates (idempotent processing).
- Local records must not be deleted just because an upload fails.
- The application should expose clear synchronization states, such as `pending`, `syncing`, `synced`, and `failed`.
- Retry logic must handle temporary network failures and server errors.
- The server must validate the owner, store, outlet, device, and transaction permissions.
- Conflict handling is required for shared data such as product details and stock.

**Real-time visibility:** When a cashier device has internet access and its transaction is accepted by the backend, the owner can monitor it remotely. If the cashier is offline, a remote owner cannot receive that new transaction until connectivity returns and synchronization succeeds.

Local-network synchronization between devices in the same shop may be explored as an additional capability. It is not required for the initial cloud-sync model and should not be presented as available until implemented and tested.

## Accounts, Ownership, and Devices

The backend is intended to manage account identity and ownership so an owner can sign in on a replacement phone.

Planned account responsibilities:

- Owner registration and authentication.
- Store and outlet ownership.
- Employee and device association.
- Device pairing and revocation.
- Subscription entitlements.
- Backup and restore coordination.

A QR pairing flow is planned for connecting an employee's device to an owner's store:

1. The owner creates a time-limited pairing QR code.
2. The employee scans it from the SUKA application.
3. The backend validates the pairing token.
4. The device is associated with the correct owner, store, outlet, role, and permissions.
5. The employee sees only the features permitted by the owner.

QR codes should contain short-lived pairing credentials, not permanent passwords or unrestricted access tokens.

Account recovery and transaction recovery are separate concerns. Signing in on a new phone restores account access, but locally stored transactions on a lost device can only be recovered if they were previously synchronized or backed up.

## Plugin Ecosystem

SUKA is planned to have a Plugin Store. Owners can install optional modules based on their operational needs.

### Proposed plugins

| Plugin | Purpose | Monetization |
|---|---|---|
| **SUKA WA Assistant** | Connect a WhatsApp Business number and provide AI-assisted customer replies and workflows. | Paid monthly subscription |
| **SUKA Warehouse** | Advanced warehouse, multi-location inventory, and stock transfer workflows. | Paid monthly subscription |
| **SUKA Calculator** | Business calculators such as cost, margin, and pricing tools. | Paid monthly subscription |
| **SUKA Analytics** | Advanced business and sales analysis. | Paid monthly subscription |
| **SUKA Member** | Customer membership and loyalty features. | Proposed plugin |
| **SUKA Order** | Additional ordering workflows. | Proposed plugin |

Plugin names and feature boundaries may evolve during development.

### Plugin subscription rules

- Core POS remains free.
- Plugins are subscribed to individually.
- A subscription is attached to the **owner account**, not to an individual outlet.
- One owner's active plugin subscription applies to all outlets owned by that account.
- Employees can use plugin features only when the owner grants the required permissions.
- Expiring or cancelling a plugin must not delete the owner's core POS data.
- Premium-only actions must be checked by the backend, not only hidden in the user interface.

### Plugin architecture and security

The plugin system should use a controlled manifest and permission model. Plugins must declare their capabilities and required permissions. Avoid executing arbitrary third-party code with unrestricted access to POS data.

The platform should define:

- Plugin identity, version, compatibility, and publisher.
- Required permissions and data access.
- Installation, activation, update, disable, and uninstall lifecycle.
- Subscription entitlement checks.
- Secure APIs between plugins and the POS core.
- Audit logs for sensitive actions.
- Data export and retention behavior when a plugin is disabled.

Local plugins may provide offline functionality. Cloud plugins may require internet access. Each plugin must clearly state its connectivity requirements.

## SUKA WA Assistant

**SUKA WA Assistant** is the planned WhatsApp Business integration plugin.

Intended capabilities include:

- Connect a WhatsApp Business account/number selected by the owner.
- AI-assisted automatic replies to customer messages.
- Answer questions using the owner's approved product, price, and availability data.
- Help customers with ordering workflows.
- Send order updates and digital receipts where supported.
- Route a customer conversation to the relevant outlet when the business has multiple outlets.
- Allow the owner to configure the assistant's behavior and employee access.

### WhatsApp connection policy

- Maximum **one active WhatsApp connection/session per owner account** under the initial subscription design.
- The owner chooses which supported WhatsApp Business number to connect.
- The connection is associated with the owner account, not a specific outlet.
- The same connected number can serve the owner's outlets, with outlet selection/routing rules configured by the owner.
- The owner can disconnect the current number and connect another supported number.

The exact connection method, supported WhatsApp capabilities, provider requirements, message charges, and Meta policy compliance must be confirmed during implementation. Do not assume that every WhatsApp feature is available through every integration method.

## Proposed Technical Direction

The following is an initial architecture direction, not a finalized or implemented technology commitment.

| Area | Proposed direction |
|---|---|
| POS application | Installable web application / PWA, with Android packaging evaluated |
| Local database | IndexedDB |
| Local database helper | Dexie.js |
| Backend | API service for authentication, ownership, synchronization, plugins, and subscriptions |
| Cloud database | PostgreSQL |
| Real-time dashboard updates | WebSocket or another server-push mechanism |
| Offline synchronization | Durable local outbox, retry queue, and idempotent server endpoints |
| Pairing | Short-lived QR pairing token |
| Plugin platform | Versioned manifests, permissions, and controlled plugin APIs |

Technology choices may change after a technical prototype and testing.

## Important Data and Security Requirements

- Separate owner identity, store, outlet, employee, device, and subscription records.
- Enforce tenant isolation: one owner must never access another owner's data.
- Validate permissions on the backend for every sensitive operation.
- Protect credentials and tokens; never place secrets in QR codes or client-side source code.
- Use secure transport for network communication.
- Make transaction synchronization idempotent.
- Keep an audit trail for sensitive actions such as refunds, stock adjustments, and permission changes.
- Define backup, restore, retention, and export procedures.
- Do not delete core business records when a plugin subscription ends.
- Treat AI-generated replies as suggestions or controlled actions; require confirmation for sensitive actions such as refunds, payment changes, or final order commitments where appropriate.

## Development Roadmap

The roadmap is a planning document and will change as implementation progresses.

- [ ] Define product requirements and core POS workflows.
- [ ] Finalize owner, store, outlet, employee, role, and device data models.
- [ ] Design owner registration and authentication.
- [ ] Design QR-based employee device pairing and revocation.
- [ ] Prototype local database and offline transaction flow.
- [ ] Implement durable transaction outbox and retry synchronization.
- [ ] Implement backend transaction ingestion and duplicate protection.
- [ ] Build owner sales dashboard and server-driven updates.
- [ ] Define backup and restore behavior for owner device replacement.
- [ ] Implement plugin manifest, permissions, and lifecycle.
- [ ] Implement owner-based plugin subscriptions and entitlement checks.
- [ ] Design and implement SUKA WA Assistant.
- [ ] Validate multi-outlet product, stock, and order routing.
- [ ] Add automated tests for offline, reconnect, retry, and data isolation scenarios.

Completed items should only be checked after implementation and verification.

## Project Status and Decisions

This section records the current product decisions so future development stays consistent.

### Product and pricing

- The core POS is intended to be free.
- Owners can add outlets and employees without a per-outlet or per-employee subscription.
- Basic products, categories, transactions, stock, transaction history, and sales summaries belong to the free core.
- Optional advanced capabilities are monetized as monthly plugins.
- Plugin subscriptions are owned by the owner account and apply to all of that owner's outlets.

### Offline and online behavior

- Local storage is the first write target for POS transactions.
- When online, the application sends transactions to the backend so the owner can monitor sales remotely.
- When offline, transactions remain stored locally and are queued for synchronization.
- Pending transactions are sent when connectivity returns.
- Remote real-time visibility is only possible after a transaction reaches the backend.
- Local-network synchronization is a possible future enhancement, not an assumed initial feature.

### Accounts and employee access

- Owner account identity and store ownership are stored in the backend.
- Employees connect to an owner's store through an owner-generated QR pairing flow.
- Roles and permissions determine which menus and actions an employee can access.
- Owner login on a replacement device restores account access; unsynchronized local data requires a separate backup or recovery path.

### Plugin and WhatsApp rules

- SUKA WA Assistant is the selected working name for the WhatsApp Business + AI assistant plugin.
- The initial plan allows one active WhatsApp connection/session per owner account.
- The owner chooses which supported WhatsApp Business number to connect.
- That connection can serve all outlets belonging to the owner, subject to outlet-routing configuration.
- The plugin is billed monthly per owner account, not per outlet.

## Development Update Log

Add a dated entry here whenever a meaningful product decision, architecture change, feature implementation, or verified milestone occurs. Keep the newest entry at the top. Distinguish clearly between **Decision**, **In progress**, and **Completed** so planned work is not mistaken for shipped functionality.

### 2026-11-17 — Initial product definition

**Decision**
- Established SUKA Project as a modular, offline-first POS platform.
- Defined free core POS and individually paid monthly plugins.
- Defined owner-account-based plugin subscriptions shared across all owned outlets.
- Defined local-first transaction storage with cloud synchronization when online.
- Defined remote owner monitoring after transactions reach the backend.
- Defined QR-based employee device pairing as the intended connection flow.
- Selected SUKA WA Assistant as the working name for the WhatsApp Business + AI plugin.
- Set the initial WhatsApp connection limit to one active session per owner, with the owner choosing the connected number.

**In progress**
- Product requirements, technical architecture, data model, and implementation roadmap.

**Completed**
- Initial project README and product decision record.

---

## How to Keep This README Updated

For every meaningful development session:

1. Update the relevant product, architecture, plugin, security, or roadmap section.
2. Add a dated entry to the top of the Development Update Log.
3. Mark a roadmap item completed only after the feature is implemented and verified.
4. Record breaking changes and explain why the decision changed.
5. Keep this README as the current high-level source of truth; detailed technical specifications can live in separate documentation files as the project grows.

## Naming

- **Project:** SUKA Project
- **Platform / brand:** SUKA
- **Core product:** SUKA POS
- **WhatsApp plugin:** SUKA WA Assistant

## License

No license has been specified yet. Until a license is added, the repository should not be assumed to grant permission to reuse, modify, or distribute its contents.
