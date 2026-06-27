# Lab 1 — Username Enumeration via Different Responses

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Apprentice
**Category:** Authentication
**Status:** Solved ✅

---

## What's the lab about?

The app has a login page that behaves differently depending on whether the username you entered actually exists or not. That difference in response is enough to enumerate valid usernames — and once you have a valid username, you can brute-force the password the same way.

The lab gives you two wordlists: one for candidate usernames, one for candidate passwords.

---

## What I did

### Step 1 — Intercepted the login request

Opened Burp Suite, turned on intercept, and submitted a dummy login on the app. Captured the POST `/login` request and sent it to **Intruder**.

### Step 2 — Set up the username enumeration attack

In Intruder, set the attack type to **Sniper**. Cleared all default payload positions and manually set the payload marker on just the `username` field in the request body.

```
username=§test§&password=test
```

Loaded the candidate usernames wordlist as the payload. Launched the attack.

### Step 3 — Found the valid username

Once the attack finished, sorted results by **response length**. Most requests came back with the same length — those are the "Invalid credentials" responses. One username (`user`) had a noticeably different response length and returned **"Incorrect password"** instead of "Invalid credentials."

That's the difference the lab is built on — the app is leaking whether the username exists through its error message.

![Intruder username enumeration results — response length difference visible for payload "user"](screenshots/lab1/username-enumeration.png)

### Step 4 — Brute-forced the password

Took the valid username, fixed it in the request, and now set the payload position on the `password` field. Loaded the candidate passwords wordlist. Ran the Sniper attack again.

This time, instead of looking at response length, watched for the **status code**. All incorrect passwords return `200 OK`. The correct one triggered a `302 Found` — meaning the app redirected, i.e., successful login.

![Intruder password brute-force — request 9 returns 302, rest return 200](screenshots/lab1/password-bruteforce.png)

### Step 5 — Logged in

Used the valid username and password in the browser. Logged in successfully and landed on the account page.

**Credentials found:**
- Username: `user`
- Password: identified via the 302 redirect in Intruder

![Lab solved — account page showing username and email](screenshots/lab1/lab-solved.png)

---

## Why this works

The app is giving away too much information through its error messages. Most secure apps return the exact same message whether the username doesn't exist or the password is wrong — something like "Invalid username or password." That way you have no way to tell which one failed.

Here, the two different messages ("Invalid credentials" vs "Incorrect password") let you enumerate usernames one by one. Combined with a wordlist, Intruder automates the whole thing in seconds.

The 302 on successful password brute-force is also a classic tell — the moment the server authenticates you, it redirects. Sorting by status code in Intruder makes it instantly obvious which payload worked.

---

## Tools used

- Burp Suite Community — Intruder (Sniper attack)
- PortSwigger candidate username + password wordlists

---

## Key takeaway

Error messages are a security control. Vague, identical messages across all failure cases prevent enumeration. The moment an app tells you *which part* of the credentials was wrong, it's giving an attacker exactly what they need.
