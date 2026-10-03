---
tags:
  - web-security
  - client-side-security
  - clickjacking
  - ui-redressing
status: learning
related:
  - "[[Content Security Policy]]"
  - "[[Security Headers]]"
  - "[[CSRF]]"
  - "[[Authentication Vulnerabilities]]"
---

# Clickjacking

> [!abstract]
> **Clickjacking** is a user-interface deception attack in which an attacker embeds a legitimate website inside a frame, hides or disguises it, and tricks the victim into clicking a real button, link, or control on that legitimate website.
>
> It is also known as **UI redressing**.

## Navigation

- [[Web Security Home]]
- [[Client-Side Security]]
- [[Content Security Policy]]
- [[Security Headers]]
- [[CSRF]]
- [[Security Testing Checklist]]

---

## How It Works

A typical clickjacking attack relies on the browser's ability to render one website inside another website using an HTML `<iframe>`.

The attacker creates a malicious page that:

1. Loads a trusted target site in an iframe.
2. Makes the iframe transparent, nearly transparent, hidden, or carefully positioned.
3. Displays a tempting fake interface over or around the iframe.
4. Causes the victim's click to land on a sensitive control in the framed target site.

The victim may already be authenticated to the target application, so their browser automatically sends the target site's session cookies with the framed request.

```text
Victim visits attacker-controlled page
            |
            v
Attacker embeds trusted-site.example in an iframe
            |
            v
Attacker overlays deceptive text or UI
            |
            v
Victim clicks what appears to be an innocent control
            |
            v
Click reaches real button in trusted iframe
            |
            v
Sensitive action may be performed under victim's session
```

> [!warning]
> Clickjacking targets the **user's click**, not necessarily a flaw in server-side authorization. A properly authenticated action can still be dangerous if an attacker can manipulate the user into approving it invisibly.

---

## Simple Example

Imagine a user is logged in to `bank.example`.

An attacker hosts a page that displays a fake button:

```html
<button>Click here to win a prize</button>
```

Behind or directly beneath that button, the attacker places a nearly invisible iframe containing a real button on `bank.example`, such as:

```text
Enable a new payment recipient
```

When the victim clicks **“Click here to win a prize,”** their click may activate the real banking action inside the transparent iframe instead.

> [!danger]
> Never test this pattern against systems you do not own or have explicit permission to assess.

---

## Attack Preconditions

For clickjacking to work, several conditions typically need to be true:

- The target page allows itself to be embedded in an iframe, frame, object, or embed element.
- The target page contains an action the attacker wants the victim to trigger.
- The victim is authenticated to the target site, or the action does not require authentication.
- The attacker can make the victim click at a precise location.
- Browser restrictions, site defenses, and user interaction requirements do not prevent the action.

Clickjacking is especially relevant for pages with sensitive buttons, such as:

- Changing account email addresses or passwords
- Granting permissions to an application
- Enabling two-factor authentication changes
- Adding a payment recipient
- Making a purchase
- Deleting data
- Publishing content
- Changing privacy settings
- Authorizing an OAuth application
- Confirming administrative actions

---

## Common Variants

### Transparent Overlay

The attacker uses CSS opacity to make the target iframe nearly invisible.

```css
.target-frame {
  position: absolute;
  opacity: 0.01;
  z-index: 2;
}
```

The attacker positions a visible lure underneath or above it so the victim thinks they are clicking something harmless.

### Hidden Frame

The attacker hides or moves the target frame and aligns one sensitive element with a visible lure. The entire target interface does not need to be visible; only the clickable region needs to align with the victim's click.

### Likejacking

**Likejacking** is a clickjacking-style attack designed to trick users into clicking social-media actions, such as liking, following, sharing, or subscribing.

### Cursorjacking

**Cursorjacking** manipulates cursor appearance or positioning to make a user believe their pointer is clicking one place while the actual click lands elsewhere.

### Drag-and-Drop Clickjacking

Some attacks abuse drag-and-drop interactions rather than a single click. A victim may drag an apparently harmless item but unknowingly drop data or trigger an action in a framed target application.

### Multi-Step Clickjacking

More complex attacks can guide a user through several carefully positioned clicks. Each click may trigger a different target action, making the malicious flow appear more convincing.

---

## Clickjacking vs CSRF

Clickjacking and [[CSRF]] can both cause unwanted actions through a victim's authenticated session, but they work differently.

| Aspect | Clickjacking | CSRF |
|---|---|---|
| Main technique | Tricks the user into clicking | Sends a forged request |
| User interaction | Usually requires a deliberate victim click | May occur without a visible user action |
| UI manipulation | Core part of the attack | Not required |
| Target embedding | Often uses an iframe | Does not require an iframe |
| Main defenses | `frame-ancestors`, `X-Frame-Options` | CSRF tokens, SameSite cookies, origin checks |
| Typical risk | User approves a disguised UI action | Browser submits an attacker-triggered request |

> [!important]
> Defend against both attacks. `SameSite` cookies and CSRF tokens do not replace frame-embedding protections, while `frame-ancestors` does not replace CSRF protections.

---

## Primary Defense: CSP frame-ancestors

The modern, preferred defense is the `frame-ancestors` directive in the HTTP `Content-Security-Policy` response header.

It controls which origins are allowed to embed the current page in `<frame>`, `<iframe>`, `<object>`, or `<embed>` elements. If an embedding ancestor does not match the policy, the browser cancels the load. [4][5]

### Block All Framing

Use this when no page, including your own site, needs to embed the protected page.

```http
Content-Security-Policy: frame-ancestors 'none';
```

This is the strongest default for sensitive pages such as:

- Account settings
- Administrative panels
- Billing pages
- Login and password-reset pages
- OAuth authorization and consent pages
- Payment-confirmation pages

### Allow Same-Origin Framing Only

Use this when pages from the same origin must embed the protected content.

```http
Content-Security-Policy: frame-ancestors 'self';
```

`'self'` means the same scheme, hostname, and port as the protected page.

### Allow Specific Trusted Origins

Use this only when you have a genuine integration requirement.

```http
Content-Security-Policy: frame-ancestors 'self' [https://portal.example.com](https://portal.example.com) [https://partner.example](https://partner.example);
```

> [!warning]
> Do not broadly allow third-party origins simply for convenience. Every allowed framing origin can potentially present your page to its users and should be trusted accordingly.

### Important Limitation

`frame-ancestors` must be delivered in an **HTTP response header**. It does not work when placed in a `<meta http-equiv="Content-Security-Policy">` tag. [3]

---

## Legacy Defense: X-Frame-Options

`X-Frame-Options` is the older frame-control response header. It is less flexible than `frame-ancestors`, but it remains useful as a fallback for older browser compatibility. [1][2]

### Deny All Framing

```http
X-Frame-Options: DENY
```

No site, including the same site, can frame the page.

### Allow Same-Origin Framing

```http
X-Frame-Options: SAMEORIGIN
```

Only pages from the same origin can frame the page.

### Avoid ALLOW-FROM

Do not rely on:

```http
X-Frame-Options: ALLOW-FROM [https://trusted.example](https://trusted.example)
```

`ALLOW-FROM` has inconsistent browser support and cannot express multiple trusted origins reliably. Use CSP `frame-ancestors` when specific external origins need permission.

---

## Recommended Header Configurations

### Most Sensitive Pages

```http
Content-Security-Policy: frame-ancestors 'none';
X-Frame-Options: DENY
```

Use this for pages that should never appear inside another site's frame.

### Same-Site Embedding Required

```http
Content-Security-Policy: frame-ancestors 'self';
X-Frame-Options: SAMEORIGIN
```

Use this when your own application legitimately uses iframes.

### Approved Partner Embedding

```http
Content-Security-Policy: frame-ancestors 'self' [https://partner.example](https://partner.example);
```

Because `X-Frame-Options` cannot reliably express this type of allowlist, carefully test the behavior required by your browser-support policy.

> [!tip]
> Send both headers where practical. Modern browsers use the more granular CSP directive, while `X-Frame-Options` can provide coverage for older clients. [1][2]

---

## Server Configuration Examples

### Nginx

Block framing entirely:

```nginx
add_header Content-Security-Policy "frame-ancestors 'none'" always;
add_header X-Frame-Options "DENY" always;
```

Allow same-origin framing:

```nginx
add_header Content-Security-Policy "frame-ancestors 'self'" always;
add_header X-Frame-Options "SAMEORIGIN" always;
```

### Apache HTTP Server

Block framing entirely:

```apache
Header always set Content-Security-Policy "frame-ancestors 'none'"
Header always set X-Frame-Options "DENY"
```

Allow same-origin framing:

```apache
Header always set Content-Security-Policy "frame-ancestors 'self'"
Header always set X-Frame-Options "SAMEORIGIN"
```

### Express.js

```javascript
app.use((req, res, next) => {
  res.setHeader(
    "Content-Security-Policy",
    "frame-ancestors 'none'"
  );

  res.setHeader("X-Frame-Options", "DENY");
  next();
});
```

### Django

```python
# settings.py
X_FRAME_OPTIONS = "DENY"
```

For CSP, configure a trusted CSP middleware or set the HTTP response header at the application or reverse-proxy layer.

### Spring Security

```java
http
  .headers(headers -> headers
    .frameOptions(frame -> frame.deny())
    .contentSecurityPolicy(csp ->
      csp.policyDirectives("frame-ancestors 'none'")
    )
  );
```

> [!note]
> Apply the headers consistently to all HTML responses that contain user-interactive controls, including error pages, legacy routes, and pages generated by separate services. [2]

---

## Why JavaScript Frame Busting Is Not Enough

Older applications sometimes use JavaScript to detect whether they are framed and then force the top-level window to navigate:

```javascript
if (window.top !== window.self) {
  window.top.location = window.self.location;
}
```

This technique is called **frame busting**.

It should not be your primary defense because:

- It runs after the page has begun loading.
- Browser behavior and isolation controls can limit its effectiveness.
- Attackers may bypass or interfere with client-side logic.
- It is harder to implement and verify consistently than server-sent headers.
- It does not provide the clear, browser-enforced policy offered by `frame-ancestors`.

> [!success]
> Prefer server-sent `Content-Security-Policy: frame-ancestors ...`. Treat frame-busting scripts only as optional, defense-in-depth measures for unusual legacy situations.

---

## Testing for Clickjacking

Only test applications you own or are explicitly authorized to test.

### Manual Test

1. Identify a sensitive target page, such as profile settings or an admin action.
2. Create a simple local HTML page with an iframe pointing to the target.
3. Open the test page in a browser.
4. Check whether the target content loads inside the iframe.
5. Inspect response headers in browser developer tools or an HTTP proxy.
6. Verify that sensitive pages return the intended `Content-Security-Policy` and `X-Frame-Options` values.

Example test page:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Authorized frame test</title>
  </head>
  <body>
    <h1>Frame test</h1>

    <iframe
      src="[https://target.example/account](https://target.example/account)"
      width="1000"
      height="700">
    </iframe>
  </body>
</html>
```

If `frame-ancestors 'none'` or `X-Frame-Options: DENY` is correctly applied, the browser should refuse to display the target page in the iframe.

### Header Inspection

```bash
curl -I [https://target.example/account](https://target.example/account)
```

Look for:

```http
Content-Security-Policy: frame-ancestors 'none';
X-Frame-Options: DENY
```

### Test Questions

- Does every sensitive HTML page return anti-framing headers?
- Are headers present on redirects and error responses where appropriate?
- Does the CSP policy accidentally permit untrusted domains?
- Do subdomains have different or missing policies?
- Can a legacy endpoint be framed even if the main application cannot?
- Is same-origin framing genuinely required before using `'self'`?
- Have approved partner origins been reviewed and minimized?

---

## Common Mistakes

### Setting Headers Only on the Homepage

A homepage may be protected while account, checkout, administration, or legacy pages remain frameable.

**Fix:** Configure anti-framing headers globally, then create narrowly reviewed exceptions only where required.

### Using CSP in a Meta Tag

```html
<meta http-equiv="Content-Security-Policy" content="frame-ancestors 'none'">
```

This does not provide `frame-ancestors` protection.

**Fix:** Send the policy as an HTTP response header.

### Allowing Too Many Origins

```http
Content-Security-Policy: frame-ancestors *;
```

This effectively allows any site to embed the page and defeats the purpose of the control.

**Fix:** Use `'none'`, `'self'`, or a small allowlist of trusted HTTPS origins.

### Relying Only on X-Frame-Options

`X-Frame-Options` cannot express a robust multi-origin allowlist and is superseded by CSP `frame-ancestors` in modern browsers. [1][3]

**Fix:** Use CSP `frame-ancestors` as the primary control and include `X-Frame-Options` where legacy support matters.

### Using ALLOW-FROM

`ALLOW-FROM` is not consistently supported.

**Fix:** Use CSP `frame-ancestors https://trusted.example`.

### Forgetting Embedded Product Requirements

Some applications legitimately need framing for dashboards, support portals, integrations, or internal administration tools.

**Fix:** Document the business need, allow only the exact required origins, and retest the embedding flow after policy changes.

---

## Related Notes

- [[Content Security Policy]]
- [[Security Headers]]
- [[CSRF]]
- [[CORS]]
- [[Authentication Vulnerabilities]]
- [[Session Management]]
- [[Security Testing Checklist]]
- [[OWASP Top 10]]

---

