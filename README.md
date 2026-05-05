## Challenge Overview
- **Platform:** ctfio.com (Multiple instances: quartz, tanzanite, barite, hennessy)
- **Difficulty:** Intermediate
- **Type:** Web Application Exploitation - OSINT, Authentication, API Security, IDOR
- **Total Flags:** 8 discovered

---
## Tools Used

**Subfinder** - Fast passive subdomain discovery using CT logs and search engines. Quickly reveals hidden subdomains without generating suspicious traffic.

**ffuf** - Web fuzzer for discovering hidden directories and files. The `-mc` flag filters for relevant status codes (200, 301, 302, 403).

**Burp Suite** - Intercept and analyze HTTP requests. Used for request inspection, Intruder (brute forcing credentials), and Repeater (manual API testing with authentication headers).

**Parrot OS** - Penetration testing distribution with pre-installed security tools.

### nslookup / dig
**Purpose:** DNS queries for discovering records and subdomains
**Use case:** Finding DNS TXT records that may contain flags or sensitive info
--- 
What I Learned:

**OSINT is the foundation** - Subdomain enumeration (Subfinder) revealed the entire attack path. Always start with reconnaissance.

**Weak auth = game over** - No rate limiting on login meant brute forcing credentials was trivial. One weak point cascades.

**Credentials are keys** - Once you got admin:password, it unlocked the next layer (API keys in /env)

**Lateral movement** - Each exploit revealed new attack surface (login → env file → API key → server endpoint → IDOR)

**APIs are often less protected** - The server subdomain was locked down until you found the API key in /env - devs sometimes prioritize UI security but forget API endpoints

**IDOR is deadly** - Numeric IDs with no access checks = you can access any user's data. Authorization must happen at object level, not just endpoint level

**Information disclosure matters** - Flags leaked in error messages, env files, and API responses. Never assume error messages are harmless.

**Tools matter** - ffuf automated discovery that would've taken hours manually. Learn your tools well.

**Defense-in-depth is real** - One vulnerability wasn't enough; the CTF required chaining multiple exploits. Securing one layer would've stopped you.

**Methodology > luck** - You didn't guess; you: enumerated → tested → exploited → repeated. That's professional pentesting.

---
## Key Takeaway

Each vulnerability in this CTF was a stepping stone to the next. Credentials led to API keys, which led to protected endpoints, which revealed IDOR vulnerabilities. This demonstrates the importance of defense-in-depth: securing multiple layers means that even if one is compromised, the attack chain stops.
