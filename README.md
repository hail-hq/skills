# Give your AI assistant a phone number and an email inbox

Let your assistant handle calls, texts, and email. Building an app? Add AI calls,
email, and SMS with one API.

This repository contains the official [Hail skill](skills/hail/SKILL.md) for AI
assistants and coding agents. It guides an assistant through connecting to Hail,
setting up a number or inbox, answering incoming calls and texts with a saved
agent, sending messages, and reading conversations and delivery events.
Incoming email is available to your assistant or application through the API
and webhooks.

## What your assistant can set up

Through MCP, your assistant can:

- Find and provision phone numbers across Hail's 120+ country coverage, using
  live quotes to check availability, channel capabilities, prices, and paperwork.
- Collect and submit required registration details and documents, including US
  A2P 10DLC applications; preview campaigns, track review, handle confirmation,
  and correct an existing application when the carrier requests more information.
- Create a Hail agent with instructions, greeting, language and voice settings,
  limits, and handover contacts, then route a number's calls and texts to it.
- Make calls, send texts and email, read incoming messages and attachments,
  manage contacts, and inspect transcripts, delivery events, and email statistics.

Purchases and registration submissions need the user's authorization and any
required credits. Carrier approval and inventory determine what is available;
not every country supports every number type or channel.

You can also connect your own application through signed webhooks. Configure
subscriptions through the REST API or CLI: the current MCP catalog has no
webhook-management tool. Explicit inbox/domain creation uses the console or
REST API; MCP can list identities and work with email.

## Use the skill

Read [SKILL.md](skills/hail/SKILL.md), or get the same instructions at
[hail.so/skill.md](https://hail.so/skill.md).

For a client that supports Agent Skills:

```bash
git clone https://github.com/hail-hq/skills.git hail-skills
```

Install `hail-skills/skills/hail` in your client's skill directory. Follow the
client's instructions for the directory location. Update installed copies when
new versions are published; a local installation is a snapshot.

## Connect your assistant or app

- [Console](https://hail.so/console): set up your account, numbers, inbox, and agents.
- [MCP](https://hail.so/mcp): connect an assistant using OAuth.
- [API documentation](https://hail.so/docs): build calls, texts, and email into your app.
- [CLI](https://hail.so/docs/cli): work from the terminal; install with `brew install hail-hq/tap/hail`.

For REST examples, use an existing `HAIL_API_KEY`. Create a new account only
when needed. The skill includes optional assistant-led signup and account
recovery instructions. Keep API keys and account claim links out of Git.

## Setup and pricing

Hail uses pay-as-you-go pricing. A new workspace starts at $0 and needs credits
before sending. Phone numbers can have setup and recurring charges. Review
[current pricing](https://hail.so/pricing) before a purchase.

A funded account still needs the right sender setup. SMS may require a dedicated
number and carrier registration. Custom email domains need DNS verification.
The skill explains these prerequisites and distinguishes a dedicated incoming
number from a shared outbound call line.

## Keep the instructions current

The website's `lib/agent-skill-doc.ts` generates both this repository's
`skills/hail/SKILL.md` and the hosted `hail.so/skill.md`. Maintainers update the
generator, export it into a task worktree of this repository, validate it, and
open linked PRs. Do not edit the generated skill independently.

Review this README and the entire skill when product positioning or platform
behavior changes. The website's publishing runbook and `AGENTS.md` define the
process. Its PR checks validate API examples; weekly checks compare the deployed
API and the two published copies. Checks detect drift, but people or coding
agents still need to update and review the instructions.

The website serves its local generator. This repository needs no website hosting
or Vercel project. GitHub updates do not automatically update an installed skill
or prove that a registry has refreshed its listing.

## Tessl listing

Once the skill is on this repository's default branch, submit
`https://github.com/hail-hq/skills` through
[Tessl's submission page](https://tessl.io/registry/skills/submit).

## License

[GNU AGPLv3](LICENSE), matching the public Hail project's license.
