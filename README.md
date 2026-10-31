# pre-launch-security

A Claude Code skill that audits your whole web app before you launch, the way a security auditor would. It maps every way into the app, picks the checks that fit your project, tests each entry point as each type of user, and reports every vulnerability it finds with a suggested fix.

It is made for vibe-coded and small apps, where nobody has done a security review yet.

## Install

Run this in your terminal:

```bash
claude plugin marketplace add mebmrauf/pre-launch-security && claude plugin install pre-launch-security@mebmrauf
```

Or, if the `claude` command is not on your PATH, type these inside Claude Code, one at a time:

```
/plugin marketplace add mebmrauf/pre-launch-security
/plugin install pre-launch-security@mebmrauf
```

Start a new Claude Code session after installing.

## Use

Open your project in Claude Code and ask: "I launch tomorrow. Is my app safe?"

Or run it by name:

```
/pre-launch-security:pre-launch-security
```

## How it works

1. **Scope:** records the commit, stack, and what it may test.
2. **Attack surface:** lists every route, webhook, job, and upload.
3. **Threat model:** what is worth stealing, and who would try.
4. **Pick the checks:** uses only the ones that fit your project, and says what it skipped and why. A static site gets a short audit, and a payments app gets a long one.
5. **Scanners:** runs the ones you already have.
6. **Entry points, one by one:** tests each as anonymous, user A, user B, and admin, then traces input and attacks the logic.
7. **Chosen checks:** covers login, permissions, secrets, injection, files, traffic, browser code, data, payments, abuse and fraud, AI features, deployment, and packages.
8. **Attacker's view:** chains small issues into bigger ones.
9. **Disprove:** argues against each finding, and proves it locally where it safely can.
10. **Fresh eyes:** runs a second pass with no memory of the first.
11. **Report and fixes:** it changes no code until you pick fixes, then retests each one.

## What makes it different

- **Only what fits.** Checks that do not apply are left out, with the evidence for skipping them.
- **Every finding needs a real attack.** It says who does what and what they get, with a confirmed, likely, or needs-check status.
- **Audit first.** Your code stays as it is until you choose the fixes.
- **A fix is not done until it is proven.** The attack must fail, and normal use must still work.
- **It asks before risky steps.** It will not rotate keys, rewrite Git history, run migrations, or change production settings without your yes.
- **It never shows secrets.** Reports give the file, line, and key type, not the value.
- **Your report stays private.** Working files go in `security-audit/`, which is kept out of Git.

## Limits

This is an AI-led review, not a professional audit or a guarantee. No audit can prove an app cannot be attacked, and this one can miss things. If your app handles payments, health data, or other sensitive data, get a human security review too.

Only run it on apps you own or have permission to test.

## Credits

The idea of a pre-launch checklist came from a reel by [@breakingbreach](https://www.instagram.com/breakingbreach/). Some ideas on proving protections and on everyday backend holes came from [zizi-skills](https://github.com/ziazon/zizi-skills) and [backend-design](https://github.com/sheitabrk/backend-design), both MIT licensed. The text here is written from scratch.

## License

[MIT](LICENSE)
