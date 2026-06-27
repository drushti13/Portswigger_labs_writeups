# Username Enumeration via Subtly Different Responses

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Practitioner
**Category:** Authentication
**Status:** Solved ✅

## What's the lab about?

This is a step up from the basic username enumeration lab. The application still leaks whether a username is valid, but the difference is much more subtle. Instead of returning completely different error messages, it only changes a tiny part of the same message. Since every username in the wordlist has a different length, sorting responses by length is no longer reliable.

<img width="1070" height="482" alt="Screenshot 2026-06-26 163551" src="https://github.com/user-attachments/assets/9fdfe30f-5b41-4099-9ac4-44b87615fb94" />

## What I did

### Step 1 — Intercepted the login request and sent it to Intruder

Captured the `POST /login` request in Burp Suite and sent it to **Intruder**. Selected the **Sniper** attack type and marked only the `username` parameter as the payload position.

Loaded the candidate usernames wordlist as the payload.

### Step 2 — Configured Grep-Extract

Unlike the previous lab, response length couldn't be used because every username in the wordlist has a different length. Instead, I configured **Grep-Extract** in Intruder to extract the error message from each response.

I highlighted the text:

```text
Invalid username or password
```

Burp then displayed the extracted error message as a separate column for every response, making it much easier to compare them.

### Step 3 — Identified the valid username

Most responses contained:

```text
Invalid username or password.
```

One username (`academico`) returned:

```text
Invalid username or password 
```

The message looked almost identical, but instead of ending with a period, it contained a trailing space. That tiny formatting difference indicated that the username existed.

<img width="1490" height="322" alt="Screenshot 2026-06-26 162854" src="https://github.com/user-attachments/assets/75740d96-e302-4035-9e52-e9e016e989bb" />

### Step 4 — Brute-forced the password

Fixed `academico` as the username, moved the payload position to the `password` parameter, and loaded the candidate passwords wordlist.

This time I monitored the **status code**. Incorrect passwords returned **200 OK**, while the correct password (`qwertyuiop`) returned **302 Found**, indicating a successful login.

<img width="1517" height="700" alt="Screenshot 2026-06-26 163022" src="https://github.com/user-attachments/assets/d131d81e-8ba2-47be-8dcb-9790c4b8af41" />

### Step 5 — Logged in

Used the discovered credentials to log in through the browser. My first attempt failed because I had mistyped the username. After correcting it, I successfully accessed the account page.

<img width="993" height="512" alt="Screenshot 2026-06-26 163128" src="https://github.com/user-attachments/assets/473e48af-ffed-410f-8651-7e0f1a700754" />

---

<img width="1642" height="747" alt="Screenshot 2026-06-26 163213" src="https://github.com/user-attachments/assets/1214841f-6f15-4f7f-be4c-a1ba8c7606e8" />

## Why this works

The application attempts to return the same error message whether the username exists or not, but a small inconsistency remains—a trailing space instead of a period. Although the difference is almost invisible to a user, Burp's Grep-Extract feature makes it easy to identify.

This demonstrates that even tiny formatting differences can leak sensitive information if responses are compared systematically.

## Tools used

* Burp Suite Community — Intruder (Sniper attack)
* Burp Suite — Grep-Extract
* PortSwigger candidate username wordlist
* PortSwigger candidate password wordlist

## Key takeaway

Security isn't just about returning the same error message—it must return the **exact same response**. Even differences in punctuation or whitespace can allow attackers to enumerate valid usernames when combined with automated tools like Burp Intruder.
