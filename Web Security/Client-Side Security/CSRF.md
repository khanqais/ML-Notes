---
tags:
  - web-security
  - client-side-security
  - csrf
  - authentication
status: learning
difficulty: intermediate
related:
  - - - Cookies
  - - - SameSite Cookies
  - - - CORS
  - - - XSS
  - - - Authentication vs Authorization
---

# Cross-Site Request Forgery (CSRF)

## Definition

Cross-Site Request Forgery (CSRF) is an attack where an attacker tricks
a victim's browser into sending an unwanted request to a trusted website
where the victim is already authenticated.

The target server sees the victim's valid session cookie and may think
the request was intentionally made by the victim.

## Core idea

```txt
Victim is logged in to bank.com
        ↓
Victim visits evil.com
        ↓
evil.com makes victim's browser send a request to bank.com
        ↓
Browser automatically includes bank.com session cookie
        ↓
bank.com accepts the request if it has no CSRF protection
```

## Important correction

Classic CSRF commonly affects cookie-based authentication.

The attacker usually cannot read:

- Cookies from `bank.com`
- JWTs in `localStorage` from `bank.com`
- Responses from `bank.com`

The attacker does not need to read the cookie.

The browser automatically attaches the victim's `bank.com` cookie when it
sends a request to `bank.com`, unless cookie protections such as `SameSite`
prevent it.

## Attack requirements

CSRF is possible when:

1. The victim is logged in to the target website.
2. The target website uses cookie-based authentication.
3. The browser automatically includes the session cookie.
4. The target endpoint changes server-side data.
5. The server does not verify that the request came from its own frontend.
6. The attacker can predict or control the request format.

## Problem scenario: Bank transfer

Victim is logged in to:

```txt
[https://bank.com](https://bank.com)
```

The bank stores a session cookie in the victim's browser:

```http
Set-Cookie: sessionId=abc123; HttpOnly; Secure
```

The victim then visits an attacker-controlled website:

```txt
[https://evil.com](https://evil.com)
```

The attacker puts this hidden form on `evil.com`:

```html
<form id="transferForm" action="[https://bank.com/api/transfer](https://bank.com/api/transfer)" method="POST">
  <input type="hidden" name="toAccount" value="attacker-account-123" />
  <input type="hidden" name="amount" value="5000" />
</form>

<script>
  document.getElementById("transferForm").submit();
</script>
```

The victim's browser sends:

```http
POST /api/transfer HTTP/1.1
Host: bank.com
Cookie: sessionId=abc123
Content-Type: application/x-www-form-urlencoded

toAccount=attacker-account-123&amount=5000
```

If `bank.com` only checks the session cookie, it may process the transfer.

## Why the attack works

The bank backend sees:

```txt
Valid session cookie = valid logged-in user
```

But it does not verify:

```txt
Did this request actually originate from bank.com's own frontend?
```

The browser sends the cookie automatically, so authentication succeeds even
though the user never intended to transfer money.

## Vulnerable backend example

```ts
export async function POST(req: Request) {
  const user = await getUserFromSessionCookie(req);

  if (!user) {
    return Response.json({ error: "Unauthorized" }, { status: 401 });
  }

  const { toAccount, amount } = await req.json();

  await transferMoney({
    fromUserId: user.id,
    toAccount,
    amount
  });

  return Response.json({ message: "Transfer successful" });
}
```

This route verifies that the user is logged in, but it does not verify that
the request was intentionally initiated from the trusted application.

## Solution 1: CSRF token

For every state-changing request, require a random CSRF token.

State-changing requests include:

- POST
- PUT
- PATCH
- DELETE

Example request from the real frontend:

```http
POST /api/transfer
Cookie: sessionId=abc123
X-CSRF-Token: random-secret-value
Content-Type: application/json
```

The attacker can cause a browser to send a form request, but cannot normally
read the CSRF token from `bank.com` and set it in a custom request header.

## CSRF token flow

```txt
1. User logs in to bank.com.
2. Server creates a random CSRF token.
3. Real bank frontend receives the token.
4. Frontend sends token in X-CSRF-Token header.
5. Backend validates the token.
6. Missing or invalid token → reject request.
```

## Example server-side validation

```ts
export async function POST(req: Request) {
  const user = await getUserFromSessionCookie(req);

  if (!user) {
    return Response.json({ error: "Unauthorized" }, { status: 401 });
  }

  const csrfToken = req.headers.get("x-csrf-token");

  if (!csrfToken || csrfToken !== user.csrfToken) {
    return Response.json({ error: "Invalid CSRF token" }, { status: 403 });
  }

  const { toAccount, amount } = await req.json();

  await transferMoney({
    fromUserId: