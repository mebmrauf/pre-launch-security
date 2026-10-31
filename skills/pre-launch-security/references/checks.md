# Audit checklist

This is a menu, not a list to run in full. Use the "Applies when" line to pick the groups this project needs, and record what you skip and the evidence in `security-audit/plan.md`. A group can apply in part, so also drop single checks that do not fit, such as AUTH-7 with no social login. For what you keep, mark each check as finding, verified OK, or not covered, and back each result with something you read or ran. Add checks this list misses when the project needs them.

## Login and accounts

*Applies when:* The app has login, accounts, or sessions, including through a provider.

- **AUTH-1 Authentication.** Uses a proven library or provider. Every private page and API route checks the session on the server. Hiding something in the UI does not protect it.
- **AUTH-2 Passwords.** Hashed with argon2, bcrypt, or scrypt. Never logged. Weak and breached passwords are refused.
- **AUTH-3 Account guessing.** A wrong email and a wrong password look the same: message, status, and about the same response time. The same goes for signup and password reset.
- **AUTH-4 Password resets.** Short expiry, single use, random token, stored hashed. Old and used links fail. The link's host does not come from the request's `Host` header.
- **AUTH-5 Session cookies.** `HttpOnly`, `Secure`, `SameSite=Lax` or stricter, sensible expiry. New session on login, so a session set before login cannot be reused. Destroyed on logout and on password change.
- **AUTH-6 Tokens.** JWT signature checked, with the algorithm fixed, so `none` and key confusion fail. Issuer, audience, and expiry are checked. The signing secret is strong. There is a way to revoke them.
- **AUTH-7 OAuth and social login.** `state` is checked, PKCE where it applies, and redirect URLs must match exactly. Accounts do not link on an unverified email.
- **AUTH-8 Multi-factor and email changes.** MFA cannot be skipped by calling the next step directly. Codes are rate limited. Changing email or password needs the current password, or a fresh login.
- **AUTH-9 Brute force.** Login, reset, and code checks are limited by account as well as by IP.
- **AUTH-10 Email verification and invites.** A new account cannot act as an email it has not proven. Invite and magic links expire, work once, and are tied to the right person.
- **AUTH-11 User API keys.** Keys the app gives to users are stored hashed, have limited scope, can be revoked, and are not put in URLs.
- **AUTH-12 Account deletion and session list.** Deleting an account ends its sessions and keys. Users can end other sessions after a suspected takeover.
- **AUTH-13 Support and impersonation tools.** Staff "log in as user" features need a strong login, and each use is logged.

## Permissions

*Applies when:* The app has more than one user, role, or customer, or any data that is not public.

- **PERM-1 Who owns the data (IDOR).** The most common serious hole. Every route with an ID confirms the record belongs to the caller, in the data query. As user A, request user B's record. Prefer `404` to `403`.
- **PERM-2 Admin routes and roles.** The role is checked on the server for each route and API call, from one shared check. As a normal user, request `/admin` and call admin APIs directly.
- **PERM-3 Old power.** If a role lives in a token or session, a role change or removal also ends that power.
- **PERM-4 Mass assignment.** `update(req.body)` can set `role`, `isAdmin`, `tenantId`, or `price`. Only a named list of fields can be changed.
- **PERM-5 Multi-tenant.** Every query is limited by tenant. Check search, exports, reports, background jobs, and file links too.
- **PERM-6 Extra data in responses.** The API returns only needed fields, with no password hashes, tokens, or other users' details, even if the UI hides them.
- **PERM-7 Other paths.** Old API versions, GraphQL, exports, and mobile endpoints enforce the same rules. GraphQL checks each field, and limits query depth and size.
- **PERM-8 Share links and guessable IDs.** Share links use long random tokens, can expire, and can be revoked. Sequential IDs alone never grant access.
- **PERM-9 Paid plans and limits.** Plan features, seat counts, and quotas are enforced on the server, not only hidden in the UI.

## Secrets

*Applies when:* Always. Every project with a Git repo gets SEC-3 and SEC-5. The rest apply if the project uses keys or env vars.

- **SEC-1 Keys off the browser.** Browser code calls your server, and your server calls the third party. Search the built client bundle and public env vars, like `NEXT_PUBLIC_*` and `VITE_*`.
- **SEC-2 Environment variables.** `.env` is in `.gitignore`, production differs from dev, and the app fails fast if one is missing. No real values in `.env.example`.
- **SEC-3 Git history.** Search all commits and branches, not only current files. Use `gitleaks git --redact --no-banner`, or `git log -p --all` with patterns for key formats, showing only file and commit. A range scan with a wrong base can report clean, so check the base.
- **SEC-4 Leaked keys.** Any secret found anywhere counts as leaked, even if deleted now. Kill the key first, then think about history. List what to rotate and where.
- **SEC-5 Exposed files.** `.env*`, backups, `db-dump.sql`, `.git/config`, and source maps are not public. Request the paths on a running build.
- **SEC-6 Key scope.** Each key has the least power it needs. Use separate keys for dev and production.
- **SEC-7 Crypto.** Uses a standard library, with no home-made encryption. Random values for tokens come from a secure source, not `Math.random`.
- **SEC-8 Secrets in other places.** Search build logs, CI output, Docker image layers, error trackers, notebooks, and client-side error reports.

## Input and injection

*Applies when:* The app takes any input a user or outside system controls. Pick by what it actually uses: no SQL means no INJ-2, and no exports means no INJ-12.

- **INJ-1 Server-side validation.** The server enforces the rules, and browser checks are only for UX. Say what is allowed and reject the rest.
- **INJ-2 SQL.** Parameterized everywhere, including raw queries inside an ORM, and including sort and column names.
- **INJ-3 NoSQL.** Input types are checked, so `{"$ne": null}` cannot pass as a value.
- **INJ-4 Commands.** Argument lists, never a shell string with user input.
- **INJ-5 XSS.** Encoded on output. Search for `dangerouslySetInnerHTML`, `innerHTML`, `v-html`, and markdown or rich text rendering. `javascript:` links are blocked. Stored content is checked as well as reflected.
- **INJ-6 Templates and code.** User input never reaches `eval`, `new Function`, or server-side template compile.
- **INJ-7 Path traversal.** Filenames and paths from users cannot reach `../`. Zip files are unpacked safely.
- **INJ-8 Unsafe parsing.** No unsafe deserialization such as `pickle` or `yaml.load` on untrusted data. XML has external entities turned off.
- **INJ-9 Prototype pollution.** Deep merges of user JSON block `__proto__` and `constructor`.
- **INJ-10 Header and email injection.** Newlines are rejected in values that reach headers and email fields.
- **INJ-11 Slow regex.** User input does not reach regexes that can hang the server.
- **INJ-12 CSV export.** Cells starting with `=`, `+`, `-`, or `@` are made safe.
- **INJ-13 Log injection.** Newlines in user input cannot forge log lines, and log viewers do not run HTML from logs.
- **INJ-14 HTML-to-PDF and image tools.** PDF generators and image processors do not fetch internal URLs or run scripts from user content. These tools are kept up to date.

## Files and URLs

*Applies when:* The app handles uploads or stored private files (FILE), fetches URLs on the server (URL-1), or redirects to a URL from a request (URL-2).

- **FILE-1 Uploads.** Real file type checked on the server, with a max size. Generated stored name. Not served as runnable code from your main origin. SVG and HTML treated as dangerous.
- **FILE-2 File access.** Private files use short-lived signed links or a permission check. Bucket is not public or listable.
- **URL-1 Server fetches (SSRF).** For webhooks, image proxies, and link previews, allow only known hosts and HTTPS. Block private and metadata addresses like `169.254.169.254`. Check all resolved IPs, and refuse redirects.
- **URL-2 Open redirects.** Redirects after login only go to your own paths.
- **FILE-3 Hostile files.** Limits on image dimensions and unzipped size stop decompression bombs. Location data is stripped from public photos. Files shared with other users are scanned for malware where the risk is real.

## Browser and traffic

*Applies when:* The project is served on the web. A purely static site usually needs only WEB-1, WEB-3, and WEB-8. WEB-4 needs cookie-based sessions, and WEB-7 needs WebSockets.

- **WEB-1 HTTPS.** HTTP redirects to HTTPS everywhere, with HSTS. Outbound calls keep TLS checks on.
- **WEB-2 CORS.** A list of known origins. Never `*` with credentials, never echoing the request's origin, and no `null` origin.
- **WEB-3 Security headers.** `Content-Security-Policy` without loose `unsafe-inline`, `Strict-Transport-Security`, `frame-ancestors` or `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`. Check the real response.
- **WEB-4 CSRF.** State-changing requests that rely on cookies need a token or origin check. No state changes on GET.
- **WEB-5 Rate limits.** On login, signup, reset, costly APIs, and anything that sends email, SMS, or calls a paid service. Serverless needs a shared store.
- **WEB-6 Real client IP.** Behind a proxy or CDN, the app sees the real IP on the deployed app, and a user cannot fake it with `X-Forwarded-For`.
- **WEB-7 WebSockets.** Login is checked on connect, and permission is checked on each message. The origin is checked.
- **WEB-8 Caching.** Private pages are not stored by a CDN or shared cache. Cache keys cannot be poisoned by headers.
- **WEB-9 Resource abuse.** Limits on body size, page size, and costly work, so one user cannot run up the bill or stall the app. Check by reading the code, and do not load test.
- **WEB-10 Host header.** The app uses a fixed list of hosts when it builds links or redirects, and does not trust the request's `Host` or `X-Forwarded-Host`.
- **WEB-11 Method tricks.** Permission checks cover every HTTP method, including `HEAD`, `OPTIONS`, and method-override headers.
- **WEB-12 TLS and certificates.** Old TLS versions are off, and certificates renew on their own.
- **WEB-13 Exposed tooling.** `/swagger`, `/graphql` playgrounds, `/metrics`, `/debug`, `/actuator`, phpMyAdmin, and similar are closed in production.

## Data and money

*Applies when:* The app stores data (DATA) or takes payments, credits, coupons, or anything else of value (PAY).

- **DATA-1 Database access.** Not open to the public internet. Least-privilege user, strong password, TLS.
- **DATA-2 Browser-to-database apps.** For Supabase, Firebase, and similar, row-level rules are on for every table and bucket, and tested as each user. The service key never reaches the browser.
- **DATA-3 Personal data.** Only what is needed is collected. It is not in URLs or logs, and it is encrypted in transit and at rest where required. Delete really deletes it.
- **DATA-4 Backups.** Private, encrypted, and restorable.
- **PAY-1 Prices.** The server looks up price, tax, and discount. Tamper with them in the request.
- **PAY-2 Webhooks.** Signatures verified on the raw body. Replays and repeats are handled.
- **PAY-3 Access after payment.** Given only after confirmed payment, and removed on refund or cancel.
- **PAY-4 Races and limits.** Send two requests at once for coupons, credits, withdrawals, stock, and free trials. Use transactions or locks.
- **DATA-5 Seed and test data.** No default admin, demo accounts, test API keys, or test-only routes in production. Staging does not hold real user data without the same protection.
- **DATA-6 Sharing with vendors.** Analytics, session replay, error trackers, and AI providers do not receive passwords, tokens, or more personal data than needed.
- **DATA-7 Privacy rules.** If the app has users in places with data laws, such as GDPR, flag what the user must handle: consent, export, deletion, and breach notice. This is not legal advice.
- **PAY-5 Money edge cases.** Check negative amounts, rounding, currency swaps, refund after use, and double refunds. Card data never touches your server, since a provider should hold it.

## AI features

*Applies when:* The app calls an LLM or sends data to one.

- **AI-1 Prompt injection.** User text, web pages, and documents can carry instructions. The app must stay safe even if the model obeys them.
- **AI-2 Model output.** Treated as untrusted, so it is encoded before display, and never run as SQL, shell, or code without checks.
- **AI-3 Model tools.** Tools run with the user's permissions and not the app's. Risky actions need confirmation.
- **AI-4 Data leaks and cost.** One user's data cannot reach another through context or memory. Model calls have per-user limits.
- **AI-5 Hidden prompts and retrieved data.** No secrets in system prompts, since they can leak. Documents used for retrieval obey the user's permissions, and nobody can plant instructions in shared ones.
- **AI-6 MCP and agents.** Connected servers have least-privilege tokens, and agents cannot take costly or destructive actions without a limit or a confirmation.

## Deployment and supply chain

*Applies when:* Always for PKG-1 and OPS-1. The rest depend on what the project runs: OPS-2 and OPS-3 need self-hosted services, OPS-5 needs its own domain, and PKG-3 needs CI.

- **OPS-1 Debug mode and errors.** Users see a generic error with a reference ID. No stack traces, debug routes, or input echoed in errors.
- **OPS-2 Default logins.** Hosted tools (admin panels, dashboards, databases) have changed their default logins. A missing password setting is a live default.
- **OPS-3 Open services.** Only needed ports are open. Admin tools, staging sites, and preview links are not public.
- **OPS-4 Cloud settings.** Storage buckets are private and IAM roles are small. Where you cannot see these, mark "Needs check" and tell the user what to look at.
- **OPS-5 DNS and email.** No subdomains pointing to dead services. SPF, DKIM, and DMARC are set if the app sends email.
- **OPS-6 Logging and alerts.** Failed logins, denied admin requests, rate-limit hits, and payment errors are logged, and someone gets alerted. No passwords, tokens, or `Authorization` headers in logs.
- **PKG-1 Vulnerable packages.** Run the audit tool. Say which issues are reachable. Lockfile is committed.
- **PKG-2 Unused and risky packages.** Remove what is unused. Look for misspelled names, unmaintained packages, and surprising install scripts.
- **PKG-3 CI and build.** Secrets are not exposed to pull requests from forks. GitHub Actions are pinned by commit SHA. Workflow permissions are small. Container base images are supported versions.
- **OPS-7 Containers and servers.** Do not run as root. Keep images small and systems patched, and do not mount the Docker socket into an app.
- **OPS-8 Serverless and edge.** Each function gets only the env vars and permissions it needs. Preview deployments do not use production secrets or data.
- **OPS-9 Incident readiness.** Tell the user if there is no way to roll back a deploy, restore a backup, rotate all keys, or end all sessions quickly. Name who gets alerted.
- **PKG-4 Hooks and agent config.** Git hooks, editor config, and AI agent settings in the repo can run code on a developer's machine. Check them, and treat MCP servers as dependencies.

## Browser code

*Applies when:* the project ships its own JavaScript to the browser or has a server-rendered frontend.

- **CLIENT-1 Data sent to the page.** Server-rendered props, such as `__NEXT_DATA__`, hold no secrets or other users' data. Admin code is not sent to normal users.
- **CLIENT-2 Token storage.** Long-lived tokens are not in `localStorage` when an `HttpOnly` cookie would work, because any XSS can read them.
- **CLIENT-3 Third-party scripts.** Each outside script can read the whole page. Keep the list short, limit them with CSP, and avoid loading them on login and payment pages. Pin CDN scripts with integrity hashes where you can.
- **CLIENT-4 `postMessage`.** Message handlers check the sender's origin, and messages never target `*` when they carry data.
- **CLIENT-5 Links and service workers.** Links to user-given sites use `rel="noopener noreferrer"`. Service workers do not cache private pages.
- **CLIENT-6 Client-only rules.** Anything enforced only in browser code, such as feature locks or hidden buttons, counts as not enforced.

## Abuse and fraud

*Applies when:* anyone can sign up, send a message, or trigger email, SMS, or paid work without much friction. A public contact form is enough.

- **ABUSE-1 Spam through your app.** Contact forms, invites, and share features cannot be used to send email to strangers, and user text cannot fill in the `To` or `Subject` fields.
- **ABUSE-2 Fake signups and trials.** Limits stop mass signups, free-trial reuse, and referral fraud. Add a CAPTCHA where abuse is likely.
- **ABUSE-3 Costly actions.** SMS, AI, and other paid features have per-user and total spending caps, so an attacker cannot run up the bill.
- **ABUSE-4 Harmful user content.** If users can post publicly, there is a way to report and remove content. Check for phishing links and hosted malware.
- **ABUSE-5 Scraping.** Listing and search endpoints cannot give away all users or all data through paging.
