# PRD: Gastos MVP

## 1. Product Overview

### 1.1 Document title and version
- **PRD:** Gastos MVP  
- **Version:** 1.1

### 1.2 Product summary
Gastos is an AI-driven spending tracker designed for individuals who want a simple, conversational way to manage their expenses.  
The MVP enables users to log, update, and query their spending using a text chat interface powered by OpenAI.  
Multi-account support ensures each user’s data is private and secure, with authentication via email OTP.

The web-based platform focuses on core expense tracking features, prioritizing ease of use, security, and fast onboarding.  
The MVP is strictly chat-only: all expense logging, updating, and querying is performed via chat.  
No other buttons or UI elements for expense management are included at this stage.  
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
| **Embeddings-based semantic search** | High | Every expense and merchant is embedded using OpenAI’s `text-embedding-3-small`. Queries are also embedded to compute similarity for natural-language search. |
| **SEA dictionary seeding** | Medium | Pre-launch script embeds a small list (~100) of regional merchants, dishes, and slang (e.g., Jollibee, Kopitiam, FairPrice, KOI) into a `term_dictionary` table to improve recall and local accuracy. |
| **Facet + tag classification** | High | Rule-based detection for major facets (`food_drink`, `groceries`, `travel`, `other`). Semantic search widens results when rules miss. |
| **User preference toggles** | Medium | Option to include/exclude groceries in food totals. Stored per user. |
| **Learning loop (post-launch)** | Medium | When a new merchant is encountered often, the system auto-embeds it and queues for admin verification before adding to dictionary. |

---

## 5. User Experience

### 5.1 Entry points & first-time flow
- Landing page with login/sign-up via email OTP.  
- Chat prompt greets new users: “Hi! Try logging your first expense (e.g., Lunch 12 at Toast Box).”

### 5.2 Core experience
- **Chat-driven expense management:** users log, update, and query expenses exclusively via chat.  
- The AI confirms actions conversationally (e.g., “Got it — logged Lunch ₱250 at Toast Box”).  
- For low-confidence parses, AI asks clarifying questions.  

### 5.3 Advanced features & edge cases
- Handle ambiguous entries with clarification flow.  
- Prevent duplicates.  
- Support corrections via chat.  
- Detect and tag new merchants for future seeding.  

### 5.4 UI/UX highlights
- Minimal UI: chat only.  
- Responsive layout.  
- Skeleton loaders for chat replies.  
- Tiny info chips (“Food totals exclude groceries — [Include?]”).  

---

## 6. Narrative

A user signs up with email OTP, logs in, and records expenses via chat.  
Gastos parses and stores each entry. When the user asks,  
> “How much did I spend on food?”  
the system embeds the query, references pre-seeded SEA terms (e.g., Kopitiam, Starbucks, Jollibee),  
finds similar expenses, and replies:  
> “You spent ₱265 on food this week (Lunch ₱250, Starbucks ₱15). Groceries excluded.”  
If the user toggles inclusion, Gastos re-runs the query to include grocery expenses.

---

## 7. Success Metrics

### 7.1 User-centric
- Number of expenses logged per user.  
- Time to first expense entry after signup.  
- Frequency of queries.  
- Weekly active users (retention).  

### 7.2 Business
- Number of signups.  
- Conversion rate from signup → first expense.  
- Feedback volume.  

### 7.3 Technical
- Auth success rate.  
- AI parse confidence rate.  
- Embedding latency < 300 ms.  
- Query response time p90 < 500 ms.  
- Error rate < 2%.  

---

## 8. Technical Considerations

### 8.1 Architecture
- **Frontend:** Next.js 15 (App Router), TypeScript, shadcn/ui.  
- **Backend:** Supabase (Postgres + pgvector, Auth, Storage).  
- **AI:** OpenAI GPT + text-embedding-3-small.  
- **Embedding store:** `pgvector` columns for `expenses.raw_embedding` and `term_dictionary.embedding`.  
- **Email OTP:** Supabase Auth or custom mailer.  

### 8.2 Data storage & privacy
- Money as integer cents.  
- `timestamptz` stored per user’s timezone.  
- Per-user row-level security.  
- No sensitive data in logs.  

### 8.3 SEA seeding
- One-time script run before first deployment:  
  - Embeds ~100 SEA merchants/dishes into `term_dictionary`.  
  - Example entries: FairPrice → groceries, Jollibee → food_drink, Grab → travel.  
- Enables Gastos to interpret SEA-specific merchants correctly from day one.  

### 8.4 Scalability & performance
- Hundreds of concurrent users.  
- Vector index (`ivfflat`) for fast semantic search.  
- Cached embeddings to minimize cost.  

### 8.5 Challenges
- Handling low-confidence parses.  
- Maintaining predictable results despite semantic fuzziness.  
- Preventing duplicate expense entries.  
- AI cost control and caching.  

---

## 9. Milestones & Sequencing

| Phase | Duration | Key Deliverables |
|-------|-----------|------------------|
| **Phase 1** | 1 week | Email OTP login, session management |
| **Phase 2** | 2 weeks | Chat UI, OpenAI integration, expense parsing, embedding storage |
| **Phase 3** | 2 weeks | Update & query expenses, embedding-based semantic search |
| **Phase 4** | 1 week | SEA seeding script, tests, polish, performance, feedback |

**Total:** ~6 weeks | **Team:** 2–3 (FE/BE developer + PM + Designer)

---

## 10. User Stories

### 10.1 GH-001  Sign-up and login with email OTP
**Description:** As a user, I want to sign up and log in securely using an email-based one-time password.  
**Acceptance:**  
- OTP request → email.  
- Valid OTP grants access.  
- Authenticated users only can log expenses.

---

### 10.2 GH-002  Log expenses via text chat
**Description:** As a user, I want to record expenses using natural language.  
**Acceptance:**  
- AI parses amount, merchant, tags, timestamp.  
- Expense stored with embedding vector.  
- Confirmation message shown.  
- Low-confidence entries trigger clarification.

---

### 10.3 GH-003  Update recent expenses via chat
**Description:** As a user, I want to correct recent expenses.  
**Acceptance:**  
- Can reference a recent expense in chat.  
- Update amount, merchant, tags, or date.  
- Changes confirmed in chat.

---

### 10.4 GH-004  Query spending via text chat
**Description:** As a user, I want to ask questions about my spending in natural language.  
**Acceptance:**  
- User can query totals by time or facet (e.g., “food last week”).  
- Query embedded and compared semantically to expenses.  
- SEA dictionary anchors help resolve local merchants.  
- User preference (`include_groceries_in_food`) respected.  
- Response summarizes spending clearly in chat.

---

### 10.5 GH-005  SEA seeding and continuous learning *(system story)*
**Description:** As the system, I must embed regional terms before launch to improve semantic accuracy for SEA users.  
**Acceptance:**  
- `term_dictionary` seeded with at least 100 local merchants/dishes/slang.  
- Each term embedded once using OpenAI embeddings.  
- Runtime auto-learn adds new merchants after admin validation.

---

### 10.6 GH-006  Semantic search fallback
**Description:** As a user, when I ask a vague or uncommon query, Gastos should still return relevant expenses using semantic similarity.  
**Acceptance:**  
- If rule-based match fails, vector search widens results.  
- Similarity threshold configurable (default 0.8).  
- System logs which path matched (`rule` / `semantic`).  

---

## 11. Appendix

### 11.1 Glossary
- **Facet:** broad category (food_drink, groceries, travel, etc.).  
- **Embedding:** numeric vector representing text meaning.  
- **Seed dictionary:** curated list of local merchants/dishes pre-embedded to guide semantic search.  
- **pgvector:** Postgres extension enabling vector similarity queries.  

---

**End of PRD v1.1**
