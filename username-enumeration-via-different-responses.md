# Lab 1 — Username Enumeration via Different Responses

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Apprentice
**Category:** Authentication
**Status:** Solved ✅

## What's the lab about?

The app has a login page that behaves differently depending on whether the username you entered actually exists or not. That difference in response is enough to enumerate valid usernames, and once you have a valid username, you can brute-force the password the same way.

The lab provides two wordlists: one containing candidate usernames and another containing candidate passwords.

<img width="800" height="300" alt="image" src="https://github.com/user-attachments/assets/e4103e6c-3262-48a0-afbf-7fd86dfdbe2e" />

## What I did

### Step 1 — Intercepted the login request

Opened Burp Suite, turned on intercept, and submitted a dummy login on the application. Captured the `POST /login` request and sent it to **Intruder**.

<img width="800" height="300" alt="Screenshot 2026-06-26 150429" src="https://github.com/user-attachments/assets/d179c437-e7e4-49bc-adad-0543a8734213" />

### Step 2 — Set up the username enumeration attack

In Intruder, selected the **Sniper** attack type. Cleared the default payload positions and marked only the `username` parameter as the payload position.

```text
username=§test§&password=test
```

Loaded the candidate usernames wordlist as the payload and launched the attack.

<img width="800" height="300" alt="Screenshot 2026-06-26 150441" src="https://github.com/user-attachments/assets/5f3f45eb-4301-4eb1-9d20-4d52ccac7d4c" />

### Step 3 — Found the valid username

After the attack completed, sorted the results by **response length**. Most requests had the same response length and returned **"Invalid credentials"**. One username (`user`) produced a different response length and returned **"Incorrect password"**, indicating that the username existed and only the password was incorrect.

<img width="800" height="300" alt="Screenshot 2026-06-26 150524" src="https://github.com/user-attachments/assets/4d6808fa-86d6-48fe-b3db-ede07aa45285" />

### Step 4 — Brute-forced the password

Fixed the username in the request and moved the payload position to the `password` parameter. Loaded the candidate passwords wordlist and launched another Sniper attack.

This time, instead of comparing response lengths, I monitored the **status code**. Incorrect passwords returned **200 OK**, while the correct password produced a **302 Found** response, indicating a successful login.

<img width="800" height="300" alt="Screenshot 2026-06-26 151347" src="https://github.com/user-attachments/assets/0a0fa94f-9e39-40dc-a4b1-816bebf5949e" />

### Step 5 — Logged in

Used the discovered credentials to log in through the browser and successfully accessed the user account.

**Credentials found:**

* Username: `user`
* Password: `123456`

<img width="800" height="300" alt="Screenshot 2026-06-26 151307" src="https://github.com/user-attachments/assets/93344439-7c7a-4da4-8e56-c913b0fba828" />

## Why this works

The application leaks information through its authentication responses. Instead of returning the same generic error for every failed login, it distinguishes between an invalid username and an incorrect password. This allows an attacker to identify valid usernames before attempting to brute-force passwords.

Once a valid username is known, Burp Intruder can automate password testing. A successful login is easy to identify because the server responds with a **302 Found** redirect instead of the normal **200 OK** response.

## Tools used

* Burp Suite Community — Intruder (Sniper attack)
* PortSwigger candidate username wordlist
* PortSwigger candidate password wordlist

## Key takeaway

Authentication responses should be consistent regardless of whether the username or password is incorrect. Small differences in error messages, response lengths, or status codes can leak sensitive information and make username enumeration and password brute-forcing significantly easier.
