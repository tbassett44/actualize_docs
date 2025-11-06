---
title: 📲 Tap to Pay
excerpt: ''
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
# Tap to Pay Plugin Documentation

The Tap to Pay plugin integrates Stripe's Tap to Pay functionality into the mobile application, enabling contactless payments using the device's NFC capabilities.

## Plugin Information

- **Plugin ID**: `cordova-plugin-stripe-tap-to-pay`
- **Version**: `2.2.2`
- **Status**: Enabled
- **Platform Support**: iOS (≥16.4), Android

## Overview

Tap to Pay allows merchants to accept contactless payments directly on their mobile devices without requiring additional hardware. This plugin wraps Stripe's Terminal SDK to provide seamless payment processing capabilities.

## Prerequisites

### Stripe Account Setup

1. **Stripe Account**: Active Stripe account with Terminal enabled
2. **Express Account**: Connected Express account for payment processing
3. **Location Registration**: Physical location registered with Stripe Terminal

### Device Requirements

- **iOS**: Version 16.4 or higher
- **Android**: NFC-enabled device
- **Permissions**: NFC access permissions

## API Reference

### Frontend JavaScript API

#### Device Capability Check

```javascript
// Check if Tap to Pay is supported on current device
if (phone.tapToPay.canUse()) {
    // Device supports Tap to Pay
    console.log('Tap to Pay is available');
} else {
    // Device doesn't support Tap to Pay
    console.log('Tap to Pay not supported');
}
```

**Behavior:**

- **iOS**: Returns `true` if device version ≥ 16.4
- **Android**: Returns `true` (assumes NFC capability)
- **Web**: Shows toast message and returns `false`

### Backend API Endpoints

#### Get Tap to Pay Options

```javascript
// Fetch user's tap to pay configuration
api.call('user/taptopay', {}, function(response) {
    if (response.success) {
        const data = response.data;
        console.log('Location info:', data.location_info);
        console.log('Stripe info:', data.stripe_info);
        console.log('Express account:', data.stripe_express);
    }
});
```

**Response Data:**

- `location_info`: Stripe Terminal location details
- `stripe_info`: User's Stripe account information
- `stripe_express`: Express account details (if available)

#### Create Tap to Pay User

```javascript
// Admin-only: Create new tap to pay user
api.call('user/tap_to_pay_create', {
    current: {
        email: 'user@example.com',
        name: 'User Name'
        // Additional user data
    }
}, function(response) {
    if (response.success) {
        console.log('User created:', response.data);
    }
});
```

**Requirements:**

- Admin role required (`RHJGBVT45LMY`)
- Valid email address
- User data in `current` object

## Stripe Integration

### Location Management

The plugin integrates with Stripe Terminal's location system:

```php
// Create new location
stripe::createLocation($request, [
    'display_name' => 'Store Location',
    'address' => [
        'line1' => '123 Main St',
        'city' => 'San Francisco',
        'state' => 'CA',
        'postal_code' => '94111',
        'country' => 'US'
    ]
]);

// Update existing location
stripe::updateLocation($request, $locationId, $updateData);
```

### Connection Tokens

For Terminal SDK authentication:

```php
// Generate connection token
$connectionToken = stripe::connectionToken();
// Returns: ['secret' => 'pst_test_...']
```

## Implementation Flow

### 1. Device Capability Check

```javascript
if (!phone.tapToPay.canUse()) {
    // Show alternative payment methods
    return;
}
```

### 2. Initialize Stripe Terminal

```javascript
// This would typically be handled by the Cordova plugin
// The plugin manages Terminal SDK initialization
```

### 3. Location Setup

```javascript
// Fetch user's location configuration
api.call('user/taptopay', {}, function(response) {
    if (response.success && response.data.location_info) {
        // Location is configured
        initializePaymentFlow();
    } else {
        // Redirect to location setup
        showLocationSetup();
    }
});
```

### 4. Payment Processing

The actual payment processing is handled by the Stripe Terminal SDK through the Cordova plugin interface.

## Configuration

### Stripe Keys

Configure Stripe keys in your environment configuration:

```json
{
    "stripe": {
        "publishable_key": "pk_test_...",
        "secret_key": "sk_test_...",
        "env": "test"
    }
}
```

### Database Collections

#### `stripe_location`

Stores Terminal location information:

```javascript
{
    "page": {"id": "user_id"},
    "stripe_location_id": "tml_...",
    "display_name": "Store Name",
    "address": {...}
}
```

#### `stripe`

Stores user Stripe account information:

```javascript
{
    "id": "user_id",
    "stripe_id": "cus_...",
    "express": {"id": "acct_..."}
}
```

## Error Handling

### Common Error Scenarios

1. **Device Not Supported**
   ```javascript
   if (!phone.tapToPay.canUse()) {
       modules.toast({
           icon: 'icon-warning-sign',
           content: 'Tap to Pay is only supported on phones'
       });
   }
   ```

2. **Location Not Configured**
   ```javascript
   if (!response.data.location_info) {
       // Redirect to location setup flow
       showLocationConfiguration();
   }
   ```

3. **Express Account Missing**
   ```javascript
   if (!response.data.stripe_info.express) {
       // Redirect to Stripe Express onboarding
       initiateExpressOnboarding();
   }
   ```

## Security Considerations

1. **Admin Restrictions**: User creation requires admin role
2. **Authentication**: All API calls require valid user authentication
3. **Stripe Keys**: Secure storage of Stripe credentials
4. **PCI Compliance**: Payment data handled by Stripe Terminal SDK

## Testing

### Test Mode

- Use Stripe test keys for development
- Test with Stripe's test card numbers
- Verify device compatibility on target devices

### Production Checklist

- [ ] Stripe live keys configured
- [ ] Express account fully onboarded
- [ ] Location registered with Stripe
- [ ] Device permissions granted
- [ ] NFC functionality tested

## Troubleshooting

### iOS Issues

- Ensure iOS version ≥ 16.4
- Check device NFC capability
- Verify app permissions

### Android Issues

- Confirm NFC is enabled
- Check device NFC hardware support
- Verify app permissions

### Stripe Issues

- Validate API keys
- Check Express account status
- Verify location configuration

## Related Documentation

- [Stripe Terminal Documentation](https://docs.stripe.com/terminal)
- [Stripe Tap to Pay Setup](https://docs.stripe.com/terminal/payments/setup-reader/tap-to-pay)
- [Cordova Plugin Development](https://cordova.apache.org/docs/en/latest/guide/hybrid/plugins/)

## Plugin Architecture

### Cordova Plugin Structure

The `cordova-plugin-stripe-tap-to-pay` plugin provides a bridge between JavaScript and native Stripe Terminal SDKs:

```
cordova-plugin-stripe-tap-to-pay/
├── plugin.xml                 # Plugin configuration
├── src/
│   ├── ios/                   # iOS implementation
│   │   ├── StripeTerminal.h
│   │   └── StripeTerminal.m
│   └── android/               # Android implementation
│       └── StripeTerminal.java
└── www/
    └── StripeTerminal.js      # JavaScript interface
```

### Native SDK Integration

#### iOS Integration

- Uses Stripe Terminal iOS SDK
- Requires iOS 16.4+ for Tap to Pay functionality
- Integrates with Core NFC framework

#### Android Integration

- Uses Stripe Terminal Android SDK
- Requires NFC-enabled device
- Integrates with Android NFC APIs

### JavaScript Interface

The plugin exposes native functionality through Cordova's plugin architecture:

```javascript
// Example plugin interface (actual implementation may vary)
window.StripeTerminal = {
    initialize: function(config, success, error) { /* ... */ },
    createPaymentIntent: function(amount, success, error) { /* ... */ },
    collectPayment: function(paymentIntent, success, error) { /* ... */ },
    processPayment: function(paymentIntent, success, error) { /* ... */ }
};
```

## Advanced Usage

### Custom Payment Flow

```javascript
// Example payment processing flow
function processPayment(amount) {
    // 1. Check device capability
    if (!phone.tapToPay.canUse()) {
        showError('Device not supported');
        return;
    }

    // 2. Get location and stripe info
    api.call('user/taptopay', {}, function(response) {
        if (!response.success) {
            showError('Configuration error');
            return;
        }

        // 3. Initialize payment intent on backend
        createPaymentIntent(amount, function(intent) {
            // 4. Collect payment using plugin
            collectPaymentWithTapToPay(intent);
        });
    });
}

function createPaymentIntent(amount, callback) {
    // Backend API call to create Stripe PaymentIntent
    api.call('stripe/create_intent', {
        amount: amount,
        currency: 'usd',
        payment_method_types: ['card_present']
    }, callback);
}
```

### Error Handling Patterns

```javascript
function handlePaymentError(error) {
    switch(error.code) {
        case 'DEVICE_NOT_SUPPORTED':
            showAlternativePaymentMethods();
            break;
        case 'LOCATION_NOT_CONFIGURED':
            redirectToLocationSetup();
            break;
        case 'PAYMENT_DECLINED':
            showDeclinedMessage();
            break;
        default:
            showGenericError(error.message);
    }
}
```

## Stripe Terminal Concepts

### Key Components

1. **Location**: Physical location registered with Stripe
2. **Reader**: The mobile device acting as a payment terminal
3. **PaymentIntent**: Stripe object representing a payment
4. **ConnectionToken**: Authentication token for Terminal SDK

### Payment Flow

1. Create PaymentIntent on backend
2. Initialize Terminal SDK with ConnectionToken
3. Collect payment method (tap/insert/swipe)
4. Process payment through Stripe
5. Confirm payment and update backend

## Fees and Pricing

### Stripe Tap to Pay Fees

- **In-person payments**: 2.7% + 5¢ per transaction
- **No additional hardware costs**
- **Standard Stripe processing fees apply**

Refer to [Stripe's pricing page](https://stripe.com/pricing) for current rates.

## Support

For plugin-specific issues:

1. Check device compatibility with `phone.tapToPay.canUse()`
2. Verify Stripe account configuration
3. Review console logs for error details
4. Test with Stripe's test environment first
5. Check plugin version compatibility
6. Verify Cordova platform versions