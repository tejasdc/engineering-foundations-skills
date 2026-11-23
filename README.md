# Engineering foundations skills

Six agent skills for building software on solid foundations. Each one is a folder with a `SKILL.md` that an AI coding agent (Claude Code, Codex, Cursor and others that read skills) loads before it does a kind of work: designing a feature, choosing how state is stored, designing an exchange between two services, deciding how much review a change needs.

They came out of a lot of real building with agents, and most of the rules exist because something broke without them. You can read them as a person too: each `SKILL.md` is a short method, and the `references/` folders hold the detail.

## The skills

| Skill | Use it when | What it gives you |
| --- | --- | --- |
| [architecture-foundations](skills/architecture-foundations/SKILL.md) | Starting a project, designing a feature, reviewing a design | The method: the requirement in the user's words, the real operating profile, who owns each piece of state, the invariant that removes a whole class of bugs, and how to check a design before building it. Start here; it points to the others. |
| [stateful-shapes](skills/stateful-shapes/SKILL.md) | Deciding how data and state are stored and kept in sync | Seven common shapes (event stream, collaborative document, durable workflow, actor per entity, live presence, sync engine, agent thread), when each fits, and when to reach for a library. |
| [protocol-design](skills/protocol-design/SKILL.md) | Two parts of a system talk to each other: client and server, two services, two agents | How to design an exchange that ends explicitly, survives retries and crashes, and never leaves either side guessing. Includes a design template and checklist. |
| [architecture-lessons](skills/architecture-lessons/SKILL.md) | A value, list or option set is needed in more than one place | One home for each fact, and the ways to enforce it so copies cannot drift apart. |
| [application-observability](skills/application-observability/SKILL.md) | Building something that runs on its own, or debugging something that did | Logs, metrics and health checks that let you (or an agent) tell what actually happened. |
| [review-gates](skills/review-gates/SKILL.md) | Deciding how much review a change needs | Three review levels matched to risk, so small changes ship fast and risky ones get a real second look. |

## A starter system prompt

[system-prompt.md](system-prompt.md) is a general version of the global instructions these skills grew up alongside: working principles like verifying before claiming done, finding the invariant instead of adding branches, matching review to risk, and git safety. It is a menu, not a file to paste whole. Ask your coding agent to read it, ask you how you work, and copy only the sections that fit into your global instructions (`~/.claude/CLAUDE.md` or `~/.codex/AGENTS.md`).

## Installing

Easiest: ask your coding agent to install `tejasdc/engineering-foundations-skills` for you.

**Claude Code** (as a plugin, so `/plugin` keeps it updated). Inside Claude Code:

```
/plugin marketplace add tejasdc/engineering-foundations-skills
/plugin install engineering-foundations@engineering-foundations
```

On Claude Code 2.1.275 or later this is one line:

```
/plugin install engineering-foundations --marketplace tejasdc/engineering-foundations-skills
```

**Any agent** (Claude Code, Codex, Cursor and others) through the open skills installer:

```sh
npx skills add tejasdc/engineering-foundations-skills
```

Or copy the folders under `skills/` into your agent's skills directory by hand. The skills link to each other by relative path, so keep them side by side.
