# Kavach 🛡️ — DPDP compliance skill for Claude

**Kavach** (कवच, "armour") is a Claude skill that turns India's **Digital Personal Data Protection Act, 2023** and **DPDP Rules, 2025** into engineering controls you can build and verify, not just read.

Use it while you design, code, review, or audit any software that handles personal data of people in India.

## What it does

| Mode | Ask Claude something like | You get |
|---|---|---|
| Design | "Design the signup and consent flow for our health app" | Data inventory, purpose registry, controls mapped to the law |
| Build | "Write the consent ledger schema and withdrawal handler" | Code and config with S./R. citations |
| Review | "Review this PR / schema / app for DPDP compliance" | Findings table with evidence, citations, severity, fixes |

It covers 17 controls: lawful basis, notice, consent, consent ledger, withdrawal, Consent Managers, accuracy, security safeguards, breach intimation (72 h), retention and erasure (including the one-year log rule), data principal rights, children, guardianship, Significant Data Fiduciary duties, cross-border transfer, processors, and exemptions.

## Install

**Claude.ai / Claude app:** download `dist/dpdp-claude-dev-skill.skill` and upload it under Skills in Claude's settings.

**Claude Code:** copy the skill folder into your personal or project skills directory:

```bash
# personal (all projects)
cp -r skills/dpdp-claude-dev-skill ~/.claude/skills/

# or per project
mkdir -p .claude/skills && cp -r skills/dpdp-claude-dev-skill .claude/skills/
```

The skill triggers automatically when you work on personal data, consent, deletion, logging, breach response, or mention DPDP.

## Key dates

| Date | In force |
|---|---|
| 13 Nov 2025 | Definitions, Data Protection Board |
| 13 Nov 2026 | Consent Manager registration |
| **13 May 2027** | Notice, consent, security, breach, retention, rights, children, cross-border (most developer obligations) |

## Repo layout

```
skills/dpdp-claude-dev-skill/SKILL.md   the skill
dist/dpdp-claude-dev-skill.skill        packaged for upload
docs/sources.md                         official texts used
examples/prompts.md                     prompts to try
```

## Disclaimer

Kavach is an engineering aid, not legal advice. It flags interpretation calls and sector laws (RBI, SEBI, IRDAI, health, telecom) for legal review. Always check for newer MeitY notifications.

## Contributing

Issues and PRs welcome, especially new MeitY notifications, sector-specific overlays, and real-world code patterns. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT, see [LICENSE](LICENSE). Maintained by [Uvera Innovation Hub](https://uvera.tech).
