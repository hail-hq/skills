# Hail agent skills

Official skills for agents that use [Hail](https://hail.so) to send email, SMS,
and voice calls, manage contacts, and query delivery events.

## Hail

Read [`skills/hail/SKILL.md`](skills/hail/SKILL.md), or use the hosted version at
[hail.so/skill.md](https://hail.so/skill.md).

To download the skill for an agent that supports Agent Skills:

```bash
git clone https://github.com/hail-hq/skills.git hail-skills
```

Install the `hail-skills/skills/hail` folder in your agent's skill directory.
Follow your client's instructions for the directory location.

The skill explains how to use an existing `HAIL_API_KEY` or sign up with your
owner's authorization. New workspaces start with a $0 balance. Your owner must
add credits before sending. Never commit API keys or claim links.

## Maintenance

This repository contains a generated copy of the skill served by the Hail
website. Maintainers update the website's `buildSkillMd` generator, run
`scripts/sync-agent-skill.mjs` from that repository against a task worktree of
this repository, and open linked PRs. Do not edit the generated skill directly.

The export includes the neutral skill without signup attribution. The website
serves its own local generator and does not fetch this repository at runtime.

## Tessl

Submit `https://github.com/hail-hq/skills` through
[Tessl's skill submission page](https://tessl.io/registry/skills/submit) once the
skill is present on the default branch. Tessl can discover the committed
`skills/hail/SKILL.md`.

## License

GNU AGPLv3; see [LICENSE](LICENSE), matching the public Hail project's license.
