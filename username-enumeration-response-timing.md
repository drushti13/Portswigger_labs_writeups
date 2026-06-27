# Username Enumeration via Response Timing

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Practitioner
**Category:** Authentication
**Status:** Solved ✅

## What's the lab about?

This lab demonstrates username enumeration through **response timing** instead of error messages or response lengths. When a valid username is supplied, the server performs password verification, which takes longer than immediately rejecting an invalid username. That timing difference can be used to identify valid accounts.

Credentials provided: `wiener:peter`

<img width="1097" height="467" alt="image" src="https://github.com/user-attachments/assets/97005a12-48ad-4935-ad16-e9925efcd6e2" />

## What I did

### Step 1 — Logged in and intercepted the login request

Opened the application, enabled Burp Intercept, and logged in using the provided credentials `wiener:peter`. Captured the `POST /login` request.

### Step 2 — Tested response timing manually in Repeater

Before automating the attack, I sent a few requests manually using **Repeater**.

* Sent a request with a **valid username** and an incorrect password. The response took noticeably longer.
* Sent another request with an **invalid username**. The response returned much faster.

This confirmed that the application leaked information through response timing.

### Step 3 — Hit the rate limit

After several login attempts, the application temporarily blocked further requests due to IP-based rate limiting.

### Step 4 — Bypassed rate limiting with `X-Forwarded-For`

The application trusted the `X-Forwarded-For` header to determine the client's IP address. By changing this value for every request, I made the server treat each request as if it originated from a different IP address, effectively bypassing the rate limit.

### Step 5 — Configured a Pitchfork attack in Intruder

Selected **Pitchfork** as the attack type because two payloads needed to advance together:

* **Payload Set 1:** Sequential values for the `X-Forwarded-For` header (`1`, `2`, `3`, ...)
* **Payload Set 2:** Candidate usernames

I also supplied a very long password to increase the time spent performing password verification, making the timing difference easier to observe.

<img width="1462" height="932" alt="Screenshot 2026-06-27 130215" src="https://github.com/user-attachments/assets/e449c5d8-5c2f-4304-894d-5e905c4eeaa0" />

### Step 6 — Identified the valid username

After running the attack, I sorted the results by **Response received** time. One username (`ag`) consistently produced a much longer response time than the others, indicating that it was valid.

![Intruder results — username "ag" shows longer response time](screenshots/username-enumeration-response-timing/timing-results.png)

### Step 7 — Brute-forced the password

Fixed the username to `ag` and configured another Pitchfork attack with:

* Sequential `X-Forwarded-For` values
* Candidate password wordlist

This time I monitored the **302 Found** responses. The password `football` returned a **302**, confirming a successful login.

<img width="1865" height="676" alt="Screenshot 2026-06-27 130307" src="https://github.com/user-attachments/assets/2567effa-e905-4486-9525-468e04f5c67f" />

---

<img width="1466" height="915" alt="Screenshot 2026-06-27 130229" src="https://github.com/user-attachments/assets/0a614db3-cf2f-4b1f-adc0-b4a6b2873850" />

### Step 8 — Logged in

Used the discovered credentials (`ag:football`) in the browser and successfully accessed the account page.

<img width="1642" height="636" alt="Screenshot 2026-06-27 130155" src="https://github.com/user-attachments/assets/11f27af9-a593-41f2-993a-d16bca1ee86f" />

## Why this works

The application performs additional work when a valid username is supplied because it attempts password verification. Invalid usernames are rejected immediately, resulting in faster responses. Measuring this timing difference allows an attacker to enumerate valid usernames even when error messages and response lengths are identical.

The rate limit bypass succeeds because the application trusts the client-controlled `X-Forwarded-For` header when identifying the client's IP address. Since this header can be modified by an attacker, it should never be relied upon for security decisions.

## Tools used

* Burp Suite — Repeater
* Burp Suite — Intruder (Pitchfork attack)
* `X-Forwarded-For` header
* PortSwigger candidate username wordlist
* PortSwigger candidate password wordlist

## Key takeaway

Response time can become an information leak if applications process valid and invalid usernames differently. Authentication should perform uniform work regardless of whether an account exists, and security controls such as rate limiting should never rely on client-supplied headers like `X-Forwarded-For`.
