## What is a Webhook?

- A **webhook** is an HTTP callback: one server automatically sends an HTTP request to another server when a specific event happens.
- Instead of your app repeatedly asking another service for updates, the other service **pushes** data to your endpoint.
- Usually implemented as an HTTP `POST` request containing event data in JSON.
- Webhooks are event-driven and provide near-real-time communication between applications.

> [!definition]
> A webhook is a URL on your server that another service calls automatically when an event occurs.

## Real-Life Example

### Payment Example

1. User places an order on your e-commerce website.
2. User pays using Razorpay, Stripe, PayPal, etc.
3. The payment gateway processes the payment.
4. When payment succeeds, the gateway sends a `POST` request to your backend webhook URL.
5. Your backend receives the event and marks the order as paid.

Example webhook URL:

```text
[https://yourapp.com/api/webhooks/payment](https://yourapp.com/api/webhooks/payment)
```

Example payload:

```json
{
  "type": "payment.captured",
  "data": {
    "orderId": "ORD_123",
    "paymentId": "PAY_456",
    "amount": 49900
  }
}
```

Your backend can then:

- Mark the order as `PAID`
- Store the payment ID
- Reduce product stock
- Send an order confirmation email
- Trigger delivery or invoice generation

## Webhook vs Polling

| Method | How it works | Example | Flaws |
|---|---|---|---|
| **Polling** | Client repeatedly asks the server for updates | Call `GET /payments/order_123` every 5 seconds | Wasted requests, delayed updates, rate limits, higher cost |
| **Webhook** | Server calls your URL when an event occurs | Payment provider calls `/api/webhooks/payment` on payment success | Needs public HTTPS URL, must verify sender, retries possible, needs idempotency |

### Polling

- Your server keeps checking another API at fixed intervals.
- Example: every 5 seconds, ask the payment API, “Is the payment completed?”
- Simple to implement, but inefficient.

### Webhook

- The payment provider notifies your server immediately after payment succeeds.
- No unnecessary repeated requests.
- Better for real-time event-driven systems.

> [!tip] Analogy
> - **Polling** = checking your phone every minute to see if someone messaged you.
> - **Webhook** = the person calls you immediately when they have news.

## Why Webhooks Are Useful

Webhooks are helpful when your app needs to react to events happening in another system.

### Common use cases

- **Payments:** Confirm successful or failed payments.
- **GitHub:** Trigger CI/CD or deployment when code is pushed.
- **Messaging apps:** Send Slack or Discord alerts for events.
- **Email services:** Track email delivered, opened, bounced, or clicked.
- **File/video processing:** Notify app when a file upload or video conversion finishes.
- **CRM tools:** Sync leads, contacts, or customer updates automatically.

## Webhook Flow

```text
Event occurs in Service A
        ↓
Service A sends HTTP POST request
        ↓
Your webhook endpoint receives request
        ↓
Verify webhook signature
        ↓
Process event / save data / trigger action
        ↓
Send 200 OK response
```

## Express Webhook Example

Install Express:

```bash
npm install express
```

### Basic Express Webhook Server

```js
import express from "express";

const app = express();

app.use(
  "/api/webhooks/payment",
  express.raw({ type: "application/json" })
);

app.post("/api/webhooks/payment", (req, res) => {
  try {
    const event = JSON.parse(req.body.toString());

    console.log("Received webhook event:", event);

    if (event.type === "payment.captured") {
      const { orderId, paymentId } = event.data;

      console.log(`Payment successful for order: ${orderId}`);

      // Real application logic:
      // await Order.updateOne(
      //   { _id: orderId },
      //   {
      //     status: "PAID",
      //     paymentId,
      //     paidAt: new Date()
      //   }
      // );
    }

    res.status(200).json({ received: true });
  } catch (error) {
    console.error("Webhook error:", error);
    res.status(400).json({ error: "Invalid webhook payload" });
  }
});

app.listen(3000, () => {
  console.log("Webhook server running on port 3000");
});
```

## Important Production Rules

### 1. Verify the Signature

- Anyone can send a fake request to your webhook URL.
- Payment providers sign webhook requests using a secret key.
- Always verify the signature before trusting the payload.

```js
const isValid = verifyWebhookSignature(rawBody, signature, webhookSecret);

if (!isValid) {
  return res.status(401).json({ error: "Invalid signature" });
}
```

### 2. Use Idempotency

- A provider may send the same webhook more than once.
- Store a unique `eventId`, `paymentId`, or transaction ID.
- If the event was already processed, ignore it.

```js
const existing = await WebhookEvent.findOne({ eventId: event.id });

if (existing) {
  return res.status(200).json({ received: true, duplicate: true });
}

await WebhookEvent.create({ eventId: event.id });
```

### 3. Respond Quickly

- Do not perform slow tasks directly inside the webhook handler.
- First return `200 OK`, then process the event using a queue.

```js
res.status(200).json({ received: true });

await paymentQueue.add("process-payment", event);
```

### 4. Use HTTPS

- Webhook endpoints must be publicly accessible.
- Always use HTTPS in production.

### 5. Handle Retries

- If your server returns `500`, the provider may retry the webhook later.
- Make your handler safe to run multiple times.

### 6. Log Webhook Events

- Store incoming webhook payloads for debugging.
- Useful when debugging failed payments or missing updates.

## When to Use Webhooks

### Use webhooks when

- You need immediate updates from another service.
- Polling would create too many unnecessary requests.
- The event may happen after the user leaves your website.
- You are integrating with third-party services like Stripe, Razorpay, GitHub, Slack, Shopify, etc.

### Use polling when

- The external service does not support webhooks.
- You need very simple local development.
- You need to check status only occasionally.

## Interview-Ready Answer

> A webhook is an event-driven HTTP callback. Instead of my server polling another service for updates, the service sends a POST request to my registered endpoint when an event occurs. For example, when a Razorpay payment succeeds, Razorpay calls my `/api/webhooks/payment` endpoint. My Express backend verifies the request signature, checks for duplicate events, updates the order status, and queues any follow-up work. Webhooks are better than polling because they provide near-real-time updates, reduce unnecessary API calls, and scale better.