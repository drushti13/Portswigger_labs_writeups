# Information Disclosure on Debug Page

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Apprentice
**Category:** Information Disclosure
**Status:** Solved ✅

## What's the lab about?

The app has a debug page that was left accessible — it dumps environment variables, config values, and system information. One of those environment variables is `SECRET_KEY`. The goal is to find the debug page and retrieve it.

<img width="1138" height="372" alt="Screenshot 2026-06-29 144908" src="https://github.com/user-attachments/assets/5f463daa-6121-4380-bdeb-1d8e0b90b006" />

## What I did

### Step 1 — Opened the app in Burp browser and started browsing

Loaded the app in the Burp browser with intercept on. Navigated around — home page, product pages — just general browsing to let Burp passively map what's there.

### Step 2 — Checked the Burp Site map

Went to **Target → Site map** in Burp. It had captured all the URLs the app referenced or returned during browsing. One URL stood out immediately:

<img width="1917" height="727" alt="Screenshot 2026-06-29 144852" src="https://github.com/user-attachments/assets/3a2c0c45-fa96-4e2a-ba61-427a9073226c" />

That's a `phpinfo()` page — a PHP debug output that dumps everything about the server environment. It appeared in the site map alongside the regular product pages, which means it was referenced somewhere in the app's responses.

### Step 3 — Navigated to the phpinfo page

Opened `/cgi-bin/phpinfo.php` directly in the browser. Loaded without any auth check — full debug page, visible to anyone who knows the path.

The page dumps a massive amount of data: PHP version, server config, compiled modules, and importantly — the **Environment** section, which lists all environment variables the server process has access to.

<img width="1590" height="1027" alt="Screenshot 2026-06-29 144831" src="https://github.com/user-attachments/assets/70f90429-b14b-40cd-8e8d-0db4ec971368" />

### Step 4 — Found the SECRET_KEY

Used Ctrl+F and searched for `envi` to jump to the Environment section. Right there in the variable table:

| Variable | Value |
|----------|-------|
| SECRET_KEY | crj6q2kar8ibpupo6bgnl62bo4cm3atd |

Along with other interesting things like `USER: carlos`, `HOME: /home/carlos`, and the remote host IP.

<img width="1910" height="1021" alt="Screenshot 2026-06-29 144800" src="https://github.com/user-attachments/assets/74febadf-bead-46a1-992a-a6c5ac21a9d6" />

### Step 5 — Submitted the key

Clicked **Submit solution**, pasted in the `SECRET_KEY` value, and the lab solved.

<img width="1795" height="947" alt="Screenshot 2026-06-29 144819" src="https://github.com/user-attachments/assets/633b9c03-d6aa-4ebc-ac8c-ae547b6ee558" />


## Why this works

`phpinfo()` pages are meant for local development — they show the full server configuration so devs can debug their setup. Leaving one accessible on a production server is a serious mistake. It hands an attacker the server software versions, enabled modules, environment variables (which often contain API keys, database credentials, secrets), and internal file paths.

The `SECRET_KEY` here would typically be used to sign session tokens or encrypt data. Having it means you can forge tokens, decrypt data, or impersonate users depending on how it's used.

## Tools used

- Burp Suite — Target / Site map (passive crawling)
- Browser

## Key takeaway

Debug pages and diagnostic endpoints need to be removed or access-controlled before going to production. `phpinfo()` is the classic example — it looks harmless but exposes a complete picture of the server environment. Burp's site map catches these passively just from normal browsing, no active scanning needed.
