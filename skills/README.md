# Claude Skills

*Reusable [Claude skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) built from the same methodology as the playbook. Each one packages a repeatable part of integration and documentation work so Claude handles it the same way every time.*

Like the rest of this repository, these are generalized versions of skills I use day to day, with anything employer-specific removed.

| Skill | What it does |
|---|---|
| [explain-the-why](explain-the-why/SKILL.md) | Drafts replies that explain a technical fix to a colleague: answer first, then the mechanism, the action, and what to expect. Built so the colleague can handle the issue themselves next time. |
| [wikijs-page-draft](wikijs-page-draft/SKILL.md) | Drafts publish-ready Wiki.js pages for internal wikis or external developer docs, including the Wiki.js syntax traps that only show up after publishing. |
| [integrator-access-review](integrator-access-review/SKILL.md) | Reviews a vendor's access request against least-privilege gate checks and returns a verdict plus a reply you can send. Pairs with the [Integrator Security Playbook](../docs/03-integrator-security-playbook.md). |

## Installing a Skill

Each skill is a folder with a `SKILL.md` file and, for some, a `references/` folder. Keep the folder together when you copy it.

- **Claude apps (web, desktop):** zip the skill's folder and upload it from the Skills section of your Claude settings. See [Use skills in Claude](https://support.claude.com/en/articles/12512180-use-skills-in-claude).
- **Claude Code:** copy the folder into `~/.claude/skills/` for personal use, or into `.claude/skills/` inside a project to share it with that project's team. See [Extend Claude with skills](https://code.claude.com/docs/en/skills).

Once installed, Claude uses a skill automatically when a request matches its description. You can also ask for it by name.

## Making Them Yours

These are starting points. The parts most worth customizing:

- **Voice and channel rules** in `explain-the-why`, to match how your team writes.
- **House facts** in `wikijs-page-draft`: create `references/house-facts.md` with the identifiers, naming conventions, and escalation thresholds your documentation must always state the same way.
- **Gate checks** in `integrator-access-review`, to match your own security standards.

Licensed under the same [CC BY 4.0](../LICENSE) terms as the rest of the repository.
