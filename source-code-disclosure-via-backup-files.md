# Source Code Disclosure via Backup Files

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Apprentice
**Category:** Information Disclosure
**Status:** Solved ✅

## What's the lab about?

The app has a backup directory that's publicly accessible. Inside it is a `.bak` file — a backup of a Java source file. That source file has a database password hard-coded directly in it. The goal is to find the backup, read the source code, and pull out the password.

<img width="1123" height="381" alt="Screenshot 2026-06-29 152022" src="https://github.com/user-attachments/assets/f8377951-09ee-4286-8d1a-239a132fa896" />

## What I did

### Step 1 — Checked robots.txt

First thing I checked was `/robots.txt`. Same reasoning as previous labs — it's a public file that often lists paths the developer wanted to hide from search engines, which makes it a useful hint for finding sensitive directories.

```
User-agent: *
Disallow: /backup
```

Found it on the first try. The app is explicitly telling crawlers not to index `/backup`.

<img width="1203" height="253" alt="Screenshot 2026-06-29 151841" src="https://github.com/user-attachments/assets/c0f0be82-5eb7-451d-b9a4-a7e4045ca17e" />

### Step 2 — Navigated to /backup

Went to `/backup` in the browser. Directory listing was enabled — it showed the contents of the folder:

```
Index of /backup

Name                        Size
ProductTemplate.java.bak    1647B
```

A `.bak` file of a Java class. No auth, no restriction, just sitting there.

<img width="1023" height="277" alt="Screenshot 2026-06-29 151903" src="https://github.com/user-attachments/assets/e81b5323-531f-4105-9a38-a80515183d4d" />

### Step 3 — Opened the backup file

Clicked the link to open `ProductTemplate.java.bak`. Full Java source code rendered in the browser.

The class is called `ProductTemplate`, it implements `Serializable`, and in the `readObject` method it builds a database connection. The `ConnectionBuilder` call has all the connection details hard-coded as string literals:

```java
ConnectionBuilder connectionBuilder = ConnectionBuilder.from(
    "org.postgresql.Driver",
    "postgresql",
    "localhost",
    5432,
    "postgres",
    "postgres",
    "5lwoxci6blre2ha669kuo5j1fmzztnxu"
).withAutoCommit();
```

The last argument is the database password: `5lwoxci6blre2ha669kuo5j1fmzztnxu`.

<img width="1205" height="1022" alt="Screenshot 2026-06-29 151939" src="https://github.com/user-attachments/assets/d2eefccd-fb3f-4e96-954a-f69ee1b48761" />

### Step 4 — Submitted the password

Clicked **Submit solution**, pasted the password, and the lab solved.

<img width="1917" height="1031" alt="Screenshot 2026-06-29 151954" src="https://github.com/user-attachments/assets/259c7d6d-75aa-466b-bf8f-e020be91e912" />

---

<img width="1881" height="941" alt="Screenshot 2026-06-29 152009" src="https://github.com/user-attachments/assets/fb9e23c8-95d1-4319-8fa6-980b18bdc5f2" />

## Why this works

Two separate problems stacked on top of each other here:

First, the backup file is in a publicly accessible directory with no access control. Backup files (`.bak`, `.old`, `.zip`, `.tar`) are commonly left behind during deployments and are a well-known recon target — directory brute-forcing tools like Gobuster specifically look for these extensions.

Second, the source code has a database password hard-coded directly in it. Even if this file was in a private location, hard-coding credentials in source code is a bad practice — it ends up in version control, in backups, in logs, anywhere the code travels. Credentials should always be injected at runtime via environment variables or a secrets manager, never baked into the code itself.


## Tools used

- Browser (direct navigation)
- `/robots.txt` for path discovery


## Key takeaway

Two lessons here: don't leave backup files in web-accessible directories, and never hard-code credentials in source code. `robots.txt` pointed straight to the backup directory, and once there, the password was just sitting in plain text in the source code. Either mistake alone is bad — both together made it trivial.
