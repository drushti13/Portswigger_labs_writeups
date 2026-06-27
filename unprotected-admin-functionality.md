# Unprotected Admin Functionality

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Apprentice
**Category:** Access Control
**Status:** Solved ✅

---

## What's the lab about?

The app has an admin panel that isn't protected — no authentication check, no role verification, nothing. If you find the URL, you can just access it. The goal is to find that panel and delete the user `carlos`.
<img width="1136" height="357" alt="Screenshot 2026-06-26 154628" src="https://github.com/user-attachments/assets/017fe161-3884-4269-b9ee-a29e3cb3313e" />

---

## What I did

### Step 1 — Tried guessing common admin paths

Opened the app, saw a login page. Went to Burp Repeater and started appending common admin paths to the base URL:

- `/admin` → `404 Not Found`
- `/administrator` → `404 Not Found`

Neither worked. So guessing wasn't going to cut it here.

<img width="1298" height="468" alt="Screenshot 2026-06-26 153644" src="https://github.com/user-attachments/assets/00ca3e36-45d7-46cc-8725-68d89fe797a7" />

All requests returned a **404 Not Found** response, indicating that the application wasn't using standard admin URLs.

### Step 2 — Checked robots.txt

When guessing fails, `robots.txt` is the next obvious place to look. It's a file that tells search engine crawlers which paths to avoid indexing — but it also ends up revealing paths the developer didn't want publicly visible.

Navigated to `/robots.txt`. Got a `200 OK` response. The file contained:
```
User-agent: *
Disallow: /administrator-panel
```
There it is.

<img width="1327" height="456" alt="Screenshot 2026-06-26 153748" src="https://github.com/user-attachments/assets/aedbf670-cf9c-42c4-a2c0-4c2f2b163aa7" />

<img width="1047" height="267" alt="Screenshot 2026-06-26 153805" src="https://github.com/user-attachments/assets/be130375-5b20-476d-80ca-7feca3891b4b" />

### Step 3 — Accessed the admin panel directly

Went to `/administrator-panel` in the browser. No login prompt, no access check — just loaded straight into the admin panel showing all users.

Listed users:
- `wiener`
- `carlos`

<img width="1090" height="481" alt="Screenshot 2026-06-26 153820" src="https://github.com/user-attachments/assets/8331859e-20f6-46db-9ad9-927ed6c17496" />

### Step 4 — Deleted carlos

Clicked **Delete** next to `carlos`. Done.

<img width="1526" height="552" alt="Screenshot 2026-06-26 153847" src="https://github.com/user-attachments/assets/97812a90-cb18-4e39-a864-26c10154621e" />

---

## Why this works

The developer made the URL harder to guess — but the app still has to expose it somewhere for it to function. Maybe it's embedded in JavaScript on the home page, maybe a request to the page source reveals it. Either way, Burp's passive crawling catches it without you having to do anything special.
An unpredictable URL buys you a little obscurity, but it's still just obscurity. The real fix is enforcing access control server-side — checking whether the logged-in user actually has admin privileges before serving the page.

---

## Tools used

- Burp Suite — Target / Site map (passive crawling)
- Browser

---

## Key takeaway

"Hard to guess" is not the same as "protected." If the app references the URL anywhere — in source code, in a script, in a redirect — Burp will find it during normal browsing. Unpredictable paths are not a security control.
