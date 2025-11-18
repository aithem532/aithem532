# ERAM Product Blueprint

## 1. Product Vision — Expanded
- Luxury bilingual (Arabic/English) digital gift card issuance and sales platform that mirrors Floward-level craftsmanship and branded elegance, covering gift cards, curated add-on products (roses, chocolates, accessories) and agent payment networks.
- Enables online/offline selling of stored-value cards plus complementary products with unified invoices that adopt IFRS-ready invoice fields, bilingual headers, QR codes, tax registration references, and sequential numbering.
- Cards remain inactive through the sales stage, only becoming spendable after recipient details and delivery logistics are confirmed, ensuring regulatory compliance and fraud mitigation.
- Agents and POS branches accept ERAM cards via secure portals, updating their wallet balances instantly while reflecting pending settlements and operational statuses.
- Finance teams execute settlement cycles, daily close procedures, and accounting reconciliations with IFRS-minded double-entry abstractions and granular audit trails.
- Full RTL and LTR support with an interface inspired by Floward’s luxury aesthetic, including typography, pastel gradients, and premium iconography with company branding guidelines (logo placement, color palette, watermark usage).

## 2. Primary Objectives
1. Deliver seamless luxury gift card sales that bundle card tiers with optional physical products, ensuring accurate pricing, taxes, and multilingual presentation.
2. Implement a multi-step activation workflow (Sale → Invoice issuance → Activation upon recipient completion) with clear statuses and audit logs.
3. Maintain precise financial operations that cover top-ups, debits, refunds, freezes, and delivery fees using a single operations ledger.
4. Provide full agent/POS lifecycle management (registration, activation, wallet balance, commissions, branch tracking).
5. Enable customer purchase flow at agents: card validation, balance check, debiting, agent wallet credit, and receipt issuance.
6. Support settlement confirmation workflow with finance approvals, transfer references, and reconciliation checkpoints.
7. Generate robust accounting reports (CSV/XLSX) with strict formatting: bilingual headers, frozen rows, numeric precision, and audit metadata.
8. Guarantee safe, logged, idempotent operations with tamper-proof audit trails and retriable workflows.
9. Offer Floward-level UI/UX with clear component hierarchy, luxury typography, card/product galleries, and responsive layouts for desktop/tablet.
10. Provide multi-language support driven by external localization files with RTL mirroring and dynamic text expansion handling.
11. Deliver comprehensive admin dashboards, cashier screens, finance consoles, auditor read-only views, and operational alerts.

## 3. User Roles & Permissions Model
- **Super Admin**: Full platform control, configuration of card tiers, products, agents, financial rules, access delegation, and viewing all reports/logs.
- **Admin**: Manages daily operations—sales processing, activation approvals, card lifecycle changes, product catalog updates, moderate access to finance reports.
- **Finance**: Oversees monetary operations—approves settlements, performs refunds, executes top-ups, verifies agent wallets, runs accounting reports, closes days.
- **Auditor**: Read-only access to financial operations, reports, ledger entries, invoices, and system configuration snapshots for compliance reviews.
- **Agent**: Access limited to their cards/purchases, wallet balance, settlement requests, POS management, and customer payment processing.
- **Cashier**: Executes frontline sales, card issuance, product bundling, and interacts with activation forms; limited to operational modules.
- **Delivery Staff (optional)**: View delivery queue, update delivery status, capture proof-of-delivery metadata, no financial access.

## 4. Core Entities — Conceptual Data Model
### Card Tiers
- **tier_code**: Unique identifier (string).
- **default_value**: Stored value amount preloaded for tier.
- **price_to_customer**: Retail price inclusive of markup and fees.
- **color**: Hex code / palette reference for UI and printed materials.
- **is_active**: Boolean to control availability.

### Products
- **product_id** (UUID/string), **name**, **cost (COGS)**, **price**, **category**, **is_active**, **created_at**.

### Sales
- **invoice_number** (sequential, IFRS compliant), **tier_code**, **product_list** (array of product_id + quantity + unit price), **giver_name**, **giver_phone**, **initial_card_status** (CREATED), **total_amount**, **tax fields** (VAT %, amount), **created_at**.

### Cards
- **username** (issuer/cashier), **serial_number / last4**, **tier_code**, **initial_balance**, **balance**, **currency**, **recipient_name**, **recipient_phone**, **delivery_date**, **delivery_location**, **delivery_fee**, **status** (CREATED/ACTIVE/DELIVERED/FROZEN/EXPIRED), **notes**.

### Operations
Operation types: SALE, TOPUP, DEBIT, REFUND, PURCHASE, PURCHASE_CONFIRMATION, AGENT_RECEIPT, SETTLEMENT, ACTIVATION, FREEZE, UNFREEZE.
- **op_id**, **operation_type**, **related_card_id**, **related_sale_id**, **agent_id/pos_id** (if applicable), **amount**, **currency**, **status** (PENDING/POSTED/FAILED), **context_data** (JSON with details), **created_at**, **posted_at**, **performed_by**.

### Agents
- **agent_code**, **shop_name**, **owner_name**, **phone**, **bank_name / wallet_name**, **account_number**, **location**, **wallet_balance**, **branches** (relation to POS).

### POS (Branches)
- **pos_code**, **agent_code**, **display_name**, **location**, **is_active**, **device_id** (optional), **contact_person**.

### Customer Purchases
- **purchase_id**, **card_id**, **agent_id**, **pos_id**, **amount**, **notes**, **status** (INITIATED/APPROVED/DECLINED/SETTLED), **created_at**, **receipts**.

### Daily Close
- **close_date**, **totals_breakdown** (operations aggregated by type, currency), **cashier_id**, **created_at**, **approval_status**, **attachments**.

## 5. End-to-End Flows
### 5.1 Sales Flow
1. Cashier logs in via secure MFA, selects **New Sale** from dashboard, and the system logs the initiation in the audit trail.
2. Card tier catalogue appears with tier previews (color, stored value, retail price, benefits, availability count); cashier selects one, triggering validation for stock and tier status.
3. Add-on product panel allows browsing by category; cashier adds products with inline quantity selector, each addition recalculating subtotal, VAT, delivery fees, and projected gross margin; availability flags warn when inventory is low.
4. Giver information screen enforces bilingual name entry (Arabic + English fields where applicable), validated phone (country-specific regex), optional message, and preferred communication channel; consent checkbox captured for notifications.
5. Payment method selection (cash, POS, online) with deposit instructions; system calculates invoice totals (card price + products + taxes + fees), displays summary card, and requires cashier confirmation.
6. Invoice generation: IFRS-compliant sequential invoice_number assigned, bilingual PDF rendered with QR code linking to verification endpoint, tax registration numbers, address, and digital signature; invoice stored and emailed/SMS based on giver preference.
7. Card record instantiated (status CRETAED/INACTIVE) capturing tier_code, initial_balance, link to sale, and placeholder recipient info; serial number reserved with duplicate check.
8. Operations ledger writes SALE operation (status PENDING) referencing invoice, user, payment method, and context data (product_list, taxes) while simultaneously logging inventory decrement for physical products.
9. Payment confirmation view displays invoice, print/download/share buttons, activation instructions, multi-language summary, and "Proceed to Activation" CTA; system records completion timestamp and user.

### 5.2 Card Activation Flow
1. Admin or cashier with activation rights accesses **Pending Activations** queue filtered by sale date, cashier, tier; selecting a card loads contextual data (invoice, giver message, card mockup).
2. Recipient details form enforces required fields: name (Arabic/English), mobile (validated), email (optional), delivery date/time slot with timezone, delivery location (map pin + formatted address), delivery instructions, and delivery fee entry with configured caps.
3. Optional "Recipient Representative" section for alternative receiver; attachments (e.g., ID, delivery notes) can be uploaded and stored.
4. System validates: delivery date not in past, phone format, fee within allowed range, location coverage; warnings displayed inline with bilingual error hints.
5. Upon submission, workflow engine calculates final card balance (initial_balance - delivery_fee if company deducts) and updates card status to ACTIVE; any delivery fee recorded as product revenue or service fee per configuration.
6. Ledger posts ACTIVATION (status POSTED) referencing card, sale, and delivery metadata; SALE operation transitions to POSTED, and unearned revenue reclassified to card liability accounts.
7. Recipient confirmation package generated (email/SMS/WhatsApp) with card preview, delivery schedule, and instructions; giver receives notification that activation completed.

### 5.3 Customer Payment at Agent
1. Agent authenticates via POS device with PIN/MFA; session logs capture device/location.
2. Customer presents card (digital or physical); agent scans QR or types serial/last4; system retrieves card details (status, balance, expiry, freeze flags) in real time.
3. Agent enters purchase amount and optional order reference; system validates amount ≤ balance, checks card status ACTIVE, verifies merchant category if restrictions exist.
4. Customer confirms transaction (signature/PIN optional); system calculates post-purchase balance, displays bilingual confirmation screen for agent/customer review.
5. Ledger posts PURCHASE operation (card liability debit) and AGENT_RECEIPT operation (agent wallet credit) atomically; context_data stores POS, geo coordinates, cashier ID, product notes.
6. Customer receives receipt via SMS/email plus printable slip; agent dashboard updates wallet balance and purchase history instantly.
7. Purchases flagged as "Awaiting Settlement" until finance clears them; automated alerts generated if wallet exceeds threshold.

### 5.4 Settlement Flow
1. Finance team accesses **Settlements** module showing agent list with wallet balances, pending aging buckets, last settlement date, and alerts for overdue settlements.
2. Selecting an agent reveals purchase breakdown, supporting receipts, disputes flags, and ability to exclude disputed transactions; finance inputs settlement amount (full/partial) and payment method (bank transfer, instant payment, cash).
3. System validates amount ≤ wallet balance, requires reference number, settlement date, and uploads (bank confirmation/PDF); multi-approver workflow (maker-checker) ensures segregation of duties.
4. Upon approval, ledger posts SETTLEMENT operation (debit agent wallet liability, credit cash/bank) and optionally PURCHASE_CONFIRMATION entries if required; wallet balance updates, settlement statement generated (PDF/CSV) with bilingual summary.
5. Agent portal displays settlement confirmation, breakdown of purchases included, transfer reference, and ability to acknowledge receipt; audit log captures acknowledgement timestamp.

### 5.5 Freeze / Unfreeze Card
1. Authorized admin accesses card detail page, clicks **Freeze Card**, selects standardized reason codes (fraud suspicion, lost card, chargeback, compliance) and enters supporting notes/attachments.
2. System prompts for MFA confirmation, creates FREEZE operation (status POSTED), changes card status to FROZEN, pushes notification to agents so terminals reject transactions, and informs giver/recipient.
3. Audit trail records actor, timestamp, and IP; card timeline displays freeze event.
4. To unfreeze, authorized role selects **Unfreeze**, verifies issue resolution, enters notes, system posts UNFREEZE operation, reverts card to prior status (ACTIVE) and sends confirmations.

### 5.6 Refunds
1. Finance searches sale/card via invoice number or serial, reviews transaction history, outstanding balance, and any deliveries/products already fulfilled.
2. Eligibility rules check: card not fully used, within refund window, no active disputes; finance selects refund reason (customer request, delivery failure, regulatory order).
3. Refund calculator suggests amounts (remaining balance, product refunds, delivery fee adjustments) with option to override within policy limits; supporting documents uploaded.
4. Upon approval, ledger posts REFUND operation (debit card liability/unearned revenue, credit cash/bank or payment gateway); card status updates to REFUNDED or FROZEN depending on policy.
5. Credit note generated with bilingual layout referencing original invoice, VAT adjustments, and new totals; notifications sent to giver and finance.

### 5.7 Searching & Filtering
- Omni-search bar supports card serial, invoice number, customer name, agent code, purchase ID; autocomplete provides context chips.
- Advanced filters per module (sales, cards, operations, reports) with multi-select dropdowns for status, tier, product, agent, POS, date range, currency, amount range, language, delivery status.
- Results table updates in real time, includes column picker, saved views, and export buttons; preview side panel shows KPIs, timeline, and related entities without leaving page.

### 5.8 Daily Close
1. Cashier initiates **Daily Close**; system pre-fills list of operations performed that day (sales, activations, refunds, cash intake) tied to that user and POS location.
2. Cashier counts physical cash/product inventory, enters closing balances, uploads photos/scans of cashbox reports; system compares expected vs actual and highlights discrepancies.
3. Daily Close record captures totals by payment method, card tier, product category, agent-related actions, and any manual adjustments requiring justification notes.
4. Finance supervisor reviews pending closes, approves or rejects with comments; once approved, record locks, operations flagged as reconciled, and daily close report exported to archive.

## 6. Accounting Requirements
- Each operation posts double-entry conceptual records: e.g., SALE (Cash/Receivable vs. Unearned Revenue), ACTIVATION (Unearned Revenue → Card Liability), PURCHASE (Card Liability → Agent Payable), SETTLEMENT (Agent Payable → Bank Cash).
- Card balances are liabilities until purchases occur; activation moves value from Unearned to Card Liability.
- Products record revenue and cost separately to calculate gross margin; COGS recognized upon delivery.
- Agent wallets represent payable liabilities until settled; ledger entries tie to agent statements.
- Settlement operations clear agent liabilities and attach payment proofs; reconciliation dashboards highlight outstanding amounts.
- Ledger view includes filters by account, operation type, currency, and IFRS tag; auditors can export entire ledger with hashes.

## 7. Reporting Module
- **Daily Sales Report**: lists sales by tier, products, languages; includes invoice numbers, giver info, payment method; export CSV/XLSX with auto column widths, frozen header row, wrapped text, bilingual headers, BOM for CSV, numeric precision 2 decimals.
- **Card Activity Report**: status changes, activations, freezes, balances; includes timestamps, user IDs, location data.
- **Agent Settlement Report**: wallet balances, pending purchases, settlement history, transfer references, attachments.
- **Purchase Logs**: all customer purchases with agent/POS, amounts, receipts, status; supports dispute resolution.
- **Product Sales Analysis**: quantity sold, revenue, margin, attachment to card tiers, seasonal trends.
- **Outstanding Liabilities**: aggregated card liabilities, agent wallets, unearned revenue, aging buckets (0-30/31-60/61-90/>90 days).
- **Gift Card Balance Aging**: card-level detail showing issued date, activation date, remaining balance, expiration timeline.
- Export formats: CSV (UTF-8 with BOM), XLSX; include bilingual column headers, date/time formatting 12h/24h toggle, currency symbols, thousands separators, consistent decimal precision.

## 8. UX & UI Blueprint — Floward-Style
- **Theme**: Warm neutral palette (cream, blush, gold accents), luxury serif headings, modern sans body; high-end photography backgrounds with subtle gradients.
- **Layout Grid**: 12-column responsive grid, 1440px desktop base, 24px gutters, safe margins for RTL mirroring.
- **Components**: Glassmorphism cards, soft shadows, pill buttons with gradient fills, micro-interactions for hover/focus states.
- **Navigation**: Left sidebar for desktop with icon+text; top breadcrumbs; quick actions panel; mobile collapsible drawer.
- **Forms**: Multi-step modals for sales/activation with progress bar; bilingual labels; floating field descriptions; inline validation.
- **Buttons/Colors**: Primary (#C89B7B), Secondary (#1B1B1F), Success (#2E8B57), Warning (#D97706), Danger (#B91C1C). Buttons have rounded corners, subtle gradients, golden borders.
- **Invoice Design**: Bilingual header, logo watermark, QR code, gold divider lines, structured sections for seller, buyer, itemized table, totals, signature block.
- **Card Design**: Digital card mockups with metallic sheen, tier color-coded backgrounds, embossed serial digits, bilingual tagline.
- **Product Display**: Carousel of curated items with high-res images, price tags, stock indicators, quick add buttons.
- **Language Toggle**: Top-right toggle between AR/EN; full RTL mirroring, icons flipping, text alignment adjustments.

## 9. Error Messages & Validation Rules
- Missing field: "Please complete all required fields." / Arabic equivalent.
- Invalid amount: "Enter a valid amount within permitted limits."
- Card frozen: "This card is temporarily frozen. Please contact support."
- Wallet insufficient: "Agent wallet balance is insufficient for this transaction."
- Duplicate serial: "Card serial already exists. Use a unique serial number."
- Invalid phone: "Phone number format is invalid."
- Required product quantity: "Specify quantity for each selected product."
- Delivery info rules: "Delivery date, time, and location are mandatory before activation."

## 10. Security & Compliance
- API keys for third-party integrations stored encrypted; rotate regularly.
- PINs (if used) hashed with modern algorithms (Argon2/bcrypt) and salted; no plaintext storage.
- Rate limiting per user/IP to prevent brute force and abuse.
- Request logging with correlation IDs, time stamps, actor IDs, and geolocation data for audits.
- Operation idempotency via unique tokens to avoid duplicate postings.
- Data protection: at-rest encryption, TLS in transit, field-level masking for sensitive info.
- Comprehensive audit log capturing before/after values, user, timestamp, IP/device; tamper-evident storage.

## 11. Non-Functional Requirements
- **Performance**: <2s response for dashboards; bulk exports ≤30s for up to 50k records.
- **Scalability**: Modular services for sales, ledger, reporting; horizontal scaling via containers.
- **Reliability**: 99.9% uptime target; redundancy for DB and file storage.
- **Observability**: Centralized logging, metrics (APM), alerting on error thresholds, tracing for critical flows.
- **Extensibility**: Plugin-style product catalog, operation types, and report templates.
- **Data Retention**: Financial data retained 10 years; archives accessible for auditors.
- **Backup Strategy**: Hourly incremental, daily full backups stored in multi-region secure storage; quarterly restore drills.
- **Environments**: Dev, Staging, Prod with isolated credentials, sanitized data in lower envs.

## 12. User Stories
1. **Sales Flow**: As a cashier, I can create a new sale selecting tier and products, so that I issue accurate invoices. *Acceptance*: Invoice generated with bilingual fields, sale recorded in ledger, card status CREATED.
2. **Activation Flow**: As an admin, I can enter recipient data to activate a card, ensuring no cards are usable without delivery details. *Acceptance*: Card status ACTIVE, activation operation logged.
3. **Product Catalog**: As an admin, I can manage products with price and cost, so add-ons stay current. *Acceptance*: Create/edit/deactivate products, updates reflected immediately in sales screen.
4. **Agent Onboarding**: As a super admin, I can register an agent with branches, so they accept cards. *Acceptance*: Agent status ACTIVE, POS codes generated, wallets initialized.
5. **Customer Purchase**: As an agent, I can accept a card payment with balance check, so customers can spend. *Acceptance*: Purchase and agent receipt operations recorded, card balance reduced.
6. **Wallet Settlement**: As finance staff, I can settle agent wallets, so liabilities are cleared. *Acceptance*: Settlement operation logged, wallet reduced, statements updated.
7. **Refund Processing**: As finance, I can refund unused card balance, so customers are reimbursed. *Acceptance*: Refund invoice/credit note issued, ledger entries posted.
8. **Freeze Control**: As admin, I can freeze/unfreeze cards, preventing fraudulent use. *Acceptance*: Card status updates, operations logged, attempts blocked.
9. **Daily Close**: As cashier, I can perform daily close summarizing operations. *Acceptance*: Daily Close record locked post approval.
10. **Reporting Export**: As finance, I can export reports (CSV/XLSX) with strict formatting. *Acceptance*: File downloads with bilingual headers, frozen row, correct precision.
11. **Localization**: As user, I can switch between Arabic and English. *Acceptance*: UI text toggles, layout mirrors, date/time format updates.
12. **Auditor Access**: As auditor, I can view read-only operations and reports. *Acceptance*: No edit rights, complete history accessible.
13. **Delivery Staff**: As delivery staff, I can view assignments and confirm deliveries. *Acceptance*: Delivery status updates, proof stored.
14. **UI Luxury Standard**: As brand manager, I can verify UI adheres to Floward-level aesthetics. *Acceptance*: Components follow design system, QA checklist passed.

## 13. Risks & Mitigation
- **Concurrency Issues**: Use optimistic locking and idempotent tokens for operations, queue-based processing for high contention areas.
- **Card Misuse**: Implement card status checks, transaction limits, freeze controls, and anomaly detection alerts.
- **Agent Disputes**: Maintain detailed purchase logs, signed receipts, and settlement references; provide dispute workflow with attachments.
- **Data Corruption**: Enforce database constraints, referential integrity, and regular backups with restore testing.
- **Timezone Confusion**: Store UTC timestamps with locale-aware display; include timezone indicators on invoices/reports.
- **Invoice Mismatches**: Sequential numbering with checksums, duplicate detection, and locked templates.
- **Report Inconsistencies**: Recompute from source ledger, schedule data validation jobs, reconciliation dashboards.

## 14. Glossary
- **Activation**: Process that makes a sold card usable after recipient info is confirmed.
- **Agent Wallet**: Virtual balance representing amounts owed to agents for customer purchases.
- **Daily Close**: End-of-day reconciliation summarizing operations for a cashier/location.
- **Ledger Operation**: Atomic financial event recorded with double-entry context.
- **POS**: Point-of-sale branch/device where agents process transactions.
- **Sale Invoice**: Official document produced when selling a card/products, IFRS compliant.
- **Settlement**: Finance-approved transfer clearing amounts owed to agents.
- **Card Tier**: Predefined stored-value level with branding, pricing, and design attributes.
- **Product Add-on**: Physical or digital item sold alongside cards (roses, chocolates, etc.).
- **Freeze**: Status preventing further card transactions until issue resolved.
