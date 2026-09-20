# Piknik.Spot REST API Skill

## Overview
REST API for local food system integration with OAuth2 authentication.

- **Base URL**: `https://piknik.spot/api`
- **Full Docs**: `https://piknik.spot/api/docs`
- **OpenAPI**: `https://piknik.spot/api/openapi`

## Authentication

### OAuth 2.0 (Recommended)
```
Authorization: https://piknik.spot/api/oauth/authorize
Token: https://piknik.spot/api/oauth/token
Header: Authorization: Bearer {access_token}
```

**Scopes**: `marketplace:write`, `events:write`, `read:profile`, `write:profile`

### Quick OAuth Flow
1. Direct user to authorize URL with your `client_id`
2. Exchange authorization code for access token at token endpoint
3. Use bearer token in `Authorization` header for API calls

## Essential Endpoints

### Get Business IDs (Required First)
```http
GET /api/v2/participants/my-businesses
Authorization: Bearer {token}
```
**Returns**: List of user's businesses with IDs needed for creating listings/events.

### Marketplace Listings

**List**: `GET /api/listings/my`

**Create**: `POST /api/listings`
```json
{
  "participant_id": "business_id",
  "listing_type": "wanted_to_sell|wanted_to_buy|service_offering",
  "title": "Product name",
  "description": "Details",
  "contact_method": "email|phone|message",
  "end_date": "YYYY-MM-DD"
}
```

**Manage**: 
- Accept: `POST /api/listings/{id}/accept`
- Decline: `POST /api/listings/{id}/decline`
- Fulfill: `POST /api/listings/{id}/fulfill`

### Events

**List**: `GET /api/events` (supports location filters)

**Create**: `POST /api/events`
```json
{
  "participant_id": "business_id",
  "title": "Event name",
  "description": "Details",
  "start_date": "2026-03-15T09:00:00Z",
  "end_date": "2026-03-15T14:00:00Z",
  "address": "Full address",
  "event_type": "market|workshop|tour|festival|meeting|other",
  "is_recurring": false,
  "recurrence_rule": "FREQ=WEEKLY;BYDAY=SA"
}
```

### Participants (Businesses)

**Search**: `GET /api/v2/participants?latitude={lat}&longitude={lng}&maxDistance={km}`

**Details**: `GET /api/v2/participants/{id}`

**Update**: `PUT /api/v2/participants/{id}`

**Nearby**: `GET /api/v2/participants/nearby?latitude={lat}&longitude={lng}&radius={km}`

### Products & Offerings

**List**: `GET /api/v2/participants/{id}/offerings`

**Create**: `POST /api/v2/participants/{id}/offerings`

**Browse**: `GET /api/browse-products?search={term}&latitude={lat}&longitude={lng}`

### User Profile

**Current User**: `GET /api/me`

**Profile**: `GET /api/profile/{username}`

### Associations

**Available**: `GET /api/v2/associations/available`

**Details**: `GET /api/v2/associations/{slug}`

**Members**: `GET /api/v2/associations/{slug}/members`

### CSA

**Interests**: `GET /api/v2/csa/interests`

**Availability**: `GET /api/v2/participants/{id}/csa-availability`

**Analytics**: `GET /api/v2/participants/{id}/csa-analytics`

**Demand Map**: `GET /api/v2/csa/demand-map?latitude={lat}&longitude={lng}&radius={km}`

### Geolocation

**Geocode**: `GET /api/geocode?address={address}`

Returns `{ latitude, longitude }` for address string.

### Reviews

**List**: `GET /api/v2/participants/{id}/reviews`

**Create**: `POST /api/v2/participants/{id}/reviews`
```json
{
  "rating": 5,
  "comment": "Great service!"
}
```

## Quick Start Pattern

```javascript
// 1. Get businesses
const { businesses } = await fetch('/api/v2/participants/my-businesses', {
  headers: { 'Authorization': `Bearer ${token}` }
}).then(r => r.json());

// 2. Create listing
await fetch('/api/listings', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    participant_id: businesses[0].id,
    listing_type: 'wanted_to_sell',
    title: 'Fresh Eggs',
    description: 'Free-range eggs available',
    contact_method: 'email',
    end_date: '2026-12-31'
  })
});
```

## Error Handling

**Status Codes**: `200` OK, `201` Created, `400` Bad Request, `401` Unauthorized, `403` Forbidden, `404` Not Found, `429` Rate Limited

**Response**:
```json
{
  "error": "Error message",
  "error_description": "Details"
}
```

## Best Practices

1. **Always fetch `participant_id`** from `/api/v2/participants/my-businesses` first
2. **Use `Content-Type: application/json`** header for POST/PUT
3. **Handle 401** by refreshing access token
4. **Validate dates** in ISO 8601 format
5. **Cache geocode results** to reduce API calls

## Full Documentation

For complete endpoint details, request/response schemas, and examples:
- Interactive API Docs: https://piknik.spot/api/docs
- OpenAPI Spec: https://piknik.spot/api/openapi

## Support

- Email: info@piknik.spot
- Main Site: https://piknik.spot
