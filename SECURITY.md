# Security

## Reporting a vulnerability

Please report security issues privately. Do not open a public issue, and do not
post details in Telegram or Discord.

**Email:** chris@zuztrade.com

Useful things to include: what you found, how to reproduce it, which surface it
affects (the bot, the Mini App, the API), and how you would expect it to be
exploited. If you need to share a proof of concept, say so and we will arrange
a channel.

What to expect:

- An acknowledgement within 72 hours.
- An assessment and a planned fix date, or an explanation of why we disagree.
- Credit in the release note if you would like it, or none if you prefer.

Please give us a reasonable window to ship a fix before publishing. We will not
pursue anyone who reports in good faith, stays within their own accounts, and
does not access, modify or withhold other people's funds or data.

### Out of scope

Reports from automated scanners with no demonstrated impact, missing headers
with no exploit path, social engineering of our team or users, denial of
service, and anything requiring physical access to a user's unlocked device.

## Custody model — read this before funding anything

**Zuz is custodial today. It is a hot wallet.** Keys are generated and held
server-side so the bot can execute exits while you are not watching. That is the
design trade: automated take-profit, stop-loss and anti-rug exits require the
server to be able to sign without you present.

What that means for you, stated plainly:

- **Whoever controls the server can, in principle, reach user keys.** No
  encryption design changes that while signing happens server-side.
- **Keep only what you trade in Zuz.** It is not a savings wallet and it is not
  a substitute for a hardware wallet.
- **Your keys are exportable at any time.** Both the raw private key and the
  24-word recovery phrase can be exported, behind two-factor authentication.
  You are never locked in.

Non-custodial execution — embedded wallets where the user holds the key and
grants a narrow signing permission for exits — is the intended direction, not a
promise with a date on it.

## Current status

We are preparing the code for an external security review. The trading source
stays private until then; reviewers and ecosystem partners can request read
access at the address above.

A detailed description of the security model, including our own list of known
weaknesses, is shared directly with reviewers rather than published here. We
will publish a fuller write-up once the review concludes and its findings are
fixed.

## Two-factor authentication

2FA is available and we strongly recommend turning it on. It is **required** for
every withdrawal, in the bot and in the Mini App, for exporting a key or
recovery phrase, and for importing a phrase.

Set it up with `/2fa` in the bot.
