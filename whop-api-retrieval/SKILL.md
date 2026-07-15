---
name: whop-api-retrieval
description: "Query Whop's REST API for members, memberships, payments, products, and other business data. Maps natural language questions to the right API calls."
---

# Whop API Retrieval Skill

Query Whop's v1 REST API for business data. Loads `WHOP_API_KEY` from `~/.hermes/.env`.

## Auth Setup

```bash
source ~/.hermes/.env
# API key in WHOP_API_KEY; pass as: -H "Authorization: Bearer $WHOP_API_KEY"
```

**Base URL:** `https://api.whop.com/api/v1`

All list endpoints use **cursor-based pagination** with `first`, `after`, `before`, `last`. Most support `order`, `direction` (asc/desc), and `created_before`/`created_after` filters.

---

## Endpoints Reference

### 1. Members (`/members`)

**List** `GET /members` → `{data: [...], page_info: {...}}`
**Retrieve** `GET /members/{id}` → single member

| Filter | Type | Description |
|--------|------|-------------|
| `company_id` | string | Company ID to scope |
| `access_level` | enum | `no_access`, `admin`, `customer` |
| `statuses` | string[] | `drafted`, `joined`, `left` |
| `product_ids` | string[] | Filter by product |
| `plan_ids` | string[] | Filter by plan |
| `user_ids` | string[] | Filter by user |
| `promo_code_ids` | string[] | Filter by promo code used |
| `most_recent_actions` | string[] | `canceling`, `churned`, `paid_subscriber`, `trialing`, `renewing`, `past_due`, `left`, `joined`, `expiring`, `paused`, `finished_split_pay`, `paid_once`, `drafted`, `pending_entry` |
| `query` | string | Search name/username/email |
| `created_after`/`created_before` | ISO datetime | Time range |

**Sortable columns:** `id`, `usd_total_spent`, `created_at`, `joined_at`, `most_recent_action`

```bash
# 5 most recent members
curl -s "https://api.whop.com/api/v1/members?first=5&order=created_at&direction=desc" \
  -H "Authorization: Bearer $WHOP_API_KEY" | jq

# Active customers (joined status, customer access)
curl -s "https://api.whop.com/api/v1/members?first=100&statuses=joined&access_level=customer" \
  -H "Authorization: Bearer $WHOP_API_KEY" | jq

# Search member by email/username
curl -s "https://api.whop.com/api/v1/members?first=10&query=sensussy" \
  -H "Authorization: Bearer $WHOP_API_KEY" | jq
```

---

### 2. Memberships (`/memberships`)

**List** `GET /memberships`
**Retrieve** `GET /memberships/{id}` — ID can be a membership ID or license key

| Filter | Type | Description |
|--------|------|-------------|
| `company_id` | string | **Required** when using API key |
| `product_ids` | string[] | Filter by product |
| `plan_ids` | string[] | Filter by plan |
| `user_ids` | string[] | Filter by user |
| `statuses` | string[] | `trialing`, `active`, `past_due`, `completed`, `canceled`, `expired`, `unresolved`, `drafted`, `canceling` |
| `cancel_options` | string[] | Filter by cancellation reason |
| `promo_code_ids` | string[] | Filter by promo code |
| `created_after`/`created_before` | ISO datetime | Time range |

**Sortable columns:** `id`, `created_at`, `status`, `canceled_at`, `date_joined`, `total_spend`

```bash
# Active memberships
curl -s "https://api.whop.com/api/v1/memberships?first=100&statuses=active" \
  -H "Authorization: Bearer $WHOP_API_KEY" | jq

# Memberships for a specific user
curl -s "https://api.whop.com/api/v1/memberships?first=50&user_ids=user_XXXXXXXXX" \
  -H "Authorization: Bearer $WHOP_API_KEY" | jq

# Find users with multiple active memberships (post-process in jq)
```

---

### 3. Companies (`/companies`)

**List** `GET /companies`
**Retrieve** `GET /companies/{id}` — ID or route slug

| Filter | Type | Description |
|--------|------|-------------|
| `parent_company_id` | string | List connected accounts under a platform |
| `created_after`/`created_before` | ISO datetime | Time range |

```bash
# Your companies
curl -s "https://api.whop.com/api/v1/companies?first=10" \
  -H "Authorization: Bearer $WHOP_API_KEY" | jq
```

---

### 4. Products (`/products`)

**List** `GET /products` — **requires** `company_id`
**Retrieve** `GET /products/{id}` — ID or route slug

| Filter | Type | Description |
|--------|------|-------------|
| `company_id` | string | **Required** |
| `product_types` | string[] | `regular`, `app`, `experience_upsell`, `api_only` |
| `visibilities` | string[] | Visibility filter |

**Sortable columns:** created_at (default)

```bash
# Products for your company
curl -s "https://api.whop.com/api/v1/products?first=50&company_id=biz_XXXXXXXXX" \
  -H "Authorization: Bearer $WHOP_API_KEY" | jq
```

---

### 5. Experiences (`/experiences`)

**List** `GET /experiences` — **requires** `company_id`

| Filter | Type | Description |
|--------|------|-------------|
| `company_id` | string | **Required** |
| `product_id` | string | Filter by product |
| `app_id` | string | Filter by app |

```bash
curl -s "https://api.whop.com/api/v1/experiences?first=50&company_id=biz_XXXXXXXXX" \
  -H "Authorization: Bearer $WHOP_API_KEY" | jq
```

---

### 6. Payments (`/payments`)

**List** `GET /payments`
**Retrieve** `GET /payments/{id}`

| Filter | Type | Description |
|--------|------|-------------|
| `company_id` | string | Company scope |
| `product_ids` | string[] | Filter by product |
| `plan_ids` | string[] | Filter by plan |
| `billing_reasons` | string[] | Billing reason filter |
| `currencies` | string[] | Currency filter |

**Sortable columns:** `final_amount`, `created_at`, `paid_at`

```bash
# Recent payments
curl -s "https://api.whop.com/api/v1/payments?first=20&order=created_at&direction=desc" \
  -H "Authorization: Bearer $WHOP_API_KEY" | jq
```

---

### 7. Invoices (`/invoices`)

**List** `GET /invoices`

| Filter | Type | Description |
|--------|------|-------------|
| `company_id` | string | Company scope |
| `product_ids` | string[] | Filter by product |
| `statuses` | string[] | Invoice status filter |
| `collection_methods` | string[] | Collection method filter |

**Sortable columns:** e.g. `created_at`, `due_date`

---

### 8. Checkout Configurations (`/checkout_configurations`)

**List** `GET /checkout_configurations` — **requires** `company_id`

| Filter | Type | Description |
|--------|------|-------------|
| `company_id` | string | **Required** |
| `plan_id` | string | Filter by plan |

---

### 9. Promo Codes (`/promo_codes`)

**List** `GET /promo_codes` — **requires** `company_id`

| Filter | Type | Description |
|--------|------|-------------|
| `company_id` | string | **Required** |
| `product_ids` | string[] | Filter by product |
| `plan_ids` | string[] | Filter by plan |
| `status` | string | Promo code status |

---

### 10. Affiliates (`/affiliates`)

**List** `GET /affiliates` — **requires** `company_id`

| Filter | Type | Description |
|--------|------|-------------|
| `company_id` | string | **Required** |
| `query` | string | Search by username |
| `status` | string | `active` or `archived` |

**Sortable columns:** various

---

### 11. Leads (`/leads`)

**List** `GET /leads` — **requires** `company_id`

| Filter | Type | Description |
|--------|------|-------------|
| `company_id` | string | **Required** |
| `product_ids` | string[] | Filter by product |

---

### 12. Disputes (`/disputes`)

**List** `GET /disputes` — **requires** `company_id`

---

### 13. Reviews (`/reviews`)

**List** `GET /reviews` — **requires** `product_id`

| Filter | Type | Description |
|--------|------|-------------|
| `product_id` | string | **Required** |
| `min_stars` / `max_stars` | integer | 1–5 star range |

---

### 14. Refunds (`/refunds`)

**List** `GET /refunds`

| Filter | Type | Description |
|--------|------|-------------|
| `company_id` / `payment_id` / `user_id` | string | Various scope filters |

---

### 15. DM Channels (`/dm_channels`)

**List** `GET /dm_channels`
**Retrieve** `GET /dm_channels/{id}`

| Filter | Type | Description |
|--------|------|-------------|
| `company_id` | string | Filter by company |

---

### 16. Chat Channels (`/chat_channels`)

**List** `GET /chat_channels`
**Retrieve** `GET /chat_channels/{id}`

---

### 17. Courses (`/courses`)

**List** `GET /courses`

---

### 18. Course Students (`/course_students`)

**List** `GET /course_students`

---

### 19. Stats (`/stats/describe`, `/stats/metric`, `/stats/raw`)

Three stats endpoints for analytics data.

---

### 20. Webhooks (`/webhooks`)

**List** `GET /webhooks`
**Retrieve** `GET /webhooks/{id}`

---

### 21. Card Transactions (`/card_transactions`)

**List** `GET /card_transactions`

---

### 22. Bounties (`/bounties`)

**List** `GET /bounties`

---

### 23. Authorized Users (`/authorized_users`)

**List** `GET /authorized_users`

---

### 24. Forum Posts (`/forum_posts`)

**List** `GET /forum_posts`

---

## NL → API Mapping Guide

When the user asks a question, use this mapping:

| User says... | API call |
|---|---|
| "5 most recent members" | `GET /members?first=5&order=created_at&direction=desc` |
| "Find member by email/username" | `GET /members?query=<search>` |
| "Active/customer members" | `GET /members?statuses=joined&access_level=customer` |
| "Members who churned/canceled" | `GET /members?most_recent_actions=churned` |
| "Member info on {id}" | `GET /members/{id}` |
| "Active memberships" | `GET /memberships?statuses=active` |
| "Users with multiple active memberships" | Fetch all `memberships?statuses=active&first=500`, then `jq '[.data[] | .user_id] | group_by(.) | map(select(length > 1))'` to find duplicates |
| "Members in trial" | `GET /members?most_recent_actions=trialing` |
| "Recent payments" | `GET /payments?first=20&order=created_at&direction=desc` |
| "My products" | `GET /products?first=50&company_id=<id>` |
| "Promo codes I have" | `GET /promo_codes?first=50&company_id=<id>` |
| "Affiliates / referrals" | `GET /affiliates?first=50&company_id=<id>` |
| "Disputes / chargebacks" | `GET /disputes?first=50&company_id=<id>` |
| "Reviews for a product" | `GET /reviews?product_id=<id>` |
| "Leads / waitlist" | `GET /leads?first=50&company_id=<id>` |
| "Invoices" | `GET /invoices?first=50&order=created_at&direction=desc` |
| "Stats / analytics" | `GET /stats/describe`, `GET /stats/metric`, `GET /stats/raw` |
| "Refunds" | `GET /refunds?first=50&direction=desc` |
| "Courses" | `GET /courses` |
| "DM / chat channels" | `GET /dm_channels` / `GET /chat_channels` |

## Helpful jq Snippets

```bash
# Pretty-print member data in table form
jq '.data[] | {id, username: .user.username, email: .user.email, access_level, status, most_recent_action, usd_total_spent, joined_at}'

# Extract all user IDs from a flat list
jq '[.data[].user.id] | unique

# Group memberships by user, find ones with 2+ entries
jq '[.data[] | {user_id: .user_id, id, status}] | group_by(.user_id) | map(select(length > 1))'

# Count by status
jq '[.data[].status] | group_by(.) | map({status: .[0], count: length})'

# Total revenue from payment list
jq '[.data[].amount] | add'
```

## Pitfalls

- **company_id is required** for memberships, products, experiences, promo codes, affiliates, leads, and disputes — you need to know your company ID first
- **Getting company ID**: The API key's company ID is NOT in the `/companies` list (needs separate permission). Get it from a member's `company.id` field via `GET /members/{member_id}`. Known: `biz_nCs0389EcD92Se` ("Harmless Creations", route "hrc")
- **Membership response shape**: `user` is a nested object (`{id, username, email}`), not a flat `user_id`. Same for `product` (`{id, title}`) and `plan` (`{id}`).
- **Cursor pagination** — use `page_info.end_cursor` as `after` param for next page. Default `first` is 50.
- **Defaults to descending** sort direction
- **`order` param varies per endpoint** — check sortable columns above
- **API key** lives in `~/.hermes/.env` as `WHOP_API_KEY` — `source` before curl
- **100 memberships max per page** — the default pagination cap is 50, and `first=500` returns a max of 100. Use cursor pagination for full data.
