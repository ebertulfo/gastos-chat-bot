# PRD: Gastos MVP

## 1. Product overview

### 1.1 Document title and version

* PRD: Gastos MVP
* Version: 1.0

### 1.2 Product summary

Gastos is an AI-driven spending tracker designed for individuals who want a simple, conversational way to manage their expenses. The MVP enables users to log, update, and query their spending using a text chat interface powered by OpenAI. Multi-account support ensures each user’s data is private and secure, with authentication via email OTP.

The web-based platform focuses on core expense tracking features, prioritizing ease of use, security, and fast onboarding. The MVP is strictly chat-only: all expense logging, updating, and querying is performed via the chat interface. No other buttons or UI elements for expense management are included at this stage. Advanced analytics, integrations, and compliance are out of scope for the MVP.

## 2. Goals

### 2.1 Business goals

* Launch a functional MVP to validate demand for AI-driven expense tracking.
* Minimize development time and complexity.
* Ensure user data is secure and private.
* Collect feedback for future feature development.

### 2.2 User goals

* Quickly log expenses via chat.
* Easily update recent expenses via chat.
* Query spending with natural language via chat.
* Sign up and log in securely.

### 2.3 Non-goals

* No analytics/charts, budgets, or bank sync.
* No mobile or native apps in MVP.
* No multi-currency or compliance features.
* No non-chat UI for expense management (no buttons, forms, or inline edits).

## 3. User personas

### 3.1 Key user types

* Individual users tracking personal expenses.

### 3.2 Basic persona details

* **Budget-conscious individual**: Wants a fast, easy way to track spending and review expenses.

### 3.3 Role-based access

* **User**: Can log, update, and query their own expenses via chat. Cannot access other users’ data.
* **Admin (future)**: Not included in MVP.

## 4. Functional requirements

* **Email OTP authentication** (Priority: High)
  * Users can sign up and log in using email-based one-time passwords.
  * Secure session management.

* **Expense logging via chat** (Priority: High)
  * Users can enter expenses in natural language (e.g., "Lunch 250 at Toast Box") via chat only.
  * AI parses and records amount, merchant, tags, and timestamp.

* **Update recent expenses via chat** (Priority: High)
  * Users can edit amount, merchant, tags, or timestamp for recent expenses using chat commands only.

* **Query spending via chat** (Priority: High)
  * Users can ask questions like "How much did I spend this month?" or "Spending on food last week?" via chat only.
  * AI returns summarized results based on user’s query.

## 5. User experience

### 5.1 Entry points & first-time user flow

* Landing page with login/sign-up via email OTP.
* Onboarding chat prompt for first expense entry.

### 5.2 Core experience

* **Chat-driven expense management**: Users interact exclusively with a chat interface to log, update, and query expenses.
  * Ensures a frictionless, conversational experience.

### 5.3 Advanced features & edge cases

* Handle ambiguous expense entries with AI clarification.
* Prevent duplicate expense logging.
* Support for correcting misparsed entries via chat.

### 5.4 UI/UX highlights

* Minimal UI: chat interface only for all expense actions.
* Responsive, accessible design.
* Skeleton loaders for fast feedback.

## 6. Narrative

A user signs up with their email, receives an OTP, and logs in. They enter expenses in chat, and Gastos parses and records each entry. If the AI is unsure, it asks for clarification. Users can update recent expenses and query their spending history conversationally, all within a secure, web-based chat interface.

## 7. Success metrics

### 7.1 User-centric metrics

* Number of expenses logged per user.
* Time to first expense entry after signup.
* Frequency of expense queries.
* User retention (weekly active users).

### 7.2 Business metrics

* Number of signups.
* Conversion rate from signup to first expense.
* Feedback submissions.

### 7.3 Technical metrics

* Authentication success rate.
* AI parse confidence rate.
* API response latency (p50 < 200ms, p90 < 500ms).
* Error rate (auth, parse, write).

## 8. Technical considerations

### 8.1 Integration points

* OpenAI API for chat parsing.
* Email service for OTP delivery.
* Database for expense and user data.

### 8.2 Data storage & privacy

* Store money as integer cents.
* Timestamps as timestamptz, converted by user timezone.
* Enforce per-user data isolation.
* No sensitive data stored in logs.

### 8.3 Scalability & performance

* Support hundreds of concurrent users.
* Fast chat response and expense write times.
* Lazy-load non-critical UI components.

### 8.4 Potential challenges

* Handling ambiguous or poorly formatted expense entries.
* Preventing duplicate expense logs.
* Ensuring secure authentication and session management.
* Managing AI API costs and rate limits.

## 9. Milestones & sequencing

### 9.1 Project estimate

* Size: Small
* Estimate: 4-6 weeks

### 9.2 Team size & composition

* Team size: 2-3
* Roles: Full-stack developer, product manager, designer

### 9.3 Suggested phases

* **Phase 1**: Authentication & onboarding (1 week)
  * Email OTP login, user session management
* **Phase 2**: Chat-driven expense logging (2 weeks)
  * Chat UI, OpenAI integration, expense parsing & storage
* **Phase 3**: Expense update & query (2 weeks)
  * Edit recent expenses, query spending via chat
* **Phase 4**: Polish & test (1 week)
  * Accessibility, error handling, performance, user feedback

## 10. User stories

### 10.1. User can sign up and log in with email OTP

* **ID**: GH-001
* **Description**: As a user, I want to sign up and log in securely using an email-based one-time password so that my account and data are protected.
* **Acceptance criteria**:
  * User can request an OTP to their email.
  * User can enter OTP to authenticate.
  * Session is securely managed.
  * Only authenticated users can access expense features.

### 10.2. User can log expenses via text chat

* **ID**: GH-002
* **Description**: As a user, I want to log expenses by sending natural language messages in chat so that I can quickly record my spending.
* **Acceptance criteria**:
  * User can enter expense details in chat.
  * AI parses and records amount, merchant, tags, and timestamp.
  * User receives confirmation of logged expense.
  * Low-confidence parses prompt for clarification.

### 10.3. User can update recently created expenses via text chat

* **ID**: GH-003
* **Description**: As a user, I want to update recent expenses using chat so that I can correct mistakes or add details.
* **Acceptance criteria**:
  * User can select or reference a recent expense in chat.
  * User can update amount, merchant, tags, or timestamp via chat only.
  * Changes are confirmed in chat.

### 10.4. User can query their spending via text chat

* **ID**: GH-004
* **Description**: As a user, I want to ask questions about my spending in chat so that I can review my expenses and track patterns.
* **Acceptance criteria**:
  * User can ask queries like "How much did I spend this month?" or "Spending on food last week?" via chat only.
  * AI returns summarized results based on query.
  * User can filter by time period, tags, or merchant via chat only.
