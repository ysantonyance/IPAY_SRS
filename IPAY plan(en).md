# Software Requirements Specification

## Amazon Clone (IPAY) — An E-Commerce Platform

| | |
|---|---|
| **Document Version** | 1.1 |
| **Status** | Draft for Review (synchronized with the current codebase) |
| **Date** | 18th of October 2026 |
| **Author** | IPAY (Incredible Project Aspnet Yes) |
| **Approved by** | Олександр Загоруйко |
| **Classification** | Internal / Academic |
| **Standard** | IEEE 830-1998 / ISO/IEC/IEEE 29148 |

### Change History

| Version | Date | Author | Description of the changes |
|---|---|---|---|
| 1.0 | 13.08.2026 | IPAY | Initial version of the SRS: functional and non-functional requirements, Use Cases, risks, metrics |
| 1.1 | 18.10.2026 | IPAY | Synchronization with the actual implementation (`IPAY_backend`, `ipay_client` repositories): real stack (.NET 10, Firestore, Firebase Auth, Cloudinary), Moderator/Seller roles, 2FA, refresh tokens, seller requests, video catalog, three interface languages; an "Implementation status" marker added to every requirement; Use Cases, risks, metrics and traceability matrix updated. IDs FR-01…FR-20 are preserved, new requirements are FR-21…FR-30 |

### Implementation Status Legend

| Marker | Meaning |
|---|---|
| ✅ | Implemented |
| 🟡 | Partially implemented (differences described in the note) |
| ❌ | Not implemented (postponed or removed from scope) |
| ➕ | New requirement that was not in version 1.0 |

---

## Contents

1. [Introduction](#1-introduction)
2. [General Description](#2-general-description)
3. [Project Scope](#3-project-scope)
4. [Functional Requirements](#4-functional-requirements)
5. [Use Cases](#5-use-cases)
6. [Non-Functional Requirements](#6-non-functional-requirements)
7. [Risks and Known Issues](#7-risks-and-known-issues)
8. [Project Success Metrics](#8-project-success-metrics)
9. [Appendices](#9-appendices)

---

## 1. Introduction

### 1.1. Purpose of the Document

This document defines the functional and non-functional requirements for the **Amazon Clone (IPAY)** web platform — an e-commerce system developed as part of a course team project for the ASP.NET Core course. Version 1.1 reflects the **actual state of the implementation** and serves as the single source of truth for the development team, QA, and project management during implementation (Scrum sprints), testing (NUnit), documentation (Scalar/OpenAPI, Doxygen), and defense.

### 1.2. Product Scope

IPAY is an e-commerce web platform that lets users browse products, manage a shopping cart, place orders with (simulated) online payment, and rate products. Sellers manage their own products; moderators and administrators manage the catalog, orders, seller requests, and users. The platform also contains a video catalog (series, episodes, genres) with metadata import from OMDb. The project is a **team project** using Scrum, Git, and Clean / Onion architecture.

### 1.3. Document Audience

The development team (Backend, Frontend, Full-stack), project manager, members of the defense committee, and QA engineers (within the educational process).

### 1.4. Definitions, Acronyms, and Abbreviations

| Term | Definition |
|---|---|
| SRS | Software Requirements Specification |
| MVP | Minimum Viable Product |
| MoSCoW | Must / Should / Could / Won't have prioritization |
| JWT | JSON Web Token |
| 2FA | Two-Factor Authentication (one-time code sent by email) |
| Firestore | Google Cloud Firestore — a document-oriented NoSQL database |
| DTO | Data Transfer Object |
| SOLID | Object-oriented design principles |
| CI/CD | Continuous Integration / Continuous Deployment |
| OMDb | Open Movie Database API (source of movie/series metadata) |
| PCI DSS | Payment Card Industry Data Security Standard |

### 1.5. References

- Methodological guidelines for the .NET course team project (2026)
- ISO/IEC/IEEE 29148:2018 — Requirements Engineering
- OWASP Top 10 (2025 edition)
- Microsoft .NET Architecture Guides: Clean Architecture, eShopOnWeb
- Law of Ukraine "On Personal Data Protection" No. 2297-VI
- Repositories: `IPAY_backend` (ASP.NET Core Web API), `ipay_client` (React SPA), `IPAY_SRS`
- API documentation: Scalar registry; code documentation: Doxygen (Render); `IPAY_User_Guide.html`

---

## 2. General Description

### 2.1. Product Perspective

IPAY is being developed as a standalone educational product from scratch; its architectural and UX decisions are based on best practices of modern e-commerce platforms (the visual style follows Amazon). The system consists of a client-side application (React SPA), a server-side application (ASP.NET Core Web API), and a data layer (Google Cloud Firestore). Authentication combines the project's own JWT tokens with Firebase Authentication (Google sign-in, email verification, password reset emails).

### 2.2. User Classes and Privileges

| Role | Description | Key Permissions |
|---|---|---|
| Guest | Unauthenticated visitor (optionally with a guest session) | Browse catalog, search, filter, sort, view product details, cart in localStorage |
| Customer | Authenticated user | Server-side cart, place and cancel orders, history, ratings and reviews, profile, apply for seller status |
| Seller | Customer whose request was approved | Everything a Customer can do + create/edit/delete **only their own** products, upload images |
| Moderator | Platform staff | Manage products, categories, manufacturers, orders, video catalog; view users; ban/unban users (except admins and moderators); assign/remove the Seller role; review seller requests |
| Administrator | Platform manager | Everything a Moderator can do + assign/remove the Moderator role, CRUD for users and guests; **nobody can be made an administrator through the API** |
| System | Automated processes | Role check in the JWT on every request, refresh token rotation, sending emails |

> **Changes from v1.0:** the Moderator role was added; the "Seller (Personal/Business)" role is clarified — there is no personal/business distinction in the implementation; "System" no longer performs moderation, archiving, or cart cleanup.

### 2.3. Operating Environment

| Component | Specification (actual) |
|---|---|
| Client | Chrome, Firefox, Safari, Edge browsers (latest 2 versions); responsive UI (Tailwind CSS 4) |
| Server | ASP.NET Core on **.NET 10**; Docker image (port 10000), hosted on **Render** |
| Client Framework | **React 19.2**, Vite 8, React Router 7, Axios, Tailwind CSS 4, Firebase JS SDK 12 |
| Server Framework | ASP.NET Core Web API (controllers), JWT Bearer, authorization policies |
| Database | **Google Cloud Firestore** (primary store; one collection per entity) |
| Data Access | Custom `FirestoreRepository<TEntity, TDocument>` + *Document* classes for mapping |
| Authentication | JWT + refresh tokens (custom), Firebase Authentication (Google, email verification, password reset) |
| Images | Cloudinary (CDN) |
| Email | SMTP (MailKit) |
| External Data | OMDb API (video catalog) |
| Architecture | Clean / Onion: `Domain` → `Application` → `Infrastructure` → `WebApi` |
| Interface Languages | English (en), Ukrainian (ua), Russian (ru) |

### 2.4. Design and Implementation Constraints

| Layer | Technology | Rationale |
|---|---|---|
| Frontend | React 19.2 + Vite + Tailwind | Component-based approach, fast builds, utility-first styling |
| Backend | ASP.NET Core Web API (.NET 10) | Cross-platform, DI, authorization policies |
| Validation / Mapping | FluentValidation 12, AutoMapper 12 | Declarative rules; DTOs isolated from domain entities |
| Database | Firestore | Serverless, no administration; real-time capabilities of the SDK |
| Auth | JWT Bearer + Firebase Authentication | Stateless API; Google sign-in, email verification, and password reset delegated to Firebase |
| API Documentation | Swagger UI (Development only) + Scalar registry | Automatic generation from OpenAPI; public reference |
| Testing | NUnit 4 + Moq + coverlet | Requirement of the methodological guidelines |
| Code Documentation | Doxygen | Public site generated from XML comments |

> **Deviations from v1.0:** ASP.NET Core Identity, EF Core / SQL Server, DocFX, SignalR, and Elasticsearch are **not used** in the project. All related requirements were reworded for Firestore (see section 6).

### 2.5. Assumptions & Dependencies

| No. | Description | Type | Impact if Violated |
|---|---|---|---|
| A-01 | The team uses Git (GitHub) with a feature-branch workflow and Pull Request reviews | Internal | Version control issues, difficulties with reporting |
| A-02 | A real payment provider (LiqPay/Fondy/Stripe) is **not connected** in the current version; `MockPaymentProvider` is used | External | Payment is only simulated (FR-10) |
| A-03 | The team follows Scrum: 2-week sprints, daily stand-ups, Kanban board | Methodological | Inability to provide a Scrum report |
| A-04 | Availability and quotas of Firebase/Firestore, Cloudinary, the SMTP server, and the OMDb API | External | Google sign-in, image uploads, 2FA emails, and movie import become unavailable |
| A-05 | Secrets (Jwt:Key, Firebase service-account key, SMTP passwords, Cloudinary/OMDb keys) are supplied via user-secrets or environment variables, not via Git | Internal | Credential compromise |

---

## 3. Project Scope

### 3.1. In Scope

- Catalog, cart, orders, simulated payment, admin panel
- Registration, authentication (email+password, Google), email 2FA, role-based authorization (Customer, Seller, Moderator, Admin)
- Seller status requests; seller area for own products
- Product ratings (1–5 stars) and text reviews
- Filtering, sorting, and cursor-based pagination of products
- Video catalog: series, episodes, genres, OMDb import
- Administrative panel: products, categories, manufacturers, media, orders, seller requests, users
- Responsive UI, internationalization (en / ua / ru)
- Unit and integration tests (NUnit) for business logic and controllers
- API documentation (Swagger/Scalar), code documentation (Doxygen), User Guide
- Deployment to Render (Docker for the backend)

### 3.2. Out of Scope

- Native mobile applications (iOS/Android)
- Own payment gateway (acquiring) and, in the current version, a real provider integration
- ML-based product recommendations
- Integration with delivery services (Nova Poshta API) — order statuses only
- Microservice architecture — a modular monolith is used
- **Postponed (not implemented):** Wishlist, push/real-time notifications (SignalR), analytics dashboard, Elasticsearch, product description and stock quantity (SKU), user avatar, category hierarchy

---

## 4. Functional Requirements

*Format: **[MoSCoW Priority]** ID — Description → Acceptance Criterion (AC) · Implementation status*

### 4.1. Authentication and Registration

**[Must] FR-01** — User registration via email + password or Google (through Firebase Authentication). ✅
→ *AC:* Name 2–50 characters (letters, space, hyphen); password ≥ 8 characters with an uppercase letter, a lowercase letter, a digit, and a special character; password confirmation; email ≤ 100 characters and unique; email verification via Firebase (`Firebase:RequireEmailVerification`); for Google sign-in the email is considered verified; validation via FluentValidation (`RegisterValidator`).

**[Must] FR-02** — Authentication with role separation (Guest / Customer / Seller / Moderator / Admin). ✅
→ *AC:* JWT (HS256) contains `NameIdentifier`, `Email`, and `Role` claims; access token lifetime is configurable (`Jwt:AccessTokenMinutes`, default 20 min); endpoints are protected by policies (`AdminOnly`, `ModeratorOnly`, `ModeratorOrAdmin`, `SellerOnly`, `CustomerOnly`, `RegisteredUser`, `AnyAuthenticated`, `NotBanned`); a banned user fails every policy that includes `NotBanned`.

**[Must] FR-03** — Password recovery via email. ✅
→ *AC:* `POST /api/auth/password-reset/start` always returns 204 (does not reveal whether an account exists); the reset email is sent by Firebase; `POST /api/auth/password-reset/confirm` accepts a Firebase ID token and the new password; failures are returned as 400 (not 401).

### 4.2. Product Catalog

**[Must] FR-04** — View a product list with pagination (default 12 per page). ✅
→ *AC:* Cursor-based server-side pagination (`limit`, `lastDocId`) at the Firestore query level; `GET /api/products/all` is also available. *Note: pagination is cursor-based, not page-number-based.*

**[Must] FR-05** — View product details. 🟡
→ *AC:* Displays name, price, discounted price (final price = discounted if `0 < DiscountedPrice < Price`), image, manufacturer, category, rating and rating count, seller. *Not implemented: a separate description field, a gallery of up to 5 photos (currently a single `ImageUrl`), stock quantity/SKU, "related products" block.*

**[Must] FR-06** — Search and filter products. 🟡
→ *AC:* `GET /api/products/filter` by `categoryId`, `minPrice`/`maxPrice`, `brand` (manufacturer), `search` (by name) with cursor-based pagination. *Not implemented: rating filter; Elasticsearch full-text search.*

**[Should] FR-07** — Sort products. ✅
→ *AC:* `GET /api/products/sort` by `sortBy` = `price` / `rating` / `date` and `sortDir` = `asc` / `desc`, performed at the database query level. *"Popular" sorting is not implemented.*

### 4.3. Cart and Checkout

**[Must] FR-08** — Add/remove products from the cart and change quantities. ✅
→ *AC:* Guest: cart in `localStorage`; authenticated user: cart in Firestore (`/api/cart`, `/api/cart/items`, `/api/cart/clear`); quantity updates without page reload; total is calculated on the client in real time; cart is cleared after an order is placed.

**[Must] FR-09** — Place an order by providing a delivery address. 🟡
→ *AC:* Order is created by an authenticated user with status `Pending`; address consists of country, city, and street; item price is frozen at order time (final product price); `UserId` is taken from the JWT. *Not implemented: storing full name and phone in the order, decreasing stock quantity.*

**[Must] FR-10** — Pay for an order. 🟡
→ *AC:* `POST /api/payments/create` starts payment for an order with status `Pending` and returns a `TransactionId` and a confirmation URL; `GET /api/payments/mock/confirm` moves the order to `Paid` on success (recorded in the status history); a transaction ID containing `FAIL` simulates a rejection; card data is never processed or stored. *There is no real integration (LiqPay/Fondy/Stripe) — `MockPaymentProvider` is used (`Provider = "MockPay"`, currency UAH); "Cash on delivery" is not implemented.*

**[Must] FR-11** — Order history in the personal account. ✅
→ *AC:* `GET /api/orders/my` returns the user's orders with items; status change history is available (`GET /api/orders/{id}/status-history`: old/new status, time).

**[Should] FR-21** ➕ — Order cancellation by its owner. ✅
→ *AC:* `POST /api/orders/{id}/cancel`: allowed only for `Pending` / `Paid` orders; other users' orders return 404; after shipping — 400.

### 4.4. Reviews and Ratings

**[Should] FR-12** — Product rating (1–5 stars) and review. 🟡
→ *AC (ratings):* `POST /api/ratings/{productId}` — one rating per user per product (repeat → 409), rating your own product as a seller is forbidden (403), `RegisteredUser` only; average rating and rating count are recalculated automatically; `GET /api/ratings/mine` returns the current user's ratings. *(AC reviews:)* `POST /api/reviews` (authenticated, non-banned roles) stores a rating and a comment. *Not implemented: the "only after purchase" and "one review per product" restrictions.*

**[Should] FR-13** — View reviews on the product page. 🟡
→ *AC:* `GET /api/reviews/product/{productId}` is public; a review can be deleted by its author, a moderator, or an admin. *Pagination and sorting of reviews are not implemented.*

### 4.5. Administrative Panel

**[Must] FR-14** — CRUD for product categories. 🟡
→ *AC:* `GET /api/categories` is public; create/update/delete and product assignment are available to Seller, Admin, and Moderator roles. *The category structure is flat (no parent/child hierarchy); prohibiting deletion of a non-empty category is not confirmed.*

**[Must] FR-15** — CRUD for products. 🟡
→ *AC:* Admin/Moderator — any products; Seller — only their own (`SellerId` comes from the JWT, another seller's product → 403); images are uploaded to Cloudinary (`POST /api/images`, 5 MB limit, an HTTPS URL is returned); DTO validation (`CreateAdminProductDto`, `CreateSellerProductDto`, etc.). *Deletion is physical (no soft delete / `IsDeleted`); "description" and "stock quantity" fields are not implemented.*

**[Must] FR-16** — Order management. 🟡
→ *AC:* View all orders (`GET /api/orders` — Admin/Moderator); change status `Pending → Paid → Shipped → Delivered / Cancelled` (`PUT /api/orders/{id}/status`); change history is stored. *Notifying the user about status changes (email/SignalR) is not implemented.*

**[Must] FR-17** — User management. 🟡
→ *AC:* User list (Admin/Moderator); ban/unban: Admin — anyone except admins; Moderator — anyone except admins and moderators; role changes follow the rules of FR-22; admin CRUD for users and guests (`/api/admin/customers`, `/api/admin/guests`). *The reason for blocking is not stored (only the `IsBanned` flag).*

### 4.6. Personal Account

**[Must] FR-18** — Edit profile. 🟡
→ *AC:* `/api/profile`: change name, phone (normalized format, validated), password (via email confirmation), email (two-step process with confirmation, uniqueness). *Not implemented: avatar, saved delivery address in the profile.*

**[Should] FR-19** — Wishlist. ❌
→ *Postponed; absent from the code.*

**[Could] FR-20** — Push notifications about order status changes. ❌
→ *Postponed. The `Hubs` and `SignalR` folders in the backend are empty placeholders. The client detects role changes by periodically pinging `/api/session` (see FR-26).*

### 4.7. New Requirements (added in version 1.1)

**[Should] FR-22** ➕ — Role management. ✅
→ *AC:* Make Seller — Admin or Moderator; Make Moderator and Remove Moderator — Admin only; Seller → Customer — Admin or Moderator; the Admin role cannot be granted through the API; admins (and, for Moderators, other moderators) cannot be changed; a banned account cannot be promoted.

**[Should] FR-23** ➕ — Seller status requests. ✅
→ *AC:* A Customer submits a request with a message (≤ 500 characters); one active (`Pending`) request per user; banned users, existing sellers, and staff cannot apply; Admin/Moderator approve (`Approved` — the role becomes Seller) or reject (`Rejected`), recording the reviewer and time (`/api/seller-requests/...`); a request cannot be reviewed twice.

**[Should] FR-24** ➕ — Two-factor authentication by email. ✅
→ *AC:* After the password, a 6-digit code is sent (lifetime 10 min, up to 5 attempts, resend no more than once every 30 s); the session is issued only after `POST /api/auth/login/verify-code`; Google sign-in does not require 2FA; enabled via `TwoFactor:Enabled` + a configured SMTP (if SMTP is missing, 2FA is disabled with a warning in the logs).

**[Must] FR-25** ➕ — Refresh tokens with rotation. ✅
→ *AC:* Refresh token is 64 random bytes; only its SHA-256 hash is stored in the DB; lifetime `Jwt:RefreshTokenDays` (default 14 days); the token is rotated on every refresh; reuse of a revoked token revokes all of the user's sessions; refresh is rejected for banned users; the role is re-read from the DB.

**[Should] FR-26** ➕ — Role synchronization between token and database. ✅
→ *AC:* On every request the role in the JWT is compared with the user's current role; on mismatch the request is rejected (401) and the client logs the user out; `GET /api/session` is used by the client to detect role changes quickly.

**[Could] FR-27** ➕ — Guest sessions. 🟡
→ *AC:* The guest service (`IGuestService`) creates guest sessions with a `SessionKey`; guests have the `Guest` role and cannot rate products or place orders.

**[Should] FR-28** ➕ — Manufacturers. ✅
→ *AC:* CRUD for manufacturers in the admin panel; in the product form the manufacturer is chosen from a list (in `Product` the manufacturer is stored as a string).

**[Could] FR-29** ➕ — Video catalog (series, episodes, genres). 🟡
→ *AC:* CRUD for series, episodes, and genres (`/api/series`, `/api/episodes`, `/api/genres`); metadata import via OMDb (`/api/omdb`); pages `/movies`, `/movies/categories`, `/movies/title/:id`. *The films module (`FilmService`, `FilmRepo`) is commented out in DI.*

**[Should] FR-30** ➕ — Multilingual interface. ✅
→ *AC:* Languages en / ua / ru (switcher in the header; the choice is stored in `localStorage`; the document `lang` attribute and date/number locale follow the language); translation dictionary in `i18n/translations.js`.

---

## 5. Use Cases

### UC-01: Placing an Order

| | |
|---|---|
| **Actor** | Customer |
| **Preconditions** | User is authenticated and not banned; products are in the cart |
| **Main Flow** | 1. User goes to the cart → 2. Checks products and quantities → 3. Clicks "Checkout" → 4. Enters the delivery address (country, city, street) → 5. System creates the order with status `Pending` and freezes item prices → 6. Client starts payment (`/api/payments/create`) → 7. Payment is confirmed (mock provider) → 8. Order moves to `Paid`, the cart is cleared, the user sees the order in the history |
| **Alternative Flow** | 7a. Payment rejected → error message, the order stays `Pending`, the user can retry; 8a. Before shipping the user cancels the order (`Cancelled`) |
| **Postconditions** | Order saved in Firestore with status `Paid`; status history entry created. *(Stock quantity is not decreased — not implemented.)* |

### UC-02: Adding a Product (Administrator, Moderator, or Seller)

| | |
|---|---|
| **Actor** | Admin / Moderator / Seller |
| **Preconditions** | Actor is authenticated with the appropriate role |
| **Main Flow** | 1. Admin/Moderator: enters the admin panel (`/admin`) → "Products" → "Add product"; Seller: page `/sell` → 2. Fills in the form (name, price, discounted price, manufacturer, category) → 3. Uploads an image (Cloudinary, ≤ 5 MB) → 4. Saves |
| **Alternative Flow** | 3a. File too large or unsupported format → error, saving is blocked; 4a. Seller tries to edit another seller's product → 403 |
| **Postconditions** | Product is available in the catalog; for a Seller, `SellerId` equals the id from the JWT |

### UC-03: Blocking a User

| | |
|---|---|
| **Actor** | Admin or Moderator |
| **Preconditions** | A violation has been detected (spam, fraud); the target is not an admin (for a moderator — also not a moderator) |
| **Main Flow** | 1. Actor opens the "Users" tab → 2. Clicks "Ban" → 3. System sets `IsBanned = true` |
| **Alternative Flow** | 3a. Target is an admin/moderator → 403 with an explanation |
| **Postconditions** | The user fails `NotBanned` policies and cannot refresh their token; the existing access token loses access to protected endpoints. *Blocking reason and an email notification are not implemented.* |

### UC-04: Seller Status Request

| | |
|---|---|
| **Actor** | Customer; Admin / Moderator |
| **Preconditions** | Customer is authenticated, not banned, has the Customer role, and has no active request |
| **Main Flow** | 1. Customer submits a request with a message → 2. Request gets status `Pending` → 3. Admin/Moderator approves it in the "Seller requests" tab → 4. The user's role becomes Seller → 5. After the next sign-in (or detection of the role mismatch in the token) the user sees the seller area |
| **Alternative Flow** | 3a. Request is rejected (`Rejected`) |
| **Postconditions** | Request is closed with the reviewer (`ReviewedBy`) and time recorded |

### UC-05: Sign-in with Two-Factor Authentication

| | |
|---|---|
| **Actor** | User with a verified email |
| **Preconditions** | `TwoFactor:Enabled` is on and SMTP is configured |
| **Main Flow** | 1. User enters email and password → 2. System returns `twoFactorRequired`, `challengeId`, `maskedEmail` → 3. User enters the code from the email → 4. `POST /api/auth/login/verify-code` returns access and refresh tokens |
| **Alternative Flow** | 3a. Wrong code — up to 5 attempts, then a new sign-in is required; 3b. Code did not arrive — resend is allowed after 30 s |
| **Postconditions** | Token pair issued; a refresh token record (hash) is created |

---

## 6. Non-Functional Requirements

*The "Status" column reflects the current implementation state.*

### 6.1. Performance

| ID | Requirement | Target | Verification Method | Status |
|---|---|---|---|---|
| NFR-01 | API response time | p95 < 300 ms for CRUD operations | k6 / Postman / Swagger | 🟡 Not systematically measured |
| NFR-02 | Catalog page load time | < 2 s on broadband | Lighthouse | 🟡 Not systematically measured |
| NFR-03 | Throughput | 100 concurrent users without degradation | Load testing (k6) | ❌ Not performed |

### 6.2. Reliability and Availability

| ID | Requirement | Status |
|---|---|---|
| NFR-04 | Uptime 99.5% (educational project, not a production SLA) | 🟡 Render hosting; no monitoring |
| NFR-05 | Graceful degradation if the payment provider is unavailable — "Cash on Delivery" | ❌ Not implemented (provider is a mock) |
| NFR-06 | Soft delete for critical entities (users, products, categories) | ❌ Deletion is physical |
| NFR-26 ➕ | Missing SMTP/OMDb configuration does not block startup; the app writes a warning to the log (2FA is disabled — "fail-open") | ✅ |

### 6.3. Security

| ID | Requirement | Status |
|---|---|---|
| NFR-07 | Protection against OWASP Top 10: input validation (FluentValidation), JWT in the `Authorization` header (CSRF-resistant), no SQL injection (NoSQL, SDK queries), Broken Access Control (authorization policies and owner checks: seller products, order cancellation) | 🟡 See R-06 — known gaps in order authorization |
| NFR-08 | Password hashing via a custom `PasswordHashingService`; refresh tokens — SHA-256 | ✅ *(ASP.NET Core Identity is not used)* |
| NFR-09 | Short-lived access token (configurable, default 20 min) + 14-day refresh token with rotation and reuse detection | ✅ |
| NFR-10 | Login rate limiting (≤ 5 attempts/min per IP) | 🟡 No general rate limiting; a 2FA code attempt limit (5) and a resend cooldown (30 s) are in place |
| NFR-11 | HTTPS (TLS 1.2+) for all communications | ✅ `UseHttpsRedirection`; TLS handled by Render |
| NFR-27 ➕ | Secrets are not stored in Git (user-secrets / environment variables; the Firebase service-account key is in `.gitignore`) | 🟡 See R-07 |
| NFR-28 ➕ | Access is closed for banned users via the `NotBanned` policy and by rejecting their refresh token | ✅ |

### 6.4. Scalability

| ID | Requirement | Status |
|---|---|---|
| NFR-12 | Clean / Onion architecture (`IPAY.Domain` → `IPAY.Application` → `IPAY.Infrastructure` → `IPAY.WebApi`) | ✅ |
| NFR-13 | async/await for all I/O operations; cursor-based pagination of Firestore queries | ✅ *(IQueryable/EF Core are not used)* |

### 6.5. Interface and Accessibility

| ID | Requirement | Status |
|---|---|---|
| NFR-14 | Responsive design (mobile 320 px, tablet 768 px, desktop 1920 px) | ✅ Tailwind CSS 4 |
| NFR-15 | Basic UX principles: readable fonts, sufficient contrast, clear error messages | 🟡 No formal accessibility audit |
| NFR-29 ➕ | SEO artifacts: `robots.txt`, `sitemap.xml`, `og-image`, favicon | ✅ |

### 6.6. Quality and Testing

| ID | Requirement | Status |
|---|---|---|
| NFR-16 | Unit test coverage (NUnit) ≥ 60% of critical business paths (registration, cart, orders, payment) | 🟡 Tests exist for authentication services (Customer, JWT, Login, Register, hashing), cart, orders, addresses, categories, manufacturers, products, media; there are no dedicated payment tests; the coverage percentage is not measured |
| NFR-17 | Integration tests for the API | 🟡 `AuthFlowIntegrationTests`, `ShopFlowIntegrationTests` on in-memory repositories; `WebApplicationFactory` is not used |
| NFR-18 | Code documentation: XML comments; documentation generation | 🟡 Doxygen site + Scalar registry; XML comments are incomplete |
| NFR-19 | Consistent naming convention documented in `CONTRIBUTING.md` | ❌ File is missing |
| NFR-30 ➕ | Backend containerization (`Dockerfile`, port 10000) and deployment to Render; the client uses `VITE_API_URL` to reach the API | ✅ |

### 6.7. Architectural Requirements (Specific to the Course Project)

| ID | Requirement | Status |
|---|---|---|
| NFR-20 | Design patterns: Repository, Unit of Work, CQRS, Specification | 🟡 Repository is implemented (`IRepository<T>`, specialized repositories); Unit of Work, CQRS/MediatR, and Specification are not used (the `Specifications` folder is empty) |
| NFR-21 | SOLID: Single Responsibility for services, Dependency Injection, dependency inversion | ✅ Interfaces in Domain/Application, implementations in Infrastructure |
| NFR-22 | Use of LINQ and Fluent API | 🟡 LINQ — yes; EF Core Fluent API — not applicable (Firestore); FluentValidation is used instead |
| NFR-23 | async/await for I/O-bound operations | ✅ |
| NFR-24 | Logging (Serilog / NLog) | 🟡 Standard ASP.NET Core `ILogger`; no Serilog/NLog and no file output |
| NFR-25 | Caching (In-Memory / Redis) | 🟡 `MemoryCache` is used only for 2FA challenges; catalog caching is on the client (`api/cache.js`) |

---

## 7. Risks and Known Issues

| ID | Risk | Probability | Impact | Mitigation Strategy | Status |
|---|---|---|---|---|---|
| R-01 | Insufficient team experience with React / ASP.NET Core | Medium | High | Early prototyping; role distribution; consultations with the project manager | Reduced |
| R-02 | No real payment provider | High | Medium | Mock provider; swappable through `IPaymentProvider` | **Implemented as a mock** |
| R-03 | Scope creep (video catalog, 3 languages, 2FA — outside the original plan) | High | Medium | Strict adherence to MoSCoW; record changes in the SRS | Occurred; documented in v1.1 |
| R-04 | Merge conflicts in Git | Medium | Medium | Divide backend/frontend responsibilities; code review | Open |
| R-05 | Low test coverage before the deadline | Medium | High | Tests in the Definition of Done; CI gate; measure with coverlet | Open |
| R-06 ➕ | **Authorization gaps in `OrdersController`:** endpoints for changing status (`PUT /orders/{id}/status`), deleting an order, working with items, and reading by `id` and by `userId` are available to any authenticated role (Customer/Seller) without an owner check; `GET /api/payments/mock/confirm` is anonymous; Sellers can modify categories | High | High | Restrict status change/deletion to Admin/Moderator; add an owner check for reads; protect the confirm endpoint (signature/secret); restrict category CRUD | **Open — priority** |
| R-07 ➕ | Secrets and the Firebase service-account key in the working directory; CORS policy is `AllowAnyOrigin` | Medium | High | Verify `.gitignore` covers both copies of `firebase-service-account.json`; restrict CORS to the client domain; rotate keys if needed | Open |
| R-08 ➕ | No application-level transactions for multi-step operations (order creation, ratings) and no stock quantities in Firestore | Medium | Medium | Use Firestore transactions/batched writes; add a stock field to Product | Open |
| R-09 ➕ | Technical debt: empty `ExamTest.*` folders, commented-out Films module, duplicate repositories (`OrderRepo`/`OrdersRepo`), two copies of the client (`WebstormProjects/ipay_client` and `source/repos/ipay_client`) | Medium | Low | Remove dead code; pick a single client repository | Open |

---

## 8. Project Success Metrics

| Metric | Target | Current State | Deadline |
|---|---|---|---|
| MVP Functionality | 100% of Must-have features (FR-01..FR-11, FR-14..FR-18) | All Must items are implemented fully or partially; deviations: FR-05, FR-06, FR-09, FR-10 (mock), FR-14, FR-15, FR-16 (no notifications), FR-17, FR-18 — see notes | Before defense |
| Test Coverage | ≥ 60% unit tests (NUnit) | Tests are written; percentage not measured (add `dotnet test --collect:"XPlat Code Coverage"`) | Before defense |
| Documentation | API reference and code documentation published | Scalar registry, Doxygen (Render), `IPAY_User_Guide.html`, README — ✅ | Before defense |
| Deployment | Website deployed and publicly accessible | Frontend: `ipay-client.onrender.com`; backend — Docker on Render — ✅ | Before defense |
| Presentation | 7–15 slides, video demo ≤ 2 min | Screenshots in `IPAY_results/`; slides and video — per team status | Before defense |
| Git Report | Commit history, branches, merge requests | Ongoing | Throughout the project |
| Scrum Report | 3+ sprints, backlog, Kanban board | Ongoing | Throughout the project |

---

## 9. Appendices

### 9.1. Glossary of Terms

| Term | Definition |
|---|---|
| Soft Delete | Logical deletion (`IsDeleted` flag); not implemented in the project |
| Mock | Simulation of an external service without real requests (`MockPaymentProvider`) |
| Clean Architecture | Layered architecture with business logic isolated from infrastructure |
| Cursor-based pagination | Page-by-page loading using the id of the last document (`lastDocId`) |
| Document (Firestore) | A class between the domain entity and the Firestore document (`*Document`) |

### 9.2. Diagrams (for the Next Iteration of the Document)

- [ ] Use Case Diagram (actors: Guest, Customer, Seller, Moderator, Admin)
- [ ] Firestore data model (collections: customers, guests, products, categories, manufacturers, carts, cartItems, orders, orderItems, addresses, orderStatusHistory, reviews, productRatings, sellerRequests, refreshTokens, series, episodes, genres)
- [ ] Sequence Diagrams for the payment flow (FR-10) and the 2FA sign-in flow (FR-24)
- [ ] Component Diagram: React → ASP.NET Core Web API → Firestore / Firebase Auth / Cloudinary / SMTP / OMDb
- [ ] Deployment Diagram: Render (client + Docker backend)

### 9.3. API Overview (per README)

| Area | Routes |
|---|---|
| Auth | `/api/auth/register`, `/login`, `/login/verify-code`, `/login/resend-code`, `/google`, `/password-reset/{start,confirm}` |
| Session | `/api/session` |
| Profile | `/api/profile`, `/profile/{username,phone,password,email}` |
| Products | `/api/products`, `/all`, `/{id}`, `/filter`, `/sort`, `/seller`, `/seller/mine` |
| Categories / Manufacturers | `/api/categories`, `/api/manufacturers` |
| Cart | `/api/cart`, `/cart/items`, `/cart/clear` |
| Orders | `/api/orders`, `/{id}`, `/my`, `/{id}/status`, `/{id}/status-history`, `/{id}/cancel`, `/{id}/items` |
| Payments | `/api/payments/create`, `/api/payments/mock/confirm` |
| Ratings / Reviews | `/api/ratings/{productId}`, `/ratings/mine`, `/api/reviews`, `/reviews/product/{productId}` |
| Seller requests | `/api/seller-requests`, `/mine`, `/{id}/{approve,reject}` |
| Media | `/api/series`, `/episodes`, `/genres`, `/omdb` |
| Images | `/api/images` |
| Admin | `/api/admin/users`, `/users/{id}/{make-seller,make-moderator,remove-moderator,make-customer}`, `/{id}/{ban,unban}`, `/customers`, `/guests` |

### 9.4. Requirements Traceability Matrix

| Requirement | Business Goal | Use Case | Automated tests (files) | Status |
|---|---|---|---|---|
| FR-01, FR-02 | Registration and sign-in | UC-05 | `RegisterDtoServiceTests`, `LoginDtoServiceTests`, `JwtServiceTests`, `AuthControllerTests`, `AuthFlowIntegrationTests` | Implemented |
| FR-08 | Cart | UC-01 | `CartServiceTests`, `CartItemServiceTests`, `CartControllerTests` | Implemented |
| FR-09, FR-11, FR-21 | Orders | UC-01 | `OrderServiceTests`, `OrderItemServiceTests`, `OrdersControllerTests`, `ShopFlowIntegrationTests` | Implemented |
| FR-10 | Payment | UC-01 | — (no payment tests) | Mock; tests planned |
| FR-15 | Product management | UC-02 | `ProductServiceTests`, `ProductsControllerTests`, `CloudinaryImageServiceTests`, `ImagesControllerTests` | Implemented |
| FR-17, FR-22 | Users and roles | UC-03 | `UserHierarchyTests`, `NotBannedHandlerTests` | Partial (Admin controller not covered) |
| FR-23 | Seller requests | UC-04 | — | Tests planned |
| FR-14, FR-28 | Categories, manufacturers | — | `CategoryServiceTests`, `ManufacturerServiceTests`, `CategoriesControllerTests`, `ManufacturersControllerTests` | Implemented |
| FR-29 | Video catalog | — | `SeriesServiceTests`, `EpisodeServiceTests`, `GenreServiceTests`, `OmdbServiceTests`, `MediaValidatorsTests` | Implemented |

*It is recommended to maintain the matrix in GitHub Projects with live synchronization.*

### 9.5. Recommended Team Role Distribution (2–4 People)

| Role | Responsibilities | Technologies |
|---|---|---|
| Backend Lead | API architecture, data layer, business logic, payment | C#, ASP.NET Core, Firestore, FluentValidation, AutoMapper |
| Frontend Lead | UI/UX, React components, contexts (Auth, Cart, Language), API integration | React 19, Vite, Tailwind, Axios, React Router |
| Full-stack / DevOps | Auth (JWT, Firebase), 2FA, Docker, deployment to Render | JWT, Firebase Auth, Docker, Render |
| QA / Analyst | NUnit tests, SRS, documentation, presentation, Scrum reports | NUnit, Moq, Scalar/Swagger, Doxygen |
