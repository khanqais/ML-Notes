---
tags:
  - web-security
  - client-side-security
  - api-security
  - cors
status: learning
difficulty: beginner
related:
  - "[[Same-Origin Policy]]"
  - "[[Authentication vs Authorization]]"
  - "[[CSRF]]"
  - "[[Rate Limiting]]"
  - "[[API Security]]"
---

# CORS — Cross-Origin Resource Sharing

## Definition

**CORS** stands for **Cross-Origin Resource Sharing**.

CORS is a browser security mechanism that allows a backend server to specify which frontend origins are allowed to read responses from its API.

```txt
CORS controls browser access to cross-origin responses.
```

It is based on the browser's [[Same-Origin Policy]].

---

## Origin

An **origin** has three parts:

```txt
protocol + domain + port
```

Examples:

```txt
http://localhost:3000
http://localhost:5173
[https://shopnest.com](https://shopnest.com)
[https://api.shopnest.com](https://api.shopnest.com)
```

These are different origins:

```txt
http://localhost:3000
http://localhost:5173
```

Reason:

```txt
Different port
```

These are also different origins:

```txt
[https://shopnest.com](https://shopnest.com)
[https://api.shopnest.com](https://api.shopnest.com)
```

Reason:

```txt
Different subdomain
```

A path is not part of an origin:

```txt
[https://shopnest.com/dashboard](https://shopnest.com/dashboard)
[https://shopnest.com/api/products](https://shopnest.com/api/products)
```

Both have the same origin:

```txt
[https://shopnest.com](https://shopnest.com)
```

---

## Problem Scenario

Suppose ShopNest has:

```txt
Frontend: [https://shopnest.com](https://shopnest.com)
Backend:  [https://api.shopnest.com](https://api.shopnest.com)
```

The frontend needs to call the backend API:

```js
fetch("[https://api.shopnest.com/api/products](https://api.shopnest.com/api/products)");
```

Because the frontend and backend are on different origins, the browser applies the Same-Origin Policy.

The browser needs confirmation from the backend:

```txt
“Does this API allow JavaScript running on shopnest.com
to read this response?”
```

CORS provides that confirmation.

---

## How CORS Works

### Step 1: Browser sends Origin header

When the frontend makes a request, the browser includes an `Origin` header:

```http
GET /api/products HTTP/1.1
Host: api.shopnest.com
Origin: [https://shopnest.com](https://shopnest.com)
```

### Step 2: Backend checks the origin

The backend checks whether the origin is in its trusted allowlist:

```txt
[https://shopnest.com](https://shopnest.com)
```

### Step 3: Backend sends CORS response headers

```http
Access-Control-Allow-Origin: [https://shopnest.com](https://shopnest.com)
```

### Step 4: Browser allows frontend JavaScript to read response

If the origin matches, browser JavaScript receives the response:

```js
const response = await fetch("[https://api.shopnest.com/api/products](https://api.shopnest.com/api/products)");
const products = await response.json();
```

---

## Important Correction

### Wrong understanding

```txt
CORS stops every other frontend, curl, Postman, bots,
or attacker from sending requests to my backend.
```

### Correct understanding

```txt
CORS mainly controls whether browser JavaScript can read
a cross-origin response.
```

CORS does **not** block direct requests made through:

```txt
curl
Postman
Python scripts
Mobile apps
Backend servers
Bots
Attack tools
```

Example:

```bash
curl [https://api.shopnest.com/api/products](https://api.shopnest.com/api/products)
```

This request can still reach the backend even if CORS allows only:

```txt
[https://shopnest.com](https://shopnest.com)
```

Therefore:

```txt
CORS is not authentication.
CORS is not authorization.
CORS is not a replacement for backend security.
```

---

## CORS and Security

CORS answers:

```txt
Can browser JavaScript from this origin read my API response?
```

Authentication answers:

```txt
Who is making this request?
```

Authorization answers:

```txt
Is this user allowed to perform this action?
```

Rate limiting answers:

```txt
How frequently can this user/IP call this endpoint?
```

CSRF protection answers:

```txt
Did this cookie-authenticated state-changing request
come from my expected application flow?
```

Input validation answers:

```txt
Is the received request data valid and safe?
```

---

## Example: CORS Cannot Protect an Unauthenticated API

Suppose this endpoint is public:

```ts
app.get("/api/products", async (req, res) => {
  const products = await Product.find();
  res.json(products);
});
```

Even if CORS only allows:

```txt
[https://shopnest.com](https://shopnest.com)
```

an attacker can still call:

```bash
curl [https://api.shopnest.com/api/products](https://api.shopnest.com/api/products)
```

Therefore, CORS does not make an endpoint private.

If the data is private, add authentication and authorization:

```ts
app.get("/api/orders/:id", authenticate, async (req, res) => {
  const order = await Order.findOne({
    _id: req.params.id,
    userId: req.user.id
  });

  if (!order) {
    return res.status(404).json({
      error: "Order not found"
    });
  }

  res.json(order);
});
```

See:

- [[Authentication vs Authorization]]
- [[IDOR and BOLA]]
- [[Broken Access Control]]

---

## Basic Express Setup

Install the package:

```bash
npm install cors
```

Basic development configuration:

```ts
import express from "express";
import cors from "cors";

const app = express();

app.use(
  cors({
    origin: "http://localhost:3000"
  })
);
```

Use this when:

```txt
Frontend: http://localhost:3000
Backend:  http://localhost:5000
```

The frontend can call the backend, but another browser origin is not allowed to read the response.

---

## Production CORS Configuration

Use a trusted origin allowlist:

```ts
import express from "express";
import cors from "cors";

const app = express();

const allowedOrigins = [
  "[https://shopnest.com](https://shopnest.com)",
  "[https://www.shopnest.com](https://www.shopnest.com)"
];

app.use(
  cors({
    origin(origin, callback) {
      if (!origin || allowedOrigins.includes(origin)) {
        return callback(null, true);
      }

      return callback(new Error("Origin not allowed by CORS"));
    }
  })
);
```

### Why `!origin` is allowed

Some requests do not have an `Origin` header:

```txt
curl
Postman
Server-to-server calls
Monitoring tools
Health checks
```

CORS is a browser policy, so requests without an `Origin` header should be handled using authentication and authorization, not CORS alone.

---

## Cookies and Credentials

If authentication uses cookies, configure CORS carefully.

### Backend

```ts
app.use(
  cors({
    origin: "[https://shopnest.com](https://shopnest.com)",
    credentials: true
  })
);
```

The backend sends headers similar to:

```http
Access-Control-Allow-Origin: [https://shopnest.com](https://shopnest.com)
Access-Control-Allow-Credentials: true
```

### Frontend

```ts
const response = await fetch(
  "[https://api.shopnest.com/api/orders](https://api.shopnest.com/api/orders)",
  {
    method: "GET",
    credentials: "include"
  }
);
```

`credentials: "include"` tells the browser to include cookies with the request.

---

## Important Credentials Rule

This is unsafe and invalid:

```ts
app.use(
  cors({
    origin: "*",
    credentials: true
  })
);
```

Do not use:

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

For cookie-authenticated requests, always use an exact trusted origin:

```ts
app.use(
  cors({
    origin: "[https://shopnest.com](https://shopnest.com)",
    credentials: true
  })
);
```

---

## Preflight Request

Some requests trigger a browser **preflight request**.

A preflight is an `OPTIONS` request sent before the actual request.

Example actual request:

```http
DELETE /api/orders/order-123
Origin: [https://shopnest.com](https://shopnest.com)
Authorization: Bearer <jwt>
Content-Type: application/json
```

Before sending the `DELETE` request, the browser may send:

```http
OPTIONS /api/orders/order-123
Origin: [https://shopnest.com](https://shopnest.com)
Access-Control-Request-Method: DELETE
Access-Control-Request-Headers: authorization, content-type
```

The backend must respond with allowed details:

```http
Access-Control-Allow-Origin: [https://shopnest.com](https://shopnest.com)
Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
```

Then the browser sends the actual request.

---

## Express Preflight Configuration

```ts
app.use(
  cors({
    origin: "[https://shopnest.com](https://shopnest.com)",
    methods: ["GET", "POST", "PUT", "PATCH", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"],
    credentials: true
  })
);
```

This allows:

```txt
Methods:
GET, POST, PUT, PATCH, DELETE

Request Headers:
Content-Type, Authorization

Origin:
[https://shopnest.com](https://shopnest.com)

Cookies:
Allowed
```

---

## Common CORS Mistakes

### 1. Allowing every origin

```ts
app.use(cors());
```

This may return:

```http
Access-Control-Allow-Origin: *
```

This is only acceptable for truly public APIs and public resources.

Examples where `*` may be acceptable:

```txt
Public documentation API
Public weather data
Public blog posts
Public product catalog with no private data
```

Do not use `*` blindly for:

```txt
User profiles
Orders
Payments
Private dashboards
Admin APIs
Authenticated APIs
```

---

### 2. Reflecting every request origin

Unsafe code:

```ts
app.use(
  cors({
    origin: true,
    credentials: true
  })
);
```

Unsafe manual code:

```ts
res.setHeader(
  "Access-Control-Allow-Origin",
  req.headers.origin
);
```

without checking an allowlist.

An attacker can send:

```http
Origin: [https://evil-site.com](https://evil-site.com)
```

If your backend reflects it:

```http
Access-Control-Allow-Origin: [https://evil-site.com](https://evil-site.com)
Access-Control-Allow-Credentials: true
```

then browser JavaScript running on `evil-site.com` may be able to read authenticated responses.

### Correct solution

Use an exact allowlist:

```ts
const allowedOrigins = [
  "[https://shopnest.com](https://shopnest.com)",
  "[https://www.shopnest.com](https://www.shopnest.com)"
];
```

---

### 3. Treating CORS as backend authorization

Wrong assumption:

```txt
Only my frontend is allowed in CORS,
therefore my API is secure.
```

Correct approach:

```txt
CORS controls browser response sharing.

JWT/session authentication verifies identity.

Authorization checks roles and ownership.

Input validation validates data.

Rate limiting limits abuse.
```

---

### 4. Adding paths to the allowed origin

Wrong:

```ts
origin: "[https://shopnest.com/dashboard](https://shopnest.com/dashboard)"
```

Correct:

```ts
origin: "[https://shopnest.com](https://shopnest.com)"
```

CORS validates origins, not URL paths.

---

## CORS vs Other Security Concepts

| Concept | Main purpose | Example question |
|---|---|---|
| CORS | Controls which browser origin can read a cross-origin response | Can `shopnest.com` read this API response? |
| [[Authentication]] | Verifies identity | Who is making the request? |
| [[Authorization]] | Checks permissions | Can this user delete this product? |
| [[IDOR and BOLA]] | Checks object ownership | Can this user access this particular order? |
| [[CSRF]] | Prevents forged cookie-based actions | Was this action initiated through my legitimate app? |
| [[Rate Limiting]] | Limits request frequency | How many login attempts are allowed? |
| [[Input Validation]] | Validates request data | Is this UUID and quantity valid? |

---

## Project Example: ShopNest

```txt
Frontend:
[https://shopnest.com](https://shopnest.com)

Backend:
[https://api.shopnest.com](https://api.shopnest.com)
```

Recommended configuration:

```ts
const allowedOrigins = [
  "[https://shopnest.com](https://shopnest.com)",
  "[https://www.shopnest.com](https://www.shopnest.com)",
  "http://localhost:3000"
];

app.use(
  cors({
    origin(origin, callback) {
      if (!origin || allowedOrigins.includes(origin)) {
        return callback(null, true);
      }

      return callback(new Error("CORS origin denied"));
    },
    methods: ["GET", "POST", "PUT", "PATCH", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"],
    credentials: true
  })
);
```

Additional security still required:

```txt
- JWT or session validation
- Admin role checks
- Order ownership checks
- Zod input validation
- Rate limiting on login and checkout
- CSRF protection if cookies authenticate requests
- Security headers
```

---

## Testing Checklist

- [ ] Test from the allowed production frontend origin
- [ ] Test from the allowed local development origin
- [ ] Test from an untrusted origin such as `https://evil-site.com`
- [ ] Verify private API routes require authentication
- [ ] Verify users cannot access another user's resource
- [ ] Verify admin routes require the `ADMIN` role
- [ ] Verify cookie-authenticated requests use `credentials: "include"`
- [ ] Verify backend uses `credentials: true` only with exact origins
- [ ] Verify `Access-Control-Allow-Origin: *` is not used for private APIs
- [ ] Verify `OPTIONS` preflight requests succeed for required methods and headers
- [ ] Verify rate limiting exists for login, signup, OTP, password reset, and checkout endpoints

---

## Key Takeaways

```txt
1. CORS means Cross-Origin Resource Sharing.

2. CORS is enforced by browsers.

3. CORS controls whether JavaScript from another origin
   can read an API response.

4. CORS does not block Postman, curl, bots, server scripts,
   mobile apps, or direct API calls.

5. CORS is not authentication or authorization.

6. Use exact trusted origins in an allowlist.

7. Do not use `*` with credentialed requests.

8. For cookie-based authentication:
   - Backend: credentials: true
   - Frontend: credentials: "include"
   - Backend: exact allowed origin

9. Use CORS together with authentication, authorization,
   ownership checks, input validation, CSRF protection,
   and rate limiting.
```

---

## Interview Answer

> CORS is a browser-enforced mechanism that controls whether JavaScript from one origin can read responses from another origin. The API uses headers such as `Access-Control-Allow-Origin` to allow only trusted frontend origins, for example `https://shopnest.com`. However, CORS is not authentication or authorization because it does not block Postman, curl, scripts, or direct API calls. Therefore, I still protect backend endpoints with session or JWT validation, role-based authorization, ownership checks, input validation, and rate limiting.

---

## Related Notes

- [[Same-Origin Policy]]
- [[Authentication vs Authorization]]
- [[JWT Authentication]]
- [[Session Management]]
- [[CSRF]]
- [[IDOR and BOLA]]
- [[Input Validation]]
- [[Rate Limiting]]
- [[Security Headers]]
- [[API Security]]

## References

- [MDN — Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)
- [MDN — Same-Origin Policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [OWASP — CORS Origin Header Scrutiny](https://owasp.org/www-community/attacks/CORS_OriginHeaderScrutiny)