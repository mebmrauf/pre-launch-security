---
name: pre-launch-security
description: Run a 31-point security pass on a whole web app before launch, fix what is safe to fix, prove the fixes hold, and report findings. Use when the user says they are about to launch, ship, or go live and asks if the app is safe, secure, or ready, when they want a security checklist or hardening pass, or when they worry about hackers getting into a vibe-coded app, even if they do not say "security review". Not for reviewing one diff or PR.
---

# Pre-launch security pass

Go through all 31 checks on the whole app, fix the problems, prove the fixes hold, and give a report.

## How to work

1. Learn the app first: stack, hosting, auth, database, payments, uploads. Read the framework's own docs for its security defaults, so you do not rebuild what it already gives you.
2. Do every check. Back each result with something you read or ran, with `file:line` where you can. "Looks fine" is not a result.
3. Mark a check N/A only when the feature does not exist, and say why.
4. Fix in the code when the fix is small and safe. Match the code around it. Use the framework's feature before you add a package.
5. Prove each fix (#31) and only then call it fixed.

## What counts as a finding

Report a problem only if you can describe the attack: who does what, and what they get. Rate your confidence as High, Medium, or Low. If you cannot describe a real path to harm, leave it out of the open items and put it under "Not checked" or as a Low note. A short, true report gets acted on. A long, noisy one gets ignored.

## Ask the user first

These are hard to undo or reach outside the repo, so list them and wait for a yes:

- Revoking or rotating a key (#10). The user usually has to do this in the provider's dashboard.
- Rewriting Git history.
- Database migrations, and access rule changes on live data.
- Major dependency upgrades.
- Changes to production settings or environment variables.

Run tests against local or staging, with throwaway data. Do not send attack-style requests to production unless the user owns it and says yes. If a tool blocks a risky test on production, that block is doing its job.

Never show a secret's value in chat or in the report, not even the first few characters. Give the file, the line, and the key's type. If a command needs a secret, refer to the variable by name.

## The 31 checks

**Accounts and access**

1. **Authentication.** Use a proven auth library or provider, not a custom one. Every private page and API route checks the session on the server. Hiding a page in the UI does not protect it.
2. **Passwords.** Hash with argon2, bcrypt, or scrypt. Never log them. A wrong email and a wrong password must look the same to the caller: same message, same status, and about the same response time. Run a dummy hash when the user does not exist.
3. **Password resets.** Short expiry, single use, random token, stored hashed. Old and used links must fail. The response must not reveal whether the email has an account.
4. **Session cookies.** `HttpOnly`, `Secure`, and `SameSite=Lax` or stricter, with a sensible expiry. New session on login, destroyed on logout.
5. **Admin routes and roles.** Check the role on the server for each route and API call, from one shared check. If a role is stored in a token or session, a role change must also update or end that token, or the old power lives on. Sign in as a normal user, request `/admin`, and confirm it is denied.
6. **Ownership of data.** This is the most common serious hole. Every route with an ID must confirm the record belongs to the caller, in the data query. Sign in as user A and request user B's record. Prefer `404` to `403`, so outsiders cannot learn which IDs exist.

**Secrets**

7. **Keys stay off the browser.** Browser code calls your server, and your server calls the third party. Search the client bundle and public env vars, like `NEXT_PUBLIC_*` and `VITE_*`.
8. **Environment variables.** `.env` is in `.gitignore`, production uses different secrets from dev, and the app fails fast if one is missing. No real values in `.env.example`.
9. **Git history.** Search all commits, not only the current files. Use `gitleaks git --redact --no-banner` if installed, or else `git log -p --all` with patterns for key formats, and show only file and commit. A range scan with a wrong base ref can report clean, so check the base first.
10. **Leaked keys.** Any secret found in #7, #8, #9, or #11 counts as leaked, even if deleted now. Rotate first, clean up second: kill the key, then think about rewriting history. Give the user the list of what to rotate and where.
11. **Exposed files.** `.env*`, backups, `db-dump.sql`, `.git/config`, and source maps must not be public. Look in the public folder, and request the paths on a running build to confirm a `404`.

**Input**

12. **Server-side validation.** Browser checks are only for UX. The server enforces the rules. Example: `quantity: -3` is rejected when the rule is 1 to 10. Say what is allowed, and reject the rest. Do not try to list what is bad.
13. **Mass assignment.** `update(req.body)` lets a user set `role`, `isAdmin`, or `tenantId`. Use an explicit list of fields a user may change.
14. **Injection.** Parameterized SQL, including raw queries inside an ORM. Commands run with an argument list, never a shell string with user input. For NoSQL, check input types so `{"$ne": null}` cannot pass as a value.
15. **XSS.** Encode on output, so `<script>` shows as text and does not run. Search for `dangerouslySetInnerHTML`, `innerHTML`, `v-html`, and user-supplied URLs in links.
16. **File uploads.** Check the real file type on the server and set a max size. Generate the stored name, and do not serve uploads as runnable code from your main origin.
17. **User-supplied URLs.** If the server fetches a URL the user gave (webhooks, image proxy, link preview), allow only known hosts and HTTPS, block private and metadata addresses like `169.254.169.254`, and refuse redirects. Redirects after login must only go to your own paths.

**Browser and traffic**

18. **HTTPS.** Redirect HTTP to HTTPS everywhere, including API routes and webhooks. Outbound calls keep TLS checks on, and no `verify=False` or `rejectUnauthorized: false`.
19. **CORS.** Allow a list of known origins. Never `*` with credentials, and never echo back the request's origin.
20. **Security headers.** `Content-Security-Policy`, `Strict-Transport-Security`, `frame-ancestors` or `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`. Check the real response with `curl -I`.
21. **CSRF.** Any state-changing request that relies on cookies needs a token or an origin check. A form on another site must fail.
22. **Rate limits.** On login, signup, password reset, costly APIs, and anything that sends email or calls a paid service. Going over returns `429`. Serverless needs a shared store. Behind a proxy or CDN, check that the app sees the real client IP on the deployed app. Otherwise limits hit everyone at once, or nobody.

**Data and money**

23. **Database.** Not open to the public internet. Least-privilege user, strong password, TLS. If the browser talks to the database directly (Supabase, Firebase), row-level rules must be on and tested.
24. **Prices and payments.** The server looks up the price, and never trusts one from the browser. Verify webhook signatures, handle repeat webhooks, and give access only after a confirmed payment.
25. **Logging and alerts.** Log failed logins, denied admin requests, rate-limit hits, and payment errors, and make sure someone gets alerted. Logs must hold no passwords, tokens, or `Authorization` headers.

**Deployment and packages**

26. **Debug mode and errors.** Users see a generic error with a reference ID, and the details stay in server logs. No stack traces, debug routes, or echoing input back in errors.
27. **Default settings.** Code cannot show you what is really deployed, so check the live setup. Any tool you host (admin panels, dashboards, databases) still has its default login until you change it. A missing password setting is a live default. Check for open ports and public storage buckets.
28. **Vulnerable packages.** Run the project's audit tool (`npm audit`, `pip-audit`, and so on). Fix what is reachable and note what is not. Commit the lockfile.
29. **Unused packages.** Fewer packages means less attack surface. Confirm a package is unused before you remove it.

**Close out**

30. **Attacker's read.** Now read the app as an attacker would, and look for what the list misses. For each entry point, ask: what does it expose, what does it trust, what can leak through it, and is it closed by default? For each credential, list everything it can already reach, because the risk is often the old access and not the new code. If a limit can be walked around another way, it protects nothing, so close every path or say plainly that the gap exists.
31. **Prove the protections.** Do not trust a green test suite until you have seen it catch a failure.
    - Send both requests: the bad one must fail, and the good one must still work. A fix that blocks everyone is also broken.
    - Test login and permission apart: a valid login with the wrong role needs its own test.
    - Where the project has tests, add them, then remove the check for a moment. If no test fails, the test proves nothing.
    - Record what came back for each fix, such as the status code and the path tried.

## Report

Write the report in plain, short English. Worst problems first.

```markdown
# Pre-launch security report

**Verdict:** Ready / Ready after open items / Not ready
Fixed: N | Open: N | Verified, no change needed: N | N/A: N

## Open items (need you)
- #<n> <check>, <severity>, confidence <High/Medium/Low>: the attack in one line, what to do. `file:line`

## Fixed and proven
- #<n> <check>: what changed, the request you tried, and what came back. `file:line`

## Verified, no change needed
- #<n> <check>: the evidence.

## N/A
- #<n> <check>: why.

## Not checked
- Anything you could not test, such as host settings or production-only config, and why.
```

Every one of the 31 appears exactly once. "Not ready" means an open item lets someone get in, read other users' data, steal a live key, or skip payment.
