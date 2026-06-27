# Basic Clickjacking with CSRF Token Protection

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Apprentice
**Category:** Clickjacking
**Status:** Solved ✅

## What's the lab about?

The app has a "Delete Account" button on the account page that's protected by a CSRF token. The goal is to trick a victim into clicking that button without them realising it — using a clickjacking attack.

Clickjacking works by embedding the target page in a transparent `<iframe>` on top of a decoy page you control. The victim sees your fake page and clicks what looks like a harmless button, but they're actually clicking the real button on the invisible iframe underneath.

Credentials provided: `wiener:peter`

<img width="1137" height="540" alt="Screenshot 2026-06-27 214007" src="https://github.com/user-attachments/assets/35117c28-554c-403a-8d4e-e6ba37baf9be" />


## What I did

### Step 1 — Logged in and explored the account page

Logged in with `wiener:peter`. On the My Account page, there's a **Delete account** button at the bottom. That's the target — the button the victim needs to be tricked into clicking.

<img width="1577" height="700" alt="Screenshot 2026-06-27 213655" src="https://github.com/user-attachments/assets/11cea251-568d-4539-bba1-4cc4c006ab30" />

---

<img width="1497" height="723" alt="Screenshot 2026-06-27 213901" src="https://github.com/user-attachments/assets/ad8f9e25-3965-4318-bf61-1b9dfeeab9c5" />

### Step 2 — Went to the exploit server

The lab provides an exploit server where you can host your own HTML and deliver it to a simulated victim. Opened the exploit server — it gives you a URL, a body editor, and a "Deliver exploit to victim" button.

<img width="1192" height="807" alt="Screenshot 2026-06-27 213750" src="https://github.com/user-attachments/assets/af9b2c2e-cbb7-4a2b-aad0-e63e90d2a44f" />

### Step 3 — Crafted the clickjacking payload

The idea is to:
1. Load the real account page inside an `<iframe>` — but make it nearly invisible (`opacity: 0.1`)
2. Place a fake "Click me" button precisely on top of the real "Delete account" button using absolute positioning
3. The victim sees "Click me", but clicks "Delete account" on the iframe underneath

Here's the payload I used:

```html
<style>
    iframe {
        position: relative;
        width: 500px;
        height: 700px;
        opacity: 0.1;
        z-index: 2;
    }
    div {
        position: absolute;
        top: 300px;
        left: 60px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://<LAB-ID>.web-security-academy.net/my-account"></iframe>
```

Key things here:
- `opacity: 0.1` — makes the iframe nearly invisible. Set higher during testing so you can see the alignment, then lower it before delivering
- `z-index: 2` on the iframe — puts it above the div in the stacking order, so the iframe captures the click
- `position: absolute` on the div with `top: 300px; left: 60px` — positions the fake button to line up with the Delete account button in the iframe
- The CSRF token doesn't help here — because the victim is the one clicking the real button while authenticated. The browser sends their cookies automatically with the request

<img width="1067" height="397" alt="Screenshot 2026-06-27 213759" src="https://github.com/user-attachments/assets/b4c6317d-ab94-4114-b0e4-4d3201564917" />


### Step 4 — Tested alignment visually

Before delivering, I previewed the exploit in the browser with `opacity: 0.1` so I could see both layers. Adjusted `top` and `left` values until "Click me" was sitting directly over "Delete account."

<img width="962" height="652" alt="Screenshot 2026-06-27 213923" src="https://github.com/user-attachments/assets/6274502b-4473-4d63-9478-27275b8212b4" />

### Step 5 — Delivered to victim

Once alignment looked right, stored the exploit and clicked **Deliver exploit to victim**. The simulated victim clicked "Click me," which triggered the Delete account button on the real page underneath.

Lab solved.

<img width="1391" height="751" alt="Screenshot 2026-06-27 213937" src="https://github.com/user-attachments/assets/43a99667-c594-462f-b47f-a6c646483bb5" />

## Why this works

CSRF tokens protect against cross-site request forgery — where an attacker crafts a forged request and tricks the browser into sending it. But clickjacking is different. The victim is clicking a real button on the real page while logged in. There's no forged request — the browser sends the victim's cookies legitimately because they're the one interacting with the page.

So CSRF tokens do nothing here. The protection against clickjacking is the `X-Frame-Options` header (or `Content-Security-Policy: frame-ancestors`) — if the app set either of these, the browser would refuse to load the page inside an iframe in the first place. This app doesn't set them, which is why the attack works.


## Tools used

- PortSwigger Exploit Server
- HTML/CSS (iframe overlay technique)
- Browser (for testing alignment)


## Key takeaway

CSRF protection and clickjacking protection are two separate things. A CSRF token doesn't stop clickjacking because the victim is the one making the real click — the attacker just controls what they think they're clicking. The fix is `X-Frame-Options: DENY` or `CSP: frame-ancestors 'none'` to prevent the page from being embedded in iframes at all.
