# SatoshiTickets Greenfield API

## Interactive Documentation

After installing the plugin, open the interactive API docs in your browser:

```
https://YOUR_BTCPAY_SERVER/plugins/satoshi-tickets/api-docs
```

The docs are powered by [Redocly](https://redocly.com/) and generated from the OpenAPI 3.0.3 specification.

## OpenAPI Specification

The raw OpenAPI spec (swagger.json) is available at:

```
https://YOUR_BTCPAY_SERVER/plugins/satoshi-tickets/api-docs/swagger.json
```

You can import this URL directly into [Postman](https://www.postman.com/), [Swagger Editor](https://editor.swagger.io/), or any other OpenAPI-compatible tool.

## Authentication

All endpoints require a BTCPay Server Greenfield API key:

```
Authorization: token YOUR_API_KEY
```

Create an API key under **Account > Manage Account > API Keys** in BTCPay Server.

**Required permission:** `btcpay.store.canmodifystoresettings`

## Base URL

```
/api/v1/stores/{storeId}/satoshi-tickets
```

## Quick Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| **Events** | | |
| `GET` | `.../events` | List active events |
| `GET` | `.../events/{eventId}` | Get single event |
| `POST` | `.../events` | Create event |
| `PUT` | `.../events/{eventId}` | Update event |
| `DELETE` | `.../events/{eventId}` | Delete event |
| `PUT` | `.../events/{eventId}/toggle` | Toggle Active/Disabled |
| `POST` | `.../events/{eventId}/logo` | Upload event logo (multipart) |
| `DELETE` | `.../events/{eventId}/logo` | Remove event logo |
| **Ticket Types** | | |
| `GET` | `.../events/{eventId}/ticket-types` | List ticket types |
| `GET` | `.../events/{eventId}/ticket-types/{id}` | Get ticket type |
| `POST` | `.../events/{eventId}/ticket-types` | Create ticket type |
| `PUT` | `.../events/{eventId}/ticket-types/{id}` | Update ticket type |
| `DELETE` | `.../ticket-types/{id}` | Delete ticket type |
| `PUT` | `.../ticket-types/{id}/toggle` | Toggle Active/Disabled |
| **Tickets** | | |
| `GET` | `.../events/{eventId}/tickets` | List settled tickets |
| `GET` | `.../events/{eventId}/tickets/export` | Export tickets CSV |
| `POST` | `.../events/{eventId}/tickets/{ticketNumber}/check-in` | Check-in ticket |
| **Orders** | | |
| `POST` | `.../events/{eventId}/purchase` | Create ticket purchase (returns order + checkout URL) |
| `GET` | `.../events/{eventId}/orders` | List settled orders |
| `POST` | `.../events/{eventId}/orders/{orderId}/tickets/{ticketId}/send-reminder` | Re-send ticket email |

## Purchase Flow (API)

Use `POST .../events/{eventId}/purchase` to create ticket orders programmatically (e.g. from WooCommerce or other e-commerce integrations):

**Request:**
```json
{
  "tickets": [
    {
      "ticketTypeId": "uuid-ticket-type",
      "quantity": 2,
      "recipients": [
        { "firstName": "Jano", "lastName": "Novak", "email": "jano@example.com" },
        { "firstName": "Maria", "lastName": "Novakova", "email": "maria@example.com" }
      ]
    }
  ],
  "redirectUrl": "https://example.com/thank-you"
}
```

**Response (201 Created):**
```json
{
  "orderId": "uuid",
  "txnId": "string",
  "invoiceId": "btcpay-invoice-id",
  "checkoutUrl": "https://btcpay.example.com/i/{invoiceId}"
}
```

**Validation:** Event must be active and in the future; ticket types must exist and have sufficient quantity; `recipients.length` must equal `quantity` for each ticket type.

## Typical Workflow

1. **Create event** — `POST .../events` (returns `Disabled` state)
2. **Upload logo** (optional) — `POST .../events/{id}/logo`
3. **Create ticket types** — `POST .../events/{id}/ticket-types`
4. **Activate event** — `PUT .../events/{id}/toggle`
5. **Monitor sales** — `GET .../events/{id}/tickets` or `.../orders`
6. **Create purchase via API** (optional) — `POST .../events/{id}/purchase` — for external integrations
7. **Check-in** — `POST .../events/{id}/tickets/{ticketNumber}/check-in`
8. **Export report** — `GET .../events/{id}/tickets/export`
