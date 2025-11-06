# GLOB PAY Orchestrate S2S Payment API Documentation

## Overview

The Globpay Orchestrate S2S (Server-to-Server) Payment API allows you to initiate card payments directly from your server.

## Base URL

```
https://api.globpay.ai/api/v1
```

## Authentication

All requests require a Bearer token in the Authorization header:

```
Authorization: Bearer <your-jwt-token>
```

---

## Payment Initiation

### Endpoint

```
POST /orchestrate/payment
```

### Request Headers

```
Content-Type: application/json
Authorization: Bearer <your-jwt-token>
```

### Request Payload

```json
{
  "impalaMerchantId": "your_merchant_id",
  "currency": "USD",
  "amount": 10,
  "externalId": "ORDER_1234599",
  "callbackUrl": "https://webhook.site/a2754b5e-4c69-47fa-85a3-9f0f4c0d5cd8",
  "redirectUrl": "https://cradlevoices.com",
  "ip_address": "192.168.1.100",
  "customer_first_name": "John",
  "customer_last_name": "Doe",
  "customer_phone_code": "0044",
  "customer_phone": "0794940160",
  "customer_email": "colls@gmail.com",
  "billing_first_name": "colls",
  "billing_last_name": "codes",
  "billing_street": "123 Main street",
  "billing_street2": "Apartment 4B",
  "billing_city": "New York",
  "billing_country": "US",
  "billing_post_code": "1001",
  "billing_state": "NY",
  "card_holder": "John Doe",
  "card_number": "536774000***69",
  "card_exp_month": "08",
  "card_exp_year": "2026",
  "card_security_code": "063",
  "description": "Test payment for order ORDER_12345",
  "account_id": "445376e5-2373-42f7-856a-4955b0ea3b34"
}
```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `impalaMerchantId` | string | Yes | Your merchant identifier |
| `currency` | string | Yes | Currency code (e.g., USD, EUR, GBP) |
| `amount` | number | Yes | Payment amount |
| `externalId` | string | Yes | Your unique order/transaction identifier |
| `callbackUrl` | string | Yes | URL to receive payment status callbacks |
| `redirectUrl` | string | No | URL to redirect customer after payment (defaults to https://cradlevoices.com) |
| `ip_address` | string | Yes | Customer's IP address |
| `customer_first_name` | string | Yes | Customer's first name |
| `customer_last_name` | string | Yes | Customer's last name |
| `customer_phone_code` | string | Yes | Phone country code |
| `customer_phone` | string | Yes | Customer's phone number |
| `customer_email` | string | Yes | Customer's email address |
| `billing_first_name` | string | Yes | Billing address first name |
| `billing_last_name` | string | Yes | Billing address last name |
| `billing_street` | string | Yes | Billing address line 1 |
| `billing_street2` | string | No | Billing address line 2 (optional) |
| `billing_city` | string | Yes | Billing city |
| `billing_country` | string | Yes | Billing country code (e.g., US, GB) |
| `billing_post_code` | string | Yes | Billing postal code |
| `billing_state` | string | Yes | Billing state/province |
| `card_holder` | string | Yes | Name on card |
| `card_number` | string | Yes | Card number |
| `card_exp_month` | string | Yes | Card expiry month (01-12) |
| `card_exp_year` | string | Yes | Card expiry year (YYYY or YY) |
| `card_security_code` | string | Yes | Card CVV/CVC |
| `description` | string | No | Payment description |
| `account_id` | string | Yes | Orchestrate account ID |

### Success Response (200 OK)

```json
{
  "status": "success",
  "message": "Payment initiated successfully",
  "payment_id": "c2aa34d5-803b-4c35-adf0-46f39ddb18f0",
  "secure_id": "OyFI1ISeGvw5Wt5i-6RB-A==",
  "external_id": "ORDER_1234599",
  "amount": 10,
  "currency": "USD",
  "payment_status": "pending",
  "payment_link": "https://orchestrate.global/checkout/c2aa34d5-803b-4c35-adf0-46f39ddb18f0",
  "url": "https://orchestrate.global/payments/c2aa34d5-803b-4c35-adf0-46f39ddb18f0/threeds"
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | Response status ("success" or "error") |
| `message` | string | Response message |
| `payment_id` | string | Orchestrate payment ID |
| `secure_id` | string | Unique secure identifier for tracking |
| `external_id` | string | Your original external ID |
| `amount` | number | Payment amount |
| `currency` | string | Payment currency |
| `payment_status` | string | Payment status ("pending", "completed", "failed") |
| `payment_link` | string | Link to Orchestrate checkout page |
| `url` | string | **Use this URL for the OTP/3DS page** - Direct link to complete payment authentication |

### Error Response (4xx/5xx)

```json
{
  "error": "Payment initiation failed",
  "details": "Error message details"
}
```

---

## Payment Callbacks

### Callback Endpoint

Your `callbackUrl` will receive POST requests when payment status changes.

### Callback for COMPLETED Status

```json
{
  "amount": 10,
  "currency": "USD",
  "externalId": "ORDER_1234599",
  "netAmount": 10,
  "secureId": "OyFI1ISeGvw5Wt5i-6RB-A==",
  "transactionReport": "COMPLETED",
  "transactionStatus": "COMPLETE"
}
```

### Callback for FAILED Status

```json
{
  "amount": 10,
  "currency": "USD",
  "externalId": "ORDER_1234599",
  "netAmount": 10,
  "secureId": "OyFI1ISeGvw5Wt5i-6RB-A==",
  "transactionReport": "FAILED",
  "transactionStatus": "FAILED"
}
```

### Callback Fields

| Field | Type | Description |
|-------|------|-------------|
| `amount` | number | Payment amount |
| `currency` | string | Payment currency |
| `externalId` | string | Your original external ID |
| `netAmount` | number | Net amount (same as amount) |
| `secureId` | string | Secure identifier for tracking |
| `transactionReport` | string | Transaction report ("COMPLETED" or "FAILED") |
| `transactionStatus` | string | Transaction status ("COMPLETE" or "FAILED") |

**Note:** Callbacks are only sent for `COMPLETED` and `FAILED` statuses. `PENDING` status updates do not trigger callbacks.

---

## Payment Flow

1. **Initiate Payment**: Send POST request to `/orchestrate/payment` with payment details
2. **Receive Response**: Get `payment_id`, `secure_id`, and `url` in response
3. **Redirect Customer**: Use the `url` field to redirect customer to Orchestrate's OTP/3DS page
4. **Customer Authentication**: Customer completes 3DS authentication on Orchestrate's page
5. **Receive Callback**: Your `callbackUrl` receives status update (COMPLETED or FAILED)

---

## Example cURL Request

```bash
curl --location 'https://api.globpay.ai/api/v1/orchestrate/payment' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer YOUR_JWT_TOKEN' \
--data-raw '{
  "impalaMerchantId": "your_merchant_id",
  "currency": "USD",
  "amount": 10,
  "externalId": "ORDER_1234599",
  "callbackUrl": "https://webhook.site/a2754b5e-4c69-47fa-85a3-9f0f4c0d5cd8",
  "redirectUrl": "https://cradlevoices.com",
  "ip_address": "192.168.1.100",
  "customer_first_name": "John",
  "customer_last_name": "Doe",
  "customer_phone_code": "0044",
  "customer_phone": "0794940160",
  "customer_email": "colls@gmail.com",
  "billing_first_name": "colls",
  "billing_last_name": "codes",
  "billing_street": "123 Main street",
  "billing_city": "New York",
  "billing_country": "US",
  "billing_post_code": "1001",
  "billing_state": "NY",
  "card_holder": "John Doe",
  "card_number": "53677400******29169",
  "card_exp_month": "08",
  "card_exp_year": "2026",
  "card_security_code": "063",
  "description": "Test payment for order ORDER_12345",
  "account_id": "445376e5-2373-42f7-856a-4955b0ea3b34"
}'
```

---

## Important Notes

1. **OTP/3DS URL**: Use the `url` field from the success response to redirect customers to the OTP/3DS authentication page
2. **Card Expiry Year**: Can be provided as 2-digit (YY) or 4-digit (YYYY) format
3. **Card Expiry Month**: Should be 2 digits with leading zero (e.g., "01", "08")
4. **Billing Address**: `billing_street2` is optional. If not provided, `billing_street` will be duplicated as `line2`
5. **Callbacks**: Only sent for COMPLETED and FAILED statuses. PENDING status updates do not trigger callbacks
6. **Idempotency**: Use unique `externalId` for each payment request
7. **Timeout**: Payment requests expire after 3600 seconds (1 hour)

---

## Status Codes

| Code | Description |
|------|-------------|
| 200 | Payment initiated successfully |
| 400 | Bad request (invalid payload) |
| 401 | Unauthorized (invalid or missing token) |
| 404 | Merchant not found |
| 500 | Internal server error |

---

## Support

For issues or questions, please contact support or refer to the main API documentation.

