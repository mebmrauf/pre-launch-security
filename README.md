# pre-launch-security

A Claude Code skill that checks your whole web app for security problems before you launch. It runs 31 checks, fixes what is safe to fix, proves each fix holds, and gives you a plain-English report.

It is made for vibe-coded and small apps, where nobody has done a security pass yet.

## Install

```bash
git clone https://github.com/mebmrauf/pre-launch-security.git
mkdir -p ~/.claude/skills
cp -R pre-launch-security/skills/pre-launch-security ~/.claude/skills/
```

Start a new Claude Code session after installing.

## Use

Open your project in Claude Code and run:

```
/pre-launch-security
```

Or just ask: "I launch tomorrow. Is my app safe?"

## What it checks

| Area | Checks |
| --- | --- |
| Accounts and access | Login, passwords, password resets, session cookies, admin routes, who owns which data |
| Secrets | Keys in the browser, env vars, Git history, leaked keys, exposed files |
| Input | Server-side validation, mass assignment, injection, XSS, file uploads, user-supplied URLs |
| Browser and traffic | HTTPS, CORS, security headers, CSRF, rate limits |
| Data and money | Database access, prices and payments, logging and alerts |
| Deployment and packages | Debug mode, default logins, vulnerable packages, unused packages |
| Close out | A read through the app as an attacker, then proof that each protection works |

## What makes it different

- **Every finding needs a real attack.** If it cannot say who does what and what they get, it does not report it as a problem.
- **A fix is not done until it is proven.** The bad request must fail and the good request must still work.
- **It asks before risky steps.** It will not rotate keys, rewrite Git history, run migrations, or change production settings without your yes.
- **It never shows secrets.** Reports give the file, line, and key type, not the value.

## Limits

This is an AI-led review, not a professional audit or a guarantee. It can miss things. If your app handles payments, health data, or other sensitive data, get a human security review too.

Only run it on apps you own or have permission to test.

## Credits

The idea of a pre-launch checklist came from a reel by [@breakingbreach](https://www.instagram.com/breakingbreach/). Some ideas on proving protections and on everyday backend holes came from [zizi-skills](https://github.com/ziazon/zizi-skills) and [backend-design](https://github.com/sheitabrk/backend-design), both MIT licensed. The text here is written from scratch.

## License

[MIT](LICENSE)
