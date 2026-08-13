# Software Requirements Specification

## Amazon Clone — An online commercial platform

| | |
|---|---|
| **Document Version** | 1.0 |
| **Status** | Draft for Review |
| **Date** | 13th of August 2026 |
| **Author** | IPAY (Incredible Project Aspnet Yes) |
| **Approved by** | Олександр Загоруйко |
| **Classification** | Internal / Academic |
| **Standard** | IEEE 830-1998 / ISO/IEC/IEEE 29148 |

### Change history

| Version | Date | Author | Description of the changes |
|---|---|---|---|
| 1.0 | 13.08.2026 | IPAY | Initial version of the SRS: figuring out metrics, requirements, Use Cases, risks |

---

## Contents

1. [Introduction](#1-introduction)
2. [General Description](#2-general-description)
3. [Project Scope](#3-project-scope)
4. [Functional Requirements](#4-functional-requirements)
5. [Use Cases](#5-use-cases)
6. [Non-Functional Requirements](#6-non-functional-requirements)
7. [Risks](#7-risks)
8. [Project Success Metrics](#8-project-success-metrics)
9. [Appendices](#9-appendices)

---

## 1. Introduction

### 1.1. Purpose of the Document

This document defines the complete set of functional and non-functional requirements for the **Amazon Clone** web platform — an e-commerce system developed as part of a course team project for the ASP.NET Core course. The document serves as the single source of truth for the development team, QA, and project management throughout all stages of the project lifecycle: planning, implementation (Scrum sprints), testing (NUnit), documentation (DocFX/OpenAPI), and defense.

### 1.2. Product Scope

Amazon Clone is a full-featured e-commerce web platform that allows users to browse products, manage a shopping cart, place orders with online payment, and leave reviews, while administrators can manage the catalog, orders, and users. The project is implemented as a **team project** using the Scrum methodology, Git version control, and Clean / Onion architecture.

### 1.3. Document Audience

The development team (Backend, Frontend, Full-stack), project manager, members of the defense committee, and QA engineers (within the educational process).

### 1.4. Definitions, Acronyms, and Abbreviations

| Term | Definition |
|---|---|
| SRS | Software Requirements Specification |
| MVP | Minimum Viable Product |
| MoSCoW | Must / Should / Could / Won't have prioritization |
| JWT | JSON Web Token |
| EF Core | Entity Framework Core |
| CQRS | Command Query Responsibility Segregation |
| SOLID | Object-oriented design principles |
| CI/CD | Continuous Integration / Continuous Deployment |
| SKU | Stock Keeping Unit (product code) |
| PCI DSS | Payment Card Industry Data Security Standard |
| PWA | Progressive Web App |

### 1.5. References

- Methodological guidelines for the .NET course team project (2026)
- ISO/IEC/IEEE 29148:2018 — Requirements Engineering
- OWASP Top 10 (2025 edition)
- Microsoft .NET Architecture Guides: Clean Architecture, eShopOnWeb
- Law of Ukraine “On Personal Data Protection” No. 2297-VI

---

## 2. General Description

### 2.1. Product Perspective

Amazon Clone is being developed as a standalone educational product from scratch; however, its architectural and UX decisions are based on best practices used by modern e-commerce platforms. The system consists of a client-side application (React SPA), a server-side application (ASP.NET Core Web API), and a data layer (Firebase).

### 2.2. User Classes and Privileges

| Role | Description | Key Permissions |
|---|---|---|
| Guest | Unauthenticated visitor | Browse catalog, search, filtering, view product details |
| Customer | Authenticated user | Cart, checkout, purchase history, reviews, wishlist, profile |
| Administrator | Platform manager | CRUD for products, categories, orders, users; access to analytics |
| System | Automated processes | Moderation, notifications, cart cleanup, archiving |

### 2.3. Operating Environment

| Component | Specification |
|---|---|
| Client | Chrome, Firefox, Safari, Edge browsers (latest 2 versions); resolution from 320 px |
| Server | ASP.NET Core 9/10, hosted on IIS / Azure / AWS / VPS |
| Client Framework | React 19+ (Vite / Next.js optional), Axios, React Router |
| Server Framework | ASP.NET Core Web API, ASP.NET Core Identity |
| Database | **Firebase Firestore** (for real-time features, as an alternative) |
| Firebase SDK, Data Connectors/Converters | For obvious reasons and without ORM |
| Real-time | SignalR (order status notifications, support chat) |
| Architecture | Clean Architecture / Onion Architecture |
| Interface Languages | Ukrainian (uk-UA), English (en-US) |

### 2.4. Design and Implementation Constraints

| Layer | Technology | Rationale |
|---|---|---|
| Frontend | React 18+ | Component-based approach, large community, fast rendering |
| Backend | ASP.NET Core Web API | Cross-platform, integration with Identity, EF Core, LINQ |
| Database | Firebase Firestore | Optional for real-time notifications or session caching |
| Auth | JWT Bearer + ASP.NET Core Identity | Stateless authentication, convenient integration with React |
| API Documentation | Swagger / OpenAPI 3.0 | Automatic generation, endpoint testing |
| Testing | NUnit + Moq | Requirement of the methodological guidelines |
| Code Documentation | DocFX / Doxygen | Generation of HTML pages from XML comments |

### 2.5. Assumptions & Dependencies

| No. | Description | Type | Impact if Violated |
|---|---|---|---|
| A-01 | The team uses Git (GitHub) with a feature-branch workflow and Pull Request reviews | Internal | Version control issues, difficulties with reporting |
| A-02 | LiqPay / Fondy / Stripe provide a sandbox for payment testing | External | Blocking of FR-08 (payment) |
| A-03 | The team follows Scrum: 2-week sprints, daily stand-ups, Kanban board | Methodological | Inability to provide a Scrum report |
| A-04 | All team members have basic knowledge of C#, React, and Firebase | Competency | Need for additional training, risk of schedule delays |

---

## 3. Project Scope

### 3.1. In Scope

- Full e-commerce platform lifecycle: catalog, cart, orders, payment, admin panel
- Registration, authentication, role-based authorization (Customer, Admin)
- Product review and rating system
- Search with filtering and pagination (Elasticsearch)
- Administrative panel with analytics
- Responsive UI (mobile-first)
- Unit tests (NUnit) for business logic
- API documentation (Swagger) and code documentation (DocFX)
- Deployment to hosting (Azure, AWS, or VPS)

### 3.2. Out of Scope

- Native mobile applications (iOS/Android) — web version only (responsive)
- Own payment gateway (acquiring) — integration with third-party providers only
- ML-based product recommendations (can be added as a stretch goal, but not in the MVP)
- Multilingual support beyond ukr/en
- Integration with delivery services (Nova Poshta API) — status simulation only
- Microservice architecture — a modular monolith (Clean Architecture) is used

---

## 4. Functional Requirements

*Format: **[MoSCoW Priority]** ID — Description → Acceptance Criterion*

### 4.1. Authentication and Registration

**[Must] FR-01** — User registration via email + password or OAuth 2.0 (Google).
→ *AC:* Password: min. 8 characters, 1 digit, 1 special character; email confirmation; duplicate emails prohibited; inline validation.

**[Must] FR-02** — Authentication with role separation (Customer / Admin).
→ *AC:* JWT token issued for 24 hours; role passed in claims; access to admin routes protected by the `[Authorize(Roles = "Admin")]` policy.

**[Must] FR-03** — Password recovery via email.
→ *AC:* Reset token valid for 15 minutes and usable once; email sent via SMTP (SendGrid/MailKit).

### 4.2. Product Catalog

**[Must] FR-04** — View a product list with pagination (12 products per page).
→ *AC:* Server-side pagination supported; response time ≤ 300 ms with 1,000+ products.

**[Must] FR-05** — View product details: name, description, price, images (up to 5 photos), category, rating, availability (SKU).
→ *AC:* Image gallery with navigation; display related products from the same category.

**[Must] FR-06** — Search and filter products by name, category, price range, brand, and rating.
→ *AC:* Search is case-insensitive; filter combinations via LINQ / EF Core; empty result — appropriate message displayed.

**[Should] FR-07** — Sort products (price ascending/descending, newest, popular, rating).
→ *AC:* Sorting is performed at the database query level (LINQ `OrderBy` / `OrderByDescending`).

### 4.3. Cart and Checkout

**[Must] FR-08** — Add/remove products from the cart and change quantities.
→ *AC:* Cart is stored in localStorage for guests and in the database for authenticated users; quantity updates without page reload; total amount calculated in real time.

**[Must] FR-09** — Place an order by providing delivery address and contact details.
→ *AC:* Validation of required fields (full name, city, address, phone); order created with status “Pending”.

**[Must] FR-10** — Pay for an order through integration with LiqPay / Fondy / Stripe (sandbox).
→ *AC:* On successful payment, status changes to “Paid”; on rejection — “Payment Failed” with an option to retry; card data is not stored on the server (PCI DSS SAQ A).

**[Must] FR-11** — Order history in the personal account with details and statuses.
→ *AC:* Orders sorted by date (newest first); detailed view of order items.

### 4.4. Reviews and Ratings

**[Should] FR-12** — Leave a review and rating (1–5 stars) only after purchasing a product.
→ *AC:* Verification of a completed order containing this product; one review per product per user; average rating recalculated automatically.

**[Should] FR-13** — View reviews on the product page with pagination.
→ *AC:* Display author name, date, rating, and text; sorting by date / rating.

### 4.5. Administrative Panel

**[Must] FR-14** — CRUD for product categories (hierarchical structure: parent / child).
→ *AC:* Deletion of a category prohibited if it contains products; tree-style display.

**[Must] FR-15** — CRUD for products (name, description, price, images, stock quantity, category).
→ *AC:* Upload images to the server / cloud; format validation (JPEG, PNG) and size (≤ 5 MB); soft delete.

**[Must] FR-16** — Order management: view list, change status (Pending → Paid → Shipped → Delivered / Cancelled).
→ *AC:* Status change history is stored; user notified when status changes (email / SignalR).

**[Must] FR-17** — User management: view, block/unblock, assign roles.
→ *AC:* A blocked user cannot authenticate; reason for blocking is stored.

### 4.6. Personal Account

**[Must] FR-18** — Edit profile (full name, email, phone, delivery address, avatar).
→ *AC:* Email is unique; phone validation using regex; avatar upload ≤ 2 MB.

**[Should] FR-19** — Wishlist.
→ *AC:* Add/remove products; saved for authenticated users; option to move products from Wishlist to cart.

**[Could] FR-20** — Push notifications about order status changes (SignalR / Firebase Cloud Messaging).
→ *AC:* Real-time status updates in the personal account; badge counter for new notifications.

---

## 5. Use Cases

### UC-01: Placing an Order

| | |
|---|---|
| **Actor** | Customer |
| **Preconditions** | User is authenticated; products are in the cart |
| **Main Flow** | 1. User goes to the cart → 2. Checks products and quantities → 3. Clicks “Checkout” → 4. Enters/confirms delivery address → 5. Selects payment method → 6. Confirms payment via LiqPay → 7. Receives confirmation and order number |
| **Alternative Flow** | 6a. Payment rejected → error displayed, return to payment method selection |
| **Postconditions** | Order saved in the database with status “Paid”; stock quantity decreased; history updated |

### UC-02: Adding a Product by an Administrator

| | |
|---|---|
| **Actor** | Administrator |
| **Preconditions** | Administrator is authenticated with the Admin role |
| **Main Flow** | 1. Enters admin panel → 2. Goes to “Products” → 3. Clicks “Add Product” → 4. Fills in the form (name, description, price, category, quantity) → 5. Uploads images → 6. Saves |
| **Alternative Flow** | 5a. Invalid image format → inline error, saving disabled |
| **Postconditions** | Product is available in the catalog; displayed in the admin panel product list |

### UC-03: Blocking a User

| | |
|---|---|
| **Actor** | Administrator |
| **Preconditions** | A violation has been detected (spam, fraud) |
| **Main Flow** | 1. Administrator views the user's profile → 2. Clicks “Block” → 3. Specifies the reason → 4. Confirms |
| **Postconditions** | User cannot log in; receives an email explaining the reason; their listings/products are hidden |

---

## 6. Non-Functional Requirements

### 6.1. Performance

| ID | Requirement | Target | Verification Method |
|---|---|---|---|
| NFR-01 | API response time | p95 < 300 ms for CRUD operations | K6 / Postman / Swagger |
| NFR-02 | Catalog page load time | < 2 s on broadband | Lighthouse CI |
| NFR-03 | Throughput | 100 concurrent users without degradation | Load testing (k6) |

### 6.2. Reliability and Availability

| ID | Requirement |
|---|---|
| NFR-04 | Uptime 99.5% (educational project, not a production SLA) |
| NFR-05 | Graceful degradation: if the payment provider is unavailable, an order can be placed with “Cash on Delivery” payment |
| NFR-06 | Soft delete for all critical entities (users, products, categories) — restoration must be possible |

### 6.3. Security

| ID | Requirement |
|---|---|
| NFR-07 | Protection against OWASP Top 10: XSS (input validation, sanitization), CSRF (JWT in header), SQL Injection (EF Core / parameterized queries), Broken Access Control (policy-based authorization) |
| NFR-08 | Password hashing using ASP.NET Core Identity (PBKDF2 / bcrypt) |
| NFR-09 | JWT tokens with a short lifetime (access: 15 min, refresh: 7 days) or sliding expiration of 24 hours |
| NFR-10 | Rate limiting: no more than 5 login attempts per minute from a single IP |
| NFR-11 | HTTPS (TLS 1.2+) mandatory for all communications |

### 6.4. Scalability

| ID | Requirement |
|---|---|
| NFR-12 | Clean / Onion architecture |
| NFR-13 | Use of async/await and IQueryable to optimize database queries |

### 6.5. Interface and Accessibility

| ID | Requirement |
|---|---|
| NFR-14 | Responsive design: mobile (320px), tablet (768px), desktop (1920px) |
| NFR-15 | Compliance with basic UX principles: readable fonts, sufficient contrast, clear error messages |

### 6.6. Quality and Testing

| ID | Requirement |
|---|---|
| NFR-16 | Unit test coverage (NUnit) ≥ 60% of critical business paths (registration, cart, orders, payment) |
| NFR-17 | Integration tests for API controllers (TestServer / WebApplicationFactory) |
| NFR-18 | Code documentation: XML comments for all public classes and methods; DocFX / Swagger generation |
| NFR-19 | Consistent naming convention (CamelCase for methods, PascalCase for classes, snake_case for the database as desired) — documented in `CONTRIBUTING.md` |

### 6.7. Architectural Requirements (Specific to the Course Project)

| ID | Requirement |
|---|---|
| NFR-20 | Application of design patterns: Repository, Unit of Work, CQRS (optionally with MediatR), Specification |
| NFR-21 | Compliance with SOLID: Single Responsibility for services, Dependency Injection, dependency inversion |
| NFR-22 | Use of LINQ and Fluent API in EF Core |
| NFR-23 | Multithreading: async/await for I/O-bound operations (database queries, external APIs) |
| NFR-24 | Logging: Serilog / NLog with output to file and console; Information / Warning / Error levels |
| NFR-25 | Caching: In-Memory Cache or Redis for frequently requested data (categories, top products) |

---

## 7. Risks

| ID | Risk | Probability | Impact | Mitigation Strategy |
|---|---|---|---|---|
| R-01 | Insufficient team experience with React / ASP.NET Core | Medium | High | Early prototyping; role distribution based on strengths; consultations with the project manager |
| R-02 | Delay in payment provider integration (sandbox unavailable) | Medium | Medium | Develop a mock payment service at an early stage; fallback to “Cash on Delivery” |
| R-03 | Excessive functionality scope (scope creep) | High | Medium | Strict adherence to MoSCoW; focus on Must-have features for the MVP |
| R-04 | Merge conflicts in Git due to unclear domain ownership | Medium | Medium | Divide backend/frontend responsibilities; code review before merge |
| R-05 | Low test coverage before the deadline | Medium | High | Include test writing in the Definition of Done for every sprint; CI gate |

---

## 8. Project Success Metrics

| Metric | Target | Deadline |
|---|---|---|
| MVP Functionality | 100% of Must-have features implemented (FR-01..FR-11, FR-14..FR-18) | Before defense |
| Test Coverage | ≥ 60% unit tests (NUnit) | Before defense |
| Documentation | Swagger UI available; DocFX built | Before defense |
| Deployment | Website deployed and publicly accessible | Before defense |
| Presentation | 7–15 slides, video demo ≤ 2 min | Before defense |
| Git Report | Commit history, branches, merge requests | Throughout the project |
| Scrum Report | 3+ sprints, backlog, Kanban board | Throughout the project |

---

## 9. Appendices

### 9.1. Glossary of Terms

| Term | Definition |
|---|---|
| Soft Delete | Logical deletion (`IsDeleted` flag); physical deletion is not performed |
| SKU | Unique product code used for inventory management |
| Mock | Simulation of an external service for testing without real requests |
| Clean Architecture | Layered architecture with business logic isolated from infrastructure |

### 9.2. Diagrams (for the Next Iteration of the Document)

- [ ] Use Case Diagram (all actors and scenarios)
- [ ] ER Diagram of entities (User, Product, Category, Order, OrderItem, Review, Payment)
- [ ] Sequence Diagram for the payment flow (FR-10)
- [ ] Component Diagram: React → ASP.NET Core Web API → Firebase
- [ ] Deployment Diagram: Azure/AWS/VPS

### 9.3. Requirements Traceability Matrix — Template

| Requirement | Business Goal | Use Case | Test Case (NUnit) | Status |
|---|---|---|---|---|
| FR-01 | User registration | — | `AuthTests.Register_ValidUser_ReturnsOk` | Planned |
| FR-08 | Order placement | UC-01 | `CartTests.AddItem_UpdatesTotal` | Planned |
| FR-10 | Payment | UC-01 | `PaymentTests.ProcessPayment_Success` | Planned |
| FR-15 | Product management | UC-02 | `ProductTests.CreateProduct_Admin_ReturnsCreated` | Planned |

*It is recommended to maintain the matrix in GitHub Projects / Jira with live synchronization.*

### 9.4. Recommended Team Role Distribution (2–4 People)

| Role | Responsibilities | Technologies |
|---|---|---|
| Backend Lead | API architecture, database, business logic, payment integration | C#, ASP.NET Core, EF Core, SQL Server |
| Frontend Lead | UI/UX, React components, state management, API integration | React, Axios, React Router, CSS/SCSS |
| Full-stack / DevOps | Auth, SignalR, deployment, CI/CD, testing | JWT, SignalR, GitHub Actions, Azure |
| QA / Analyst | NUnit tests, SRS, presentation, Scrum reports | NUnit, Moq, Swagger, PowerPoint |
