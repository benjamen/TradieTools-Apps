# Stripe Integration Setup — TradieTools Quote Builder

Enable $9.99/month Professional tier for unlimited quotes + custom branding.

## Prerequisites

- Stripe account (https://stripe.com)
- Quote Builder deployed to tradietools.optified.nz
- API backend running (localhost:8003 or production)

## Step 1: Create Stripe Products & Prices

1. Go to **Stripe Dashboard** → **Products**
2. Click **+ Add product**
   - **Name:** Professional Quote Builder
   - **Description:** Unlimited quotes + custom branding
   - **Recurring price:** $9.99 NZD / month
3. Copy the **Price ID** (starts with `price_`)
4. Go to **Settings** → **API keys**
5. Copy:
   - **Secret key** (starts with `sk_live_` or `sk_test_`)
   - **Publishable key** (starts with `pk_live_` or `pk_test_`)

## Step 2: Generate Webhook Signing Secret

1. Go to **Developers** → **Webhooks**
2. Click **Add endpoint**
   - **URL:** `https://tradietools.optified.nz/api/stripe/webhook`
   - **Events:** Select:
     - `customer.subscription.created`
     - `customer.subscription.updated`
     - `customer.subscription.deleted`
     - `invoice.payment_succeeded`
     - `invoice.payment_failed`
3. Copy the **Signing secret** (starts with `whsec_`)

## Step 3: Set Environment Variables

Add to `.env` (or your deployment environment):

```bash
# Stripe API Keys
STRIPE_SECRET_KEY=sk_live_YOUR_SECRET_KEY_HERE
STRIPE_PUBLISHABLE_KEY=pk_live_YOUR_PUBLISHABLE_KEY_HERE

# Stripe Product Configuration
STRIPE_PROFESSIONAL_PRICE_ID=price_YOUR_PRICE_ID_HERE

# Webhook Secret
STRIPE_WEBHOOK_SECRET=whsec_YOUR_WEBHOOK_SECRET_HERE
```

**Development (Test Mode):**
Use `sk_test_*` and `pk_test_*` keys for testing.

**Production (Live Mode):**
Use `sk_live_*` and `pk_live_*` keys for real charges.

## Step 4: Update Quote Builder Component

The Quote Builder Vue3 component needs to call the subscription endpoint:

```javascript
// In quote-builder-branded.vue, add:

const upgradeToPaid = async () => {
  // Create payment method via Stripe.js
  const { paymentMethod, error } = await stripe.createPaymentMethod({
    type: 'card',
    card: cardElement
  });

  if (error) {
    alert('Payment error: ' + error.message);
    return;
  }

  // Call backend to create subscription
  const response = await fetch('/api/stripe/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      email: userEmail,
      name: userName,
      payment_method_id: paymentMethod.id
    })
  });

  if (response.ok) {
    const result = await response.json();
    // Store subscription ID in localStorage
    localStorage.setItem('subscription_id', result.subscription_id);
    alert('✅ Subscription successful! You now have unlimited quotes.');
    // Reload to show unlimited tier
    window.location.reload();
  } else {
    alert('Subscription failed. Please try again.');
  }
};
```

## Step 5: Wire Stripe.js to Quote Builder

Add Stripe.js to the HTML template:

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
  const stripe = Stripe('YOUR_PUBLISHABLE_KEY');
  const elements = stripe.elements();
  const cardElement = elements.create('card');
  cardElement.mount('#card-element');
</script>
```

Add a card input div to the upgrade modal:

```html
<div id="card-element" style="border: 1px solid #ddd; padding: 10px; margin: 20px 0; border-radius: 4px;"></div>
```

## Step 6: Test the Integration

### Test Payment Cards (Test Mode Only)

| Card Number | Exp | CVC | Result |
|---|---|---|---|
| 4242 4242 4242 4242 | 12/25 | 123 | ✅ Success |
| 4000 0000 0000 9995 | 12/25 | 123 | ❌ Decline |
| 4000 0025 0000 3155 | 12/25 | 123 | ⚠️ Requires auth |

### Manual Testing Flow

1. Visit https://tradietools.optified.nz/app/quote-builder
2. Create 3 quotes (hit free tier limit)
3. Click "Upgrade Now"
4. Enter test card `4242 4242 4242 4242`
5. Click "Subscribe"
6. Verify subscription created in Stripe Dashboard
7. Confirm unlimited quotes now available

### Test Webhook Locally

```bash
# Install Stripe CLI
brew install stripe/stripe-cli/stripe

# Login to Stripe account
stripe login

# Forward webhooks to local server
stripe listen --forward-to localhost:8003/api/stripe/webhook

# Copy the signing secret and add to .env as STRIPE_WEBHOOK_SECRET

# Trigger a test webhook
stripe trigger customer.subscription.created
```

## Step 7: Monitor in Production

### Stripe Dashboard Checks

- **Customers** → Monitor subscription count and active users
- **Events** → Check webhook delivery success rate
- **Revenue** → Track recurring revenue (MRR)
- **Disputes** → Monitor chargebacks and disputes

### Backend Logging

Add logging to `api/tradie_stripe.py`:

```python
import logging
logger = logging.getLogger(__name__)

@router.post("/subscribe", response_model=SubscriptionResponse)
async def create_subscription(req: CreateSubscriptionRequest):
    logger.info(f"New subscription attempt: {req.email}")
    # ... rest of function ...
    logger.info(f"Subscription created: {subscription.id} for {req.email}")
```

## API Endpoints

### Create Subscription
```bash
POST /api/stripe/subscribe
Content-Type: application/json

{
  "email": "user@example.com",
  "name": "User Name",
  "payment_method_id": "pm_1234567890abcdef"
}

Response:
{
  "subscription_id": "sub_1234567890abcdef",
  "status": "active",
  "current_period_end": "2026-07-07T00:00:00",
  "amount": 9.99,
  "currency": "nzd"
}
```

### Get Subscription Status
```bash
GET /api/stripe/subscription/{user_email}

Response:
{
  "subscription_id": "sub_1234567890abcdef",
  "status": "active",
  "is_active": true,
  "current_period_end": "2026-07-07T00:00:00",
  "quotes_remaining": null  # null = unlimited
}
```

### Check Quote Quota
```bash
POST /api/stripe/check-quota?user_email=user@example.com

Response:
{
  "can_create": true,
  "quotes_used": 2,
  "quotes_remaining": 1,
  "tier": "free"
}
```

### Cancel Subscription
```bash
POST /api/stripe/cancel-subscription/{subscription_id}

Response:
{
  "status": "canceled",
  "message": "Subscription canceled. Your account reverted to free tier (3 quotes/month)."
}
```

### Get Pricing
```bash
GET /api/stripe/pricing

Response:
{
  "tiers": [
    {
      "name": "Free",
      "price": 0,
      "currency": "nzd",
      "period": "monthly",
      "features": ["3 quotes per month", "Basic templates", ...]
    },
    {
      "name": "Professional",
      "price": 9.99,
      "currency": "nzd",
      "period": "monthly",
      "features": ["Unlimited quotes", "Custom branding", ...]
    }
  ]
}
```

## Troubleshooting

### "Invalid API key"
- Verify `STRIPE_SECRET_KEY` is set correctly
- Check key starts with `sk_live_` (production) or `sk_test_` (dev)
- Regenerate key if needed in Stripe Dashboard

### "Price not found"
- Verify `STRIPE_PROFESSIONAL_PRICE_ID` matches Stripe product
- Must be the exact Price ID, not Product ID

### "Webhook signature verification failed"
- Confirm `STRIPE_WEBHOOK_SECRET` is correct (starts with `whsec_`)
- Webhook must be POST request, not GET
- Check timestamp is within 5 minutes of webhook creation

### "Customer already has subscription"
- User may have multiple subscriptions
- Implement idempotency key to prevent duplicates:
  ```python
  subscription = stripe.Subscription.create(
    customer=customer.id,
    items=[{"price": PROFESSIONAL_PRICE_ID}],
    idempotency_key=f"{customer.id}-{datetime.now().date()}"
  )
  ```

## Implementation Roadmap

| Feature | Status | ETA |
|---|---|---|
| Stripe API integration | ✅ Done | June 7 |
| Quote quota enforcement | ⏳ In progress | June 8 |
| Stripe.js card input | ⏳ To do | June 8 |
| Email receipts | ⏳ To do | June 9 |
| Subscription management dashboard | ⏳ To do | June 10 |
| Refunds & dispute handling | ⏳ To do | June 11 |

## Support

- Stripe Documentation: https://stripe.com/docs/api
- API Status: https://status.stripe.com
- Support: https://support.stripe.com

---

**Last Updated:** June 7, 2026
