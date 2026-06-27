# Unprotected Admin Functionality with Unpredictable URL

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Apprentice
**Category:** Access Control
**Status:** Solved ✅

## What's the lab about?

Same concept as the previous lab — an admin panel with no access control — but this time the URL isn't something you can guess. It's a randomly generated path. The trick is that the location is disclosed somewhere inside the application itself.

<img width="792" height="395" alt="image" src="https://github.com/user-attachments/assets/61bbfd07-0420-4b18-912d-fc5b16d6a86f" />

## What I did

### Step 1 — Let Burp map the application

Opened Burp Suite and turned on intercept. Started browsing the app — visited the home page, clicked through product pages, and generally interacted with it while Burp was running in the background.

The **Target → Site map** in Burp automatically records every URL the application touches during normal browsing. This is how I mapped out what directories and endpoints existed without manually guessing anything.

<img width="1911" height="608" alt="Screenshot 2026-06-26 155129" src="https://github.com/user-attachments/assets/a3e4e2bb-ce96-4101-819c-9a6db8db72d6" />

### Step 2 — Found the admin URL in the site map

In the site map, one URL stood out — something like `/admin-6byh5` (a random-looking path). Burp picked it up because the application itself referenced it somewhere in the page source or in one of the requests made while browsing.

That's the thing about "unpredictable" URLs — the application still has to load or reference them somewhere. Burp simply captures them during passive browsing.

### Step 3 — Navigated to the admin panel

Opened that URL directly in the browser. The admin panel loaded without any authentication or authorization checks and displayed both users.

<img width="1372" height="496" alt="Screenshot 2026-06-26 155109" src="https://github.com/user-attachments/assets/fdd60e00-dbd4-47a2-b38e-1962aebab4f8" />

### Step 4 — Deleted carlos

Clicked **Delete** next to `carlos`. Lab solved.

<img width="1666" height="506" alt="Screenshot 2026-06-26 155152" src="https://github.com/user-attachments/assets/70108c05-a494-4f0d-9ded-419225b8beaf" />

## Why this works

The developer made the URL harder to guess, but the application still had to expose it somewhere for it to function. Whether it's referenced in the page source, JavaScript, or another request, Burp's passive crawling can discover it automatically.

An unpredictable URL provides obscurity, not security. The real protection should come from server-side access control that verifies whether the logged-in user has permission to access the admin panel.

## Tools used

* Burp Suite — Target / Site map (passive crawling)
* Browser

## Key takeaway

"Hard to guess" is not the same as "protected." If the application references the URL anywhere, Burp can discover it during normal browsing. Unpredictable URLs should never be relied upon as a security control.
