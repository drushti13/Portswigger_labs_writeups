# Information Disclosure in Error Messages

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Apprentice
**Category:** Information Disclosure
**Status:** Solved ✅


## What's the lab about?

The app has verbose error messages that reveal more than they should. Specifically, when something goes wrong server-side, the error response leaks the name and version of the third-party framework the app is built on. The goal is to trigger one of those errors and pull out the framework version.

<img width="1130" height="382" alt="Screenshot 2026-06-29 102110" src="https://github.com/user-attachments/assets/da9f6f41-f3e1-48ad-abe7-25872e4856bd" />


## What I did

### Step 1 — Explored the app and intercepted requests in Burp

Opened the app in the Burp browser with intercept running. The shop has individual product pages, each loaded via a `productId` parameter in the URL:

Sent one of these requests to **Repeater**.

<img width="1917" height="212" alt="Screenshot 2026-06-29 102016" src="https://github.com/user-attachments/assets/0b9ae684-5f68-4309-bd13-1dd39073f5d1" />


### Step 2 — Replaced the productId with a string

In Repeater, changed the `productId` value from an integer to the string `hi`:

The app expects an integer here — passing a string causes it to throw an unhandled exception server-side.

<img width="1491" height="782" alt="Screenshot 2026-06-29 102003" src="https://github.com/user-attachments/assets/256632fe-c62e-4e0b-94e6-e2893c6bfb25" />

### Step 3 — Got a 500 error with the framework version

The response came back as a `500 Internal Server Error`. Instead of a generic error page, the app dumped a full Java stack trace in the response. At the bottom of the stack trace:

```
Apache Struts 2 2.3.31
```

The framework name and version, right there in the error output.

### Step 4 — Submitted the version

Clicked **Submit solution**, entered `2 2.3.31`, and the lab marked as solved.

<img width="1901" height="1011" alt="Screenshot 2026-06-29 101938" src="https://github.com/user-attachments/assets/63a0e1d1-68e3-4321-959b-4e9727d87a5e" />

---

<img width="1816" height="903" alt="Screenshot 2026-06-29 102051" src="https://github.com/user-attachments/assets/02900fb7-1ea5-40db-8636-3ce470176758" />

## Why this works

Apache Struts 2.3.31 is a known vulnerable version — it's one of the versions affected by CVE-2017-5638, the same vuln that led to the Equifax breach. So this isn't just a lab exercise; leaking a framework version in an error message gives an attacker exactly the information they need to look up public exploits.

Verbose error messages are a developer convenience that becomes a security problem in production. Stack traces, framework versions, database error details — none of that should ever reach the client. The fix is proper error handling: log the full error server-side, return a generic message to the user.

## Tools used

- Burp Suite — HTTP history, Repeater

## Key takeaway

Never expose stack traces or framework details in production error responses. A 500 error should say "something went wrong" and nothing else. Version information in error output is a direct path to known CVEs — attackers don't need to fingerprint the tech stack if the app tells them itself.
