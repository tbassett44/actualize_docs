---
title: Subscription Management System Documentation
deprecated: false
hidden: false
metadata:
  robots: index
---
## Overview

The subscription system manages user memberships through Stripe, handling billing cycles, trials, cancellations, and usage tracking. There are two primary subsystems:

1. **Modern System** (`account.php` + `stripe_subscription` collection) - Used by the new account portal
2. **Legacy System** (`stripe.php` + `subscription` + `current_subscription_info` collections) - Used by the original app

***

## Database Collections (Source of Truth)

### Primary Collections

| Collection                  | Purpose                                                         | Key Fields                                                                                                                    |
| --------------------------- | --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `stripe_subscription`       | **Modern system** - Stores Stripe subscription objects          | `id`, `user_id`, `status`, `plan`, `cancel_at_period_end`, `cancel_at`, `canceled_at`, `trial_end`, `current_period_end`      |
| `subscription`              | **Legacy system** - Stores subscription with nested Stripe data | `page.id` (user ID), `status`, `stripe` (full Stripe object), `ts`                                                            |
| `current_subscription_info` | **Legacy** - Tracks validity period and membership details      | `page.id`, `valid_until`, `membership`, `canceled`, `overdue`, `card`, `donation`                                             |
| `subscription_plan`         | Plan configurations                                             | `id`, `name`, `description`, `monthly_price_id`, `yearly_price_id`, `monthly_price`, `yearly_price`, `ai_limit`, `product_id` |
| `stripe`                    | User's Stripe customer mapping                                  | `id` (user_id), `stripe_id` (Stripe customer ID)                                                                              |

### Supporting Collections

| Collection                  | Purpose                                                          |
| --------------------------- | ---------------------------------------------------------------- |
| `trial_code`                | Trial code definitions (code, amount, units, limit, valid_until) |
| `trial_code_used`           | Tracks trial code usage (code, used_by, status: pending/claimed) |
| `stripe_webhook`            | Stores all webhook events from Stripe                            |
| `payment_info`              | Payment records for successful charges                           |
| `paid_subscription`         | Invoice records from successful subscription payments            |
| `subscription_info_history` | Historical membership changes                                    |

***

## Subscription States

### Stripe Subscription Status Values

| Status       | Description                      | UI Badge Color  |
| ------------ | -------------------------------- | --------------- |
| `active`     | Normal active subscription       | Green (#22c55e) |
| `trialing`   | In trial period (no charges yet) | Blue (#60a5fa)  |
| `canceled`   | Fully canceled by Stripe         | Red (#ef4444)   |
| `past_due`   | Payment failed, Stripe retrying  | Amber           |
| `unpaid`     | All retry attempts failed        | Red             |
| `incomplete` | Initial payment failed           | Gray            |

### Cancel-at-Period-End State

When `cancel_at_period_end = true`:

* Status remains `active` or `trialing`
* User retains full access until `current_period_end`
* UI shows "Pending Cancellation" badge (Amber #fbbf24)
* `cancel_at` contains the timestamp when access ends

***

## API Endpoints

### Modern System (account.php)

| Endpoint                  | Method | Purpose                                                                                  |
| ------------------------- | ------ | ---------------------------------------------------------------------------------------- |
| `/account/display`        | GET    | Returns subscription, plan, and usage data for current user                              |
| `/account/subscribe`      | POST   | Creates or updates subscription (params: `price_id`, `method_id`, optional `trial_code`) |
| `/account/stop`           | POST   | Sets `cancel_at_period_end = true` (graceful cancellation)                               |
| `/account/membership`     | GET    | Lists available plans and current subscription                                           |
| `/account/editmembership` | POST   | Preview plan change with proration calculation                                           |
| `/account/usage`          | GET    | Returns AI usage (used, limit, reset_date)                                               |
| `/account/link`           | GET    | Redirects to account portal with temp token                                              |

### Legacy System (stripe.php)

| Endpoint                          | Method | Purpose                                               |
| --------------------------------- | ------ | ----------------------------------------------------- |
| `/stripe/core/load`               | GET    | Loads cards, membership ranges, current subscription  |
| `/stripe/core/updatesubscription` | POST   | Creates/updates subscription with plans and donations |
| `/stripe/core/stop`               | POST   | Immediately cancels subscription                      |

***

## Subscription Lifecycle

### 1. Subscription Creation

**Flow** (`account::subscribe()`):

```
1. Validate user has Stripe customer account
2. Check for existing subscription
3. If trial_code provided:
   a. Validate code exists and is not expired
   b. Check usage limit not exceeded
   c. Check user hasn't already used code
   d. Create trial_code_used record (status: pending)
   e. Calculate trial_end timestamp
4. Create Stripe subscription with trial_end if applicable
5. Save subscription to stripe_subscription collection
6. Mark trial_code_used as claimed (if applicable)
```

### 2. Billing Cycle Changes

**Monthly → Yearly**:

```php
$updateParams['proration_behavior'] = 'always_invoice';
$updateParams['billing_cycle_anchor'] = 'now';
// Charges prorated amount immediately, resets billing cycle
```

**Yearly → Monthly**:

```php
$updateParams['proration_behavior'] = 'none';
$updateParams['billing_cycle_anchor'] = 'unchanged';
// No proration, change takes effect at period end
```

### 3. Subscription Cancellation

**Modern (account::stop())** - Graceful cancellation:

```php
$updated = \Stripe\Subscription::update($current['id'], [
    'cancel_at_period_end' => true
]);
// Updates local DB immediately
db2::update(DB,'stripe_subscription',['id'=>$current['id']],['$set'=>[
    'cancel_at_period_end'=>true,
    'cancel_at'=>$updated->cancel_at,
    'canceled_at'=>$updated->canceled_at
]]);
```

**Legacy (stripe::stop())** - Immediate cancellation:

```php
$sub->cancel(); // Immediately cancels in Stripe
core::update('current_subscription_info',[...],['canceled'=>time()]);
core::update('subscription',['id'=>$current['id']],['status'=>'canceled']);
```

### 4. Reactivation

When updating a subscription that has `cancel_at_period_end = true`:

```php
if($current['status']!='cancelled'){
    if(isset($current['cancel_at'])&&$current['cancel_at']>time()){
        $updateParams['cancel_at_period_end']=false;
        $updateParams['cancel_at']=null;
    }
}
```

***

## Webhooks

All webhooks are processed in `stripe::webhook()` and stored in `stripe_webhook` collection.

### Webhook Handlers

| Event                           | Handler         | Actions                                                                           |
| ------------------------------- | --------------- | --------------------------------------------------------------------------------- |
| `invoice.payment_succeeded`     | Lines 2423-2509 | Creates `payment_info` record, updates `paid_subscription`, extends `valid_until` |
| `invoice.payment_failed`        | Lines 2510-2519 | Currently logs only (TODO: notify user)                                           |
| `customer.subscription.deleted` | Lines 2520-2526 | Calls `stripe::stop()` if not already canceled locally                            |
| `customer.subscription.updated` | Line 2527       | No-op (handled synchronously)                                                     |
| `customer.subscription.created` | Line 2528       | No-op (handled synchronously)                                                     |
| `invoice.upcoming`              | Line 2421       | No-op (could send reminder email)                                                 |
| `charge.refunded`               | Lines 2418-2419 | Logs refund                                                                       |

### invoice.payment_succeeded Detail

```php
// 1. Skip if $0 invoice (trial start)
if($data['data']['object']['total']==0){
    phi::log('✅ Start of a metered subscription!');
    break;
}

// 2. Calculate payment split
$split=self::calcSplit($total, $membership, $donation);

// 3. Create payment_info record with fees
core::save('payment_info', $payment_info);

// 4. Save invoice to paid_subscription
db2::update(DB,'paid_subscription',[...]);

// 5. Update valid_until (current_period_end + 1 day buffer)
$csub=self::getCurrentSubscription($data['user'],1);
$valid_until=$csub['current_period_end']+(60*60*24*1);
core::update('current_subscription_info',['page.id'=>$data['user']],[
    'valid_until'=>$valid_until
],[...],['canceled','overdue']); // Unsets canceled/overdue flags
```

***

## Trial Code System

### Trial Code Fields

| Field         | Type      | Description                   |
| ------------- | --------- | ----------------------------- |
| `code`        | string    | User-entered code             |
| `amount`      | number    | Duration amount               |
| `units`       | string    | `months` or `years`           |
| `limit`       | number    | Maximum uses (optional)       |
| `valid_until` | timestamp | Expiration date (optional)    |
| `timezone`    | string    | Timezone for expiration check |
| `price_id`    | string    | Stripe price ID to apply      |
| `claimed`     | number    | Current usage count           |

### Validation Flow (`trial_code_used::ensureAvailability`)

1. Verify code exists
2. Check user hasn't already claimed this code (in prod)
3. Check usage limit not exceeded
4. Check code hasn't expired

### Trial Subscription Creation

```php
$trialDate = new DateTime();
$trialDate->modify("+{$trial_code['amount']} months");
$trial_ends = $trialDate->getTimestamp();
$createOpts['trial_end'] = $trial_ends;
$subscription = \Stripe\Subscription::create($createOpts);
// Status will be 'trialing' until trial_ends
```

***

## Usage Tracking

### Usage Fields (`account::usage()`)

| Field        | Description                                                                       |
| ------------ | --------------------------------------------------------------------------------- |
| `used`       | AI interactions this billing period                                               |
| `limit`      | Monthly limit based on plan (default: 50, Pro: from `subscription_plan.ai_limit`) |
| `reset_date` | First day of next month (formatted)                                               |

### Usage Calculation

```php
$start = strtotime('first day of this month');
$end = strtotime('23:59:59 on the last day of this month');
$reset = date('F j, Y', strtotime('first day of next month'));

// Get limit from active subscription plan
$plan = db2::findOne(DB,'subscription_plan',['$or'=>[
    ['monthly_price_id'=>$price],
    ['yearly_price_id'=>$price]
]]);
$limit = $plan['ai_limit']; // e.g., 200 for Pro
```

***

## Key Functions Reference

### account.php

| Function                              | Purpose                                                         |
| ------------------------------------- | --------------------------------------------------------------- |
| `getCurrentSubscription($uid, $plan)` | Returns active/trialing subscription from `stripe_subscription` |
| `display($r)`                         | Returns subscription + plan + usage                             |
| `subscribe($r)`                       | Create or update subscription                                   |
| `stop($r)`                            | Set cancel_at_period_end                                        |
| `usage($r)`                           | Calculate current month usage                                   |

### stripe.php

| Function                                    | Purpose                                                                 |
| ------------------------------------------- | ----------------------------------------------------------------------- |
| `getCurrentSubscription($uid, $force)`      | Legacy - from `subscription` collection, optionally refresh from Stripe |
| `getCurrentStatus($current)`                | Returns 'not_active', 'active', or 'overdue'                            |
| `stop($r, $uid, $from_stripe)`              | Legacy - immediate cancellation                                         |
| `webhook($r, $data)`                        | Process Stripe webhooks                                                 |
| `updateUsage($opts)`                        | Update metered billing usage                                            |
| `calcSplit($total, $membership, $donation)` | Calculate payment splits                                                |

***

## Edge Cases

### 1. Subscription Already Pending Cancellation

When user updates subscription while `cancel_at_period_end = true`:

* System clears `cancel_at_period_end` and `cancel_at`
* Subscription continues with new plan

### 2. Trial to Paid Transition

When updating subscription during trial:

```php
if($current['status']=='trialing'){
    $updateParams['trial_end']='now';  // End trial immediately
    $updateParams['proration_behavior'] = 'always_invoice';
}
```

### 3. Stripe Deletes Subscription Externally

Webhook `customer.subscription.deleted` triggers:

```php
if($current && !isset($current['canceled'])){
    self::stop(false, $data['user'], 1); // from_stripe=true
}
```

### 4. Double Source of Truth

The legacy `current_subscription_info.valid_until` and modern `stripe_subscription.current_period_end` may differ by 1 day (buffer added in legacy system).

***

## Configuration

### Stripe Keys

Retrieved via `stripe::getStripeKeys()` - returns different keys for dev vs prod environments.

### Plan Configuration

Stored in `subscription_plan` collection:

```json
{
    "id": "SP-52KHLD8N7RT3",
    "name": "Flow",
    "description": "Ongoing reflection",
    "monthly_price": 2200,
    "monthly_price_id": "price_xxx",
    "yearly_price": 20000,
    "yearly_price_id": "price_yyy",
    "ai_limit": 200,
    "product_id": "prod_xxx"
}
```

***

## Known Issues / TODOs

1. **invoice.payment_failed** - Currently only logs, should notify user
2. **Double source of truth** - `stripe_subscription` vs `current_subscription_info` collections
3. **Legacy stop() vs modern stop()** - Different cancellation behaviors (immediate vs graceful)
4. **Webhook idempotency** - Webhooks stored but response not checked for duplicates
