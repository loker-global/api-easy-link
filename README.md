# Meta API Documentation

## Overview
The Meta API provides endpoints for managing WhatsApp Business API senders through Twilio's messaging services. This API allows developers to validate phone numbers, register senders, verify registrations, and manage sender configurations.

## Authentication
All API requests must include an authorization header with a JWT token containing the Twilio account credentials:

```
Authorization: Bearer <base64-encoded-accountSid:authToken>
```

The JWT token should be a base64-encoded string containing your Twilio Account SID and Auth Token separated by a colon.

## Base URL
```
POST /meta/<endpoint>
```

## Available Endpoints

### 1. Validate Phone Number
**Endpoint:** `POST /meta/validate`

Validates a phone number format and returns the formatted international number.

#### Request Body
```json
{
  "country": "US",
  "phone": "1234567890"
}
```

#### Parameters
- `country` (string, required): ISO country code (e.g., "US", "GB", "IN")
- `phone` (string, required): Phone number to validate

#### Response
```json
{
  "json": true,
  "success": true,
  "body": {
    "valid": true,
    "formatted_number": "+11234567890"
  },
  "raw": {
    "validateNumber": {
      // Raw validation response
    }
  }
}
```

#### Error Response
```json
{
  "json": true,
  "success": false,
  "error": "Missing country or phone parameters",
  "body": null
}
```

---

### 2. Register Sender
**Endpoint:** `POST /meta/register`

Registers a new WhatsApp Business sender with Twilio.

#### Request Body
```json
{
  "country": "US",
  "phone": "1234567890",
  "displayName": "My Business",
  "logoUrl": "https://example.com/logo.png",
  "webhookUrl": "https://example.com/webhook",
  "waba": "your-waba-id"
}
```

#### Parameters
- `country` (string, required): ISO country code
- `phone` (string, required): Phone number to register
- `displayName` (string, required): Display name for the sender
- `logoUrl` (string, optional): URL for the business logo
- `webhookUrl` (string, optional): Webhook URL for message callbacks
- `waba` (string, required): WhatsApp Business Account ID

#### Response
```json
{
  "json": true,
  "success": true,
  "body": {
    "registration": {
      "sid": "sender-id-12345",
      // Other Twilio registration response fields
    },
    "phone_number": "+11234567890",
    "senderID": "sender-id-12345"
  },
  "raw": {
    "update": {
      // Raw update response if logo/webhook were provided
    },
    "register": {
      // Raw registration response
    }
  }
}
```

#### Error Responses
```json
{
  "json": true,
  "success": false,
  "error": "Missing required parameters: country, phone, displayName",
  "body": null
}
```

```json
{
  "json": true,
  "success": false,
  "error": "Invalid phone number",
  "body": null
}
```

---

### 3. Verify Sender
**Endpoint:** `POST /meta/verify`

Verifies a sender registration using the verification code received via SMS/WhatsApp.

#### Request Body
```json
{
  "senderID": "sender-id-12345",
  "code": "123456"
}
```

#### Parameters
- `senderID` (string, required): The sender ID returned from registration
- `code` (string, required): 6-digit verification code

#### Response
```json
{
  "json": true,
  "success": true,
  "body": {
    "verification": {
      // Twilio verification response
    },
    "senderID": "sender-id-12345"
  },
  "raw": {
    "verify": {
      // Raw verification response
    }
  }
}
```

#### Error Response
```json
{
  "json": true,
  "success": false,
  "error": "Missing senderID or verification code",
  "body": null
}
```

---

### 4. Read Sender Information
**Endpoint:** `POST /meta/sender`

Retrieves information about a registered sender.

#### Request Body
```json
{
  "senderID": "sender-id-12345"
}
```

#### Parameters
- `senderID` (string, required): The sender ID to query

#### Response
```json
{
  "json": true,
  "success": true,
  "body": {
    "sender": {
      // Complete sender information from Twilio
    },
    "senderID": "sender-id-12345"
  }
}
```

#### Error Response
```json
{
  "json": true,
  "success": false,
  "error": "Missing senderID parameter",
  "body": null
}
```

---

### 5. Update Sender
**Endpoint:** `POST /meta/update`

Updates an existing sender's configuration.

#### Request Body
```json
{
  "senderID": "sender-id-12345",
  "data": {
    "profile": {
      "logo_url": "https://example.com/new-logo.png"
    },
    "webhook": {
      "callback_method": "POST",
      "callback_url": "https://example.com/webhook",
      "fallback_method": "POST",
      "fallback_url": "https://example.com/webhook/retry",
      "status_callback_url": "https://example.com/webhook/status"
    }
  }
}
```

#### Parameters
- `senderID` (string, required): The sender ID to update
- `data` (object, required): Update data object containing:
  - `profile` (object, optional): Profile information including logo_url
  - `webhook` (object, optional): Webhook configuration

#### Response
```json
{
  "json": true,
  "success": true,
  "body": {
    "update": {
      // Twilio update response
    },
    "senderID": "sender-id-12345"
  }
}
```

#### Error Response
```json
{
  "json": true,
  "success": false,
  "error": "Missing senderID or update data",
  "body": null
}
```

---

## Error Handling

All endpoints return consistent error responses:

```json
{
  "json": true,
  "success": false,
  "error": "Error description",
  "body": null
}
```

Common error scenarios:
- **401 Unauthorized**: Invalid or missing JWT token
- **400 Bad Request**: Missing required parameters
- **500 Internal Server Error**: Server-side processing errors

## Example Usage

### JavaScript/Node.js
```javascript
const axios = require('axios');

// Encode your Twilio credentials
const accountSid = 'your-account-sid';
const authToken = 'your-auth-token';
const jwt = Buffer.from(`${accountSid}:${authToken}`).toString('base64');

// Validate a phone number
const response = await axios.post('/meta/validate', {
  country: 'US',
  phone: '1234567890'
}, {
  headers: {
    'Authorization': `Bearer ${jwt}`,
    'Content-Type': 'application/json'
  }
});

console.log(response.data);
```

### cURL
```bash
# Encode credentials (replace with your actual credentials)
JWT=$(echo -n "your-account-sid:your-auth-token" | base64)

# Validate phone number
curl -X POST /meta/validate \
  -H "Authorization: Bearer $JWT" \
  -H "Content-Type: application/json" \
  -d '{
    "country": "US",
    "phone": "1234567890"
  }'
```

## Workflow Example

1. **Validate** the phone number format
2. **Register** the sender with required information
3. **Verify** the sender using the received code
4. **Update** sender configuration as needed
5. **Read** sender information to confirm setup

## Rate Limits

The API inherits Twilio's rate limits for the underlying services. Please refer to Twilio's documentation for current rate limiting policies.

## Support

For technical support or questions about this API, please contact the development team.
