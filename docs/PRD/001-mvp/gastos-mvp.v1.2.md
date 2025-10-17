# PRD: Gastos MVP

## 1. Product Overview

### 1.1 Document title and version
- **PRD:** Gastos MVP  
- **Version:** 1.2  
- **Previous:** [v1.1](./Gastos_PRD_v1.1.md)  
- **Date:** 2025-10-17

### 1.2 Product summary
Gastos is an AI-driven spending tracker designed for individuals who want a simple, conversational way to manage their expenses.  
The MVP enables users to log, update, and query their spending using a text chat interface powered by OpenAI.  
Multi-account support ensures each user’s data is private and secure, with authentication via email OTP.

The web-based platform focuses on core expense tracking features, prioritizing ease of use, security, and fast onboarding.  
The MVP is strictly chat-only: all expense logging, updating, and querying is performed via chat.  
Advanced analytics, integrations, and compliance are out of scope for the MVP.

---

## 2. Goals

### 2.1 Business goals
- Launch a functional MVP to validate demand for AI-driven expense tracking.  
- Minimize development time and complexity.  
- Ensure user data is secure and private.  
- Collect feedback for future feature development.  

### 2.2 User goals
- Quickly log expenses via chat.  
- Easily update recent expenses via chat.  
- Query spending with natural language via chat.  
- Sign up and log in securely.  

### 2.3 Non-goals
- No analytics/charts, budgets, or bank sync.  
- No mobile or native apps in MVP.  
- No multi-currency or compliance features.  
- No non-chat UI for expense management (no buttons, forms, or inline edits).

---

## 3. User Personas

### 3.1 Key user types
- Individual users tracking personal expenses.

### 3.2 Basic persona details
**Budget-conscious individual:** wants a fast, easy way to track spending and review expenses.

### 3.3 Role-based access
- **User:** can log, update, and query their own expenses via chat.  
- **Admin (future):** for data insights and merchant tagging; not included in MVP.

---

## 4. Functional Requirements

### Core features
| Feature | Priority | Description |
|----------|-----------|-------------|
| **Email OTP authentication** | High | Users sign up and log in using email-based OTP. Secure session management. |
| **Expense logging via chat** | High | Users enter natural language expenses (e.g. “Lunch 250 at Toast Box”). AI parses amount, merchant, tags, and timestamp. |
| **Update recent expenses via chat** | High | Users can modify recent expenses through natural language (e.g. “change my lunch to 300”). |
| **Query spending via chat** | High | Users ask questions like “How much did I spend this week?” or “How much on food last month?” and receive summarized results. |

### AI and Data Intelligence
| Capability | Priority | Description |
|-------------|-----------|-------------|
| **Embeddings-based semantic search** | High | Every expense and merchant is embedded using OpenAI’s `text-embedding-3-small`. Queries are embedded to compute similarity for natural-language search. |
| **Facet + tag classification (12-facet system)** | High | Gastos uses a lightweight 12-facet taxonomy based on Plaid, COICOP, MCC, and Monzo standards. Each expense receives a *soft facet* (a best guess) to boost precision and enable clear toggles (“exclude groceries”). Queries still use semantic widening for flexibility. |
| **SEA dictionary seeding** | Medium | Pre-launch script embeds ~100 SEA merchants/dishes/slang (e.g., Jollibee, Kopitiam, FairPrice, KOI) into a `term_dictionary` table for regional context. |
| **User preference toggles** | Medium | Option to include/exclude groceries in food totals. Stored per user. |
| **Learning loop (post-launch)** | Medium | When a new merchant appears often, the system auto-embeds and queues it for admin review before adding to dictionary. |

---

## 5. Facet Definitions

Gastos v1.2 uses 12 high-level facets aligned with proven taxonomies (Plaid PFC, COICOP, MCC, Monzo):

| # | Facet | Description | Examples |
|---|--------|--------------|-----------|
| 1 | **Food & Drink (Out)** | Meals, cafés, coffee, snacks, restaurants. | Starbucks, Toast Box, McDonald’s, Jollibee |
| 2 | **Groceries** | Supermarkets, convenience stores, wet markets. | FairPrice, Don Don Donki, 7-Eleven, Coles |
| 3 | **Transport (Local)** | Rides, public transport, fuel, parking, tolls. | Grab, Gojek, MRT, Myki, petrol |
| 4 | **Travel (Long-distance)** | Flights, hotels, visas, intercity trips. | SIA, Booking.com, Airbnb |
| 5 | **Shopping (Retail)** | Apparel, electronics, home goods, marketplaces. | Lazada, Shopee, Uniqlo |
| 6 | **Bills & Subscriptions** | Utilities, telecom, streaming, SaaS. | Netflix, Singtel, StarHub, iCloud |
| 7 | **Housing & Utilities** | Rent, mortgage, water, electricity, gas. | HDB, SP Group |
| 8 | **Health & Pharmacy** | Clinics, pharmacies, insurance. | Mercury Drug, Guardian, Watsons |
| 9 | **Entertainment & Recreation** | Movies, events, gyms, hobbies, games. | Golden Village, Steam, ClassPass |
|10 | **Education** | Tuition, courses, books, learning apps. | Udemy, Coursera |
|11 | **Financial & Fees** | Bank fees, interest, FX, cash withdrawal. | DBS fee, PayPal FX |
|12 | **Other / Misc** | Uncategorised or low-frequency items. | Gifts, donations |

*Rules:*
- Deterministic matching via keyword/merchant map (priority: `groceries > food_drink > transport > travel > other`).  
- Always embed `raw_text` and extract tags; facets are hints, not hard categories.  
- Users can correct facets inline (“Looks like Food. [Change]”), which updates this expense and future defaults for that merchant.  

---

## 6. User Experience

### 6.1 Entry & onboarding
- Landing page with email OTP sign-up/login.  
- Onboarding chat prompt for first expense entry.  

### 6.2 Core experience
- Chat-only expense management: log, update, and query.  
- AI clarifies low-confidence inputs.  
- Inline correction chips for facets.

### 6.3 UX highlights
- Minimal chat UI.  
- Responsive layout.  
- Skeleton loaders.  
- Info chip: “Food totals exclude groceries — [Include?]”.

---

## 7. Technical Considerations

### 7.1 Architecture
- **Frontend:** Next.js 15, TypeScript, shadcn/ui.  
- **Backend:** Supabase (Postgres + pgvector, Auth, Storage).  
- **AI:** OpenAI GPT + text-embedding-3-small.  
- **Embedding store:** `pgvector` columns for `expenses.raw_embedding` and `term_dictionary.embedding`.  

### 7.2 Data model
| Table | Key Fields | Purpose |
|--------|-------------|----------|
| `expenses` | `amount`, `raw_text`, `facet`, `raw_embedding`, `user_id` | Core expense data. |
| `expense_tags` | `expense_id`, `tag` | Free-form keyword tags. |
| `term_dictionary` | `term`, `facet`, `scope`, `embedding` | Seeded anchor terms for regional context. |
| `merchant_preferences` | `user_id`, `merchant`, `facet` | Per-user overrides learned from corrections. |

### 7.3 Seeding process
- One-time script before deployment: embed ~100 regional merchants/slang per scope (e.g., SEA, AU).  
- Stored in `term_dictionary`.  
- Query-time: restrict search to `scope IN ('global', user_scope)`.

### 7.4 Alignment & extensibility
- Facet taxonomy aligns with Plaid PFC, COICOP, and MCC codes for future bank-sync mapping.  
- Region expansion (e.g., Australia) = add new seed file (`au_terms.ts`) + run embed script; no schema change.  
- Optional future: allow user-defined custom facets built atop tags.

### 7.5 Scalability & performance
- ANN index (`ivfflat`) on embeddings for fast search.  
- Cached embeddings to limit API cost.  
- Facet prefiltering reduces vector workload by ~80–90%.

### 7.6 Challenges
- Handling ambiguous merchants across regions.  
- Maintaining predictable “Food vs. Groceries” behavior.  
- Balancing semantic flexibility with deterministic clarity.

---

## 8. Milestones

| Phase | Duration | Key Deliverables |
|-------|-----------|------------------|
| **1. Auth & Onboarding** | 1 week | Email OTP, session management. |
| **2. Chat + Logging** | 2 weeks | Chat UI, OpenAI parsing, embedding storage. |
| **3. Update & Query** | 2 weeks | Edit expenses, semantic search, facet scoring. |
| **4. Seeding + Polish** | 1 week | Regional seed script, tests, performance tuning. |

**Total:** ~6 weeks | **Team:** 2–3 (Full-stack Dev, PM, Designer)

---

## 9. User Stories

### GH-001 Sign up / login via email OTP
*As a user, I want to sign up and log in securely using an email-based one-time password.*

### GH-002 Log expenses via chat
*As a user, I want to log expenses conversationally.*  
- AI parses amount, merchant, tags, timestamp.  
- Stores embedding and soft facet.

### GH-003 Update recent expenses
*As a user, I want to fix mistakes or add details via chat.*

### GH-004 Query spending
*As a user, I want to ask “How much did I spend on food?” or “Spending on transport last week?”*  
- Queries embed intent → match via facets + similarity.  
- Respects “include groceries” preference.  
- Uses seeded anchors for local accuracy.

### GH-005 SEA seeding and continuous learning
*As the system, I seed key SEA merchants/dishes before launch and learn new ones post-launch.*

### GH-006 Facet correction memory
*As a user, when I correct a facet, Gastos remembers for next time.*

---

## 10. Appendix

### 10.1 Glossary
- **Facet:** broad, soft category label for quick filtering and explainability.  
- **Embedding:** numeric vector representing meaning of text.  
- **Seed dictionary:** curated set of region-specific anchors pre-embedded to guide search.  
- **pgvector:** Postgres extension for vector similarity.  
- **MCC / COICOP / Plaid PFC:** standard category systems Gastos aligns with.

---

**End of PRD v1.2**
