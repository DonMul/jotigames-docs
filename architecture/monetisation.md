# Monetisation & Subscriptions

## Overview

JotiGames supports an optional monetisation layer controlled by the `ENABLE_MONETISATION` environment variable (default: `true`). When disabled, all platform features are free with unlimited minutes.

## Subscription Tiers

| Slug       | Name      | Minutes/month | Price   | Stripe recurring |
|------------|-----------|--------------|---------|-----------------|
| `free`     | Free      | 600          | €0.00   | No              |
| `beginner` | Beginner  | 3,000        | €9.99   | Yes             |
| `pro`      | Pro       | 10,000       | €24.99  | Yes             |
| `unlimited`| Unlimited | Unlimited    | €49.99  | Yes             |

Plans are seeded via the super admin panel ("Seed Defaults" button) or `SubscriptionService.seed_default_plans()`.

## Default Plan for New Users

Super admins can designate one plan as the **default** for new user registrations. When set, any user who registers (via the public registration form **or** admin-created accounts) is automatically subscribed to the default plan.

- Configured in the admin panel: **Subscriptions → Default Plan for New Users**
- Stored as `is_default` boolean column on `subscription_plan` (exactly one plan can be default)
- Only active plans can serve as defaults
- Auto-subscription is best-effort and never blocks the registration flow
- Setting `No default plan` clears the flag; new users register without a subscription
- The default plan badge is visible in the plans table

The public pricing page (`/pricing`) also shows the default package explicitly and includes a minute calculator (teams × hours) that recommends a package with safety margin.

## Game Minutes

- Each active game (start_at ≤ now ≤ end_at) consumes **1 minute per team per tick** from the game owner's balance.
- Games owned by users with `ROLE_SUPER_ADMIN` are excluded from minute ticking.
- Balance is tracked per user per calendar month (`minute_balance` table).
- When subscription balance is exhausted, top-up minutes are consumed (oldest expiry first).
- Unlimited plans skip balance checks entirely.

## Top-up Packages

One-off minute packages purchasable by users:
- 12-month validity from purchase date
- Consumed only after subscription balance is fully used
- Managed via super admin panel

## Stripe Integration

- **Subscriptions**: Stripe recurring subscriptions for paid tiers (beginner, pro, unlimited)
- **Top-ups**: Stripe PaymentIntent for one-off purchases
- **Frontend**: Subscribe/change-plan actions call backend directly and then redirect to Stripe Checkout (`result.payment_url`) when immediate payment is required. `@stripe/stripe-js` + `@stripe/react-stripe-js` `<CardElement>` remains for top-up card collection.
- **Webhooks**: `POST /api/subscription/webhook/stripe` handles:
  - `checkout.session.completed` → link Stripe subscription to local record, allocate minutes
  - `invoice.payment_succeeded` → transition matching backend payment from `pending` to `succeeded`
  - `invoice.payment_failed` → transition matching backend payment from `pending` to `failed` and rollback subscription plan when it was a failed paid-plan switch
  - `payment_intent.succeeded` → transition top-up payment `pending` → `succeeded`
  - `payment_intent.payment_failed` → transition top-up payment `pending` → `failed`
  - `customer.subscription.deleted` → mark cancelled
  - `customer.subscription.updated` → sync status
  - Endpoint is intentionally visible in OpenAPI/Swagger for easier operational setup and testing.

### Payment state lifecycle

- On payment initialization (subscription checkout, paid upgrade invoice, top-up intent), backend creates a `payment_record` with `status=pending`.
- Stripe webhooks are the source of truth for terminal state:
  - success callback updates pending record to `succeeded`.
  - failure callback updates pending record to `failed`.
- If Stripe invoice payment fails during a plan switch upgrade, backend restores the user to the previous tier (rollback) and refreshes local minute allocation for that restored plan.

### Upgrade vs downgrade behavior

- **Upgrade (free→paid or paid→paid higher tier)**:
  - **Free→paid or new subscription**: A Stripe Checkout Session (`mode=subscription`) is created. Backend returns `result.payment_url` (Stripe-hosted checkout URL). Frontend redirects the user to Stripe where they pick a payment method and complete payment. The `checkout.session.completed` webhook links the resulting Stripe subscription to the local record.
  - **Paid→paid upgrade**: Backend modifies the existing Stripe subscription with `proration_behavior=always_invoice`. The hosted invoice URL is returned as `payment_url` for immediate payment.
  - **Missing default payment method during paid→paid change**: Backend returns a Stripe Billing Portal payment-method-update URL. The same existing Stripe subscription is kept (no replacement subscription is created); user updates payment method and retries change-plan.
  - **Missing local Stripe subscription link**: When a user is on a paid plan but local `stripe_subscription_id` is empty, backend first reconciles against Stripe customer subscriptions and reuses the existing Stripe subscription via `Subscription.modify` instead of creating a new subscription.
- **Subscription payment method configuration**:
  - Checkout Sessions specify `payment_method_types` to expose multi-method checkout.
  - Configured set: `card` (includes Cartes Bancaires), `link`, `paypal`, `bancontact`, `ideal` (iDEAL | Wero), `sepa_debit`.
  - For non-EUR currencies, EUR-local methods (`bancontact`, `ideal`, `sepa_debit`) are omitted automatically.
- **Downgrade (paid→paid lower tier)**:
  - Stripe subscription item is changed with `proration_behavior=none`.
  - No immediate payment redirect; new amount applies on next renewal.
- **Paid→free**:
  - Stripe subscription is canceled on Stripe and local subscription plan is switched to free.

### Configuration

```env
ENABLE_MONETISATION=true
STRIPE_SECRET_KEY=sk_live_...
STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_CURRENCY=eur
```

## API Endpoints

### User-facing (`/api/subscription/...`)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/status` | None | Monetisation status + Stripe publishable key |
| GET | `/plans` | None | List active subscription plans |
| GET | `/topup-packages` | None | List active top-up packages |
| GET | `/me` | User | Current subscription summary |
| POST | `/subscribe` | User | Subscribe to a plan |
| POST | `/change-plan` | User | Upgrade/downgrade plan |
| POST | `/cancel` | User | Cancel subscription |
| POST | `/reactivate` | User | Reactivate pending cancellation |
| POST | `/topup` | User | Purchase minute top-up |
| GET | `/my-payments` | User | User payment history |
| POST | `/webhook/stripe` | Stripe sig | Stripe webhook handler |

### User profile self-service (`/api/auth/...`)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/me` | User | Current user's profile (email, username, principal_id) |
| PUT | `/me` | User | Update email and/or display name |
| PUT | `/me/password` | User | Change password (requires current password) |

### Super admin (`/api/super-admin/subscription/...`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/plans` | List all plans (including inactive) |
| POST | `/plans` | Create plan |
| PUT | `/plans/reorder` | Bulk reorder plans (accepts `plan_ids` array) |
| PUT | `/plans/:id` | Update plan (slug, name, minutes, price, active, stripe_price_id) |
| POST | `/seed-plans` | Seed default plans |
| GET | `/default-plan` | Get default plan for new users |
| PUT | `/default-plan` | Set/clear default plan for new users |
| GET | `/topup-packages` | List all top-up packages |
| POST | `/topup-packages` | Create top-up package |
| PUT | `/topup-packages/:id` | Update top-up package |
| GET | `/subscriptions` | List all user subscriptions |
| GET | `/users/:userId` | View user subscription details |
| GET | `/revenue` | Revenue statistics |
| GET | `/payments` | Payment records log |

### Plan Management

Super admins can:
- **Enable/disable plans**: Toggle `is_active` via the admin UI switch. Disabled plans are hidden from the public pricing page and cannot be purchased, but existing subscribers keep their subscription unchanged.
- **Edit plans inline**: Click the pencil icon to edit slug, name, minutes/month, price, Stripe price ID, and active status. Changes apply on next billing cycle for existing subscribers.
- **Reorder plans**: Use arrow buttons to change display order. The `PUT /plans/reorder` endpoint accepts an ordered array of plan IDs and updates `sort_order` accordingly.
- **Slug uniqueness**: Slug changes are validated for uniqueness; a 409 Conflict is returned on duplicates.

`/subscription/revenue` includes:
- `summary`: historical succeeded payments grouped by payment type/currency
- `projected`: estimated subscription revenue for renewals due in the next 30 days

## Database Tables

Created by migration `20260325_0005_create_monetisation_tables.py`:

1. **subscription_plan** – tier definitions (includes `is_default` flag, added by `20260326_0006`)
2. **user_subscription** – user↔plan binding with Stripe IDs
3. **minute_balance** – monthly minute allocation per user
4. **minute_topup_package** – purchasable one-off packages
5. **minute_topup_purchase** – user purchases with 12-month expiry
6. **payment_record** – immutable payment audit trail
7. **minute_usage_log** – consumption audit per game

## Cron Jobs

| Script | Schedule | Purpose |
|--------|----------|---------|
| `backend/scripts/tick_game_minutes.py` | Every 60s | Deduct minutes for active games |

`renew_subscriptions.py` remains available for manual/reconciliation runs, but is not part of the managed `system/cron/crontab.template` schedule.

`tick_game_minutes.py` passes `owner_id`, `game_id`, deducted `minutes`, and `team_count` into `SubscriptionService.consume_minutes(...)` so minute usage logs stay attributable per game/team snapshot.

Both scripts support `--loop` mode for continuous operation or single-pass for crontab.

## Cancellation & Grace

- **At period end** (default): Subscription stays active until `current_period_end`, then cancelled
- **Immediate**: Subscription cancelled instantly, Stripe subscription deleted
- **Reactivation**: Users can reverse a pending cancellation before period end
- **Past due**: On payment failure, subscription marked `past_due` (not immediately cancelled)

## Architecture

```
backend/app/
├── models.py                           # ORM models (7 new classes)
├── config.py                           # 5 monetisation settings
├── modules/subscription.py             # User-facing API module
├── modules/super_admin.py              # Admin subscription endpoints
├── repositories/subscription_repository.py  # Data access layer
├── services/subscription_service.py    # Business logic + Stripe
backend/scripts/
├── tick_game_minutes.py                # Minute consumption cron
├── renew_subscriptions.py              # Manual/reconciliation renewal utility
backend/tests/
├── test_subscription_full_unit.py      # 103 unit tests
admin/src/
├── pages/SubscriptionsPage.jsx         # Admin panel subscription management
├── lib/api.js                          # subscriptionApi methods
frontend/src/
├── pages/admin/SubscriptionPage.jsx    # User-facing subscription page (Stripe Elements)
├── lib/api.js                          # gameApi subscription methods
frontend/src/
├── pages/admin/SubscriptionPage.jsx    # User subscription management (also re-exported for /account/subscription)
├── pages/account/ProfilePage.jsx       # User profile: email, display name, password change
├── pages/account/AccountSubscriptionPage.jsx  # Re-export of SubscriptionPage for account layout
├── pages/account/PaymentsPage.jsx      # Full-page payment history with pagination
├── components/AccountLayout.jsx        # Sidebar layout for /account/* routes
├── lib/api.js                          # Subscription + profile API methods
```
