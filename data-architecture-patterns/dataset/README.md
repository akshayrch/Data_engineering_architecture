# Dataset: E-Commerce Clickstream

All four architecture implementations in this repository use this dataset. The domain is a mid-scale e-commerce platform — think of it as a company with millions of monthly active users, hundreds of thousands of SKUs, and peak traffic during sales events.

Using the same dataset across all four architectures is the point. The tradeoffs aren't hypothetical — they show up as actual code differences, latency differences, and operational differences against the same inputs and requirements.

---

## Event Schema

All events share a common envelope:

```json
{
  "event_id": "uuid-v4",
  "event_type": "page_view | add_to_cart | checkout_start | checkout_complete | purchase | product_impression | search",
  "user_id": "string | null (anonymous)",
  "session_id": "uuid-v4",
  "device_type": "web | mobile_ios | mobile_android | tablet",
  "event_timestamp": "ISO-8601 UTC",
  "ingestion_timestamp": "ISO-8601 UTC (set at Kafka produce time)",
  "payload": { /* event-specific fields below */ }
}
```

### `page_view`
```json
{
  "page_type": "home | category | product | cart | search_results | checkout",
  "page_url": "string",
  "product_id": "string | null",
  "category_id": "string | null",
  "referrer": "organic | paid_search | email | social | direct | affiliate",
  "time_on_previous_page_ms": "integer | null"
}
```

### `product_impression`
```json
{
  "product_id": "string",
  "impression_source": "search | recommendation | category_browse | homepage_featured",
  "position": "integer (rank in list)",
  "search_query": "string | null",
  "recommendation_model_id": "string | null"
}
```

### `add_to_cart`
```json
{
  "product_id": "string",
  "sku_id": "string",
  "quantity": "integer",
  "unit_price": "decimal",
  "currency": "USD | EUR | GBP",
  "cart_id": "uuid-v4"
}
```

### `checkout_start`
```json
{
  "cart_id": "uuid-v4",
  "cart_item_count": "integer",
  "cart_value": "decimal",
  "currency": "string",
  "has_promo_code": "boolean"
}
```

### `checkout_complete` / `purchase`
```json
{
  "order_id": "uuid-v4",
  "cart_id": "uuid-v4",
  "items": [
    {
      "product_id": "string",
      "sku_id": "string",
      "quantity": "integer",
      "unit_price": "decimal",
      "discount_amount": "decimal"
    }
  ],
  "subtotal": "decimal",
  "discount_total": "decimal",
  "tax": "decimal",
  "order_total": "decimal",
  "currency": "string",
  "payment_method": "credit_card | paypal | apple_pay | google_pay | buy_now_pay_later",
  "shipping_method": "standard | express | overnight",
  "promo_code": "string | null"
}
```

### `search`
```json
{
  "query": "string",
  "result_count": "integer",
  "filters_applied": ["string"],
  "clicked_product_id": "string | null",
  "no_results": "boolean"
}
```

---

## Reference Data (Dimensions)

### Product Catalog
```json
{
  "product_id": "string",
  "sku_id": "string",
  "product_name": "string",
  "brand": "string",
  "category_l1": "Electronics | Apparel | Home | Sports | Beauty",
  "category_l2": "string",
  "category_l3": "string",
  "base_price": "decimal",
  "is_active": "boolean",
  "created_at": "ISO-8601",
  "updated_at": "ISO-8601"
}
```

### User Profiles
```json
{
  "user_id": "string",
  "user_segment": "new | returning | loyal | lapsed | high_value",
  "acquisition_channel": "organic | paid | referral | email",
  "account_created_at": "ISO-8601",
  "country": "string",
  "is_email_subscribed": "boolean"
}
```

---

## Data Characteristics (by design)

These characteristics were chosen specifically to stress-test each architecture differently:

| Characteristic | Detail | Why it matters |
|---|---|---|
| Volume | ~10,000 events/sec peak, ~500/sec average | Justifies streaming architectures |
| Late data | Mobile events up to 6 hours late | Tests Lambda's speed layer, Kappa's watermarking |
| Duplicates | ~0.3% duplicate rate (at-least-once Kafka delivery) | Tests ACID / MERGE INTO in Lakehouse |
| Schema drift | New event fields added quarterly | Tests schema evolution in Iceberg/Delta |
| Slowly changing dimensions | Product prices and categories change weekly | Tests SCD handling in Silver layer |
| Seasonal spikes | 10x volume during flash sales | Tests autoscaling and backpressure |
| Anonymous users | ~25% of sessions have null user_id | Tests null handling and session attribution |
| Multi-currency | Orders in USD, EUR, GBP | Tests currency normalization in Silver |

---

## Analytical Requirements

These are the business questions the dataset must answer — and what makes each architecture's implementation interesting:

**Real-time (< 30 seconds):**
- Revenue in last 15 minutes by product
- Active sessions right now
- Cart abandonment alerts (add_to_cart with no purchase in 30 min)
- Fraud signals (> 3 purchases in 5 minutes from same user)

**Near-real-time (< 5 minutes):**
- Conversion funnel for last hour (impression → view → cart → purchase)
- Search queries with zero results (product catalog gap detection)

**Batch (daily):**
- Daily revenue by product / category / region
- 30-day / 90-day user cohort retention
- Product performance: impression → purchase rate
- Promo code effectiveness
- LTV estimation by acquisition channel

**Historical / ad-hoc:**
- Point-in-time revenue reconstruction for compliance
- Cohort analysis going back 2+ years
- ML training datasets (reproducible, pinned to specific date)

---

## Sample Data Generator

A synthetic data generator is coming in `dataset/generator/`. It will produce realistic clickstream events with the characteristics above for local testing of all four architecture implementations without requiring production data access.
