
# Cross-Site Scripting (XSS)

Cross-Site Scripting (XSS) is a vulnerability where an attacker injects
malicious client-side code, usually JavaScript, into a web application.

The vulnerability occurs when the application places untrusted user input
into a web page without correct output encoding or sanitization. The victim's
browser then interprets the attacker-controlled input as code instead of text.

Because the script runs under the trusted website's origin, it can read page
data, modify the UI, capture user input, make requests as the victim, or
perform actions using the victim's active session.


## Stored XSS scenario


A website allows users to submit comments.

An attacker submits a comment containing malicious HTML or JavaScript:

```html
<script>alert("XSS")</script>
```

If the server stores this comment in the database and the frontend renders it
as HTML without escaping or sanitizing it, the script executes whenever another
user opens that comment page.

This is called Stored XSS because the malicious payload is stored in the
application's database and affects later visitors.

****
<span style="font-size:20px; font-weight: bold">Token Stealing</span>
If the application stores a session token in a JavaScript-accessible location,
such as `localStorage`, injected JavaScript may be able to read and exfiltrate it.

Cookies with the `HttpOnly` flag cannot be read directly through JavaScript.
However, HttpOnly does not remove the XSS vulnerability: malicious code may
still read sensitive page data, capture keystrokes, modify the page, and send
authenticated requests as the victim.