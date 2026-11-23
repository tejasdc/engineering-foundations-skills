---
name: architecture-lessons
description: "Use BEFORE adding a value, list, enum, option set, limit or default that more than one file, package, service or repository will need — a model list, a dropdown's options, a status vocabulary, a timeout, a feature's allowed values — and before changing one that already exists in more than one place. Triggers on: one source of truth, single source of truth, a change that did not propagate to every place, two lists that disagree, a stale picker or dropdown, a value hard-coded in two places, adding a value to an enum, finding everywhere a value is used. Covers: which piece of knowledge gets one home; the three enforcements ranked by strength (derive so a gap cannot compile, publish so consumers read instead of copying, refuse the second copy with a check); why a derived guard you have not watched fail is not a guard; why a fallback list is the second copy in disguise; and how to inventory the duplicates you already have. Do NOT use for duplicated *code* that merely looks alike — that is not what DRY means and deduplicating it usually makes things worse."
---

# One home per piece of knowledge

This skill owns one architectural rule: duplicated knowledge. For the method of designing or
reviewing a whole system or feature (requirement in the requester's words, operating profile, lifetimes
and owners, the invariant, exchanges that end explicitly, rollout that reaches running code, the
comparative decision case, checks before building) load [architecture-foundations](../architecture-foundations/SKILL.md), which routes
here for this rule. The rule also applies to splitting stores by what they own: for example, a
table of obligations (what must still happen, with cursors) and a log of observations (what was
seen) should each own their facts, and the log must never independently settle an obligation.
The rest of such a review needs the broader skill.

**DRY is about knowledge, not about text.** Hunt and Thomas wrote it as *"every piece of
knowledge must have a single, unambiguous, authoritative representation within a system"*.
Two functions that happen to look alike are not a violation. Two places that both have to be
*told* when something changes are — and that is the failure this skill exists to stop.

The test is not "does this look like something else". It is: **when this fact changes, how
many places must be edited, and what happens if one is missed?** If the answer is "the wrong
thing quietly shows up somewhere", it needs one home now.

## The three enforcements, strongest first

Reach for the strongest one you can afford. A rule nobody *can* break beats one everybody
must remember, and "we'll be careful" fails.

**1. Derive it, so a gap cannot compile.** Best. Make the second place a *consequence* of the
first rather than a copy of it: a map keyed by a literal union, an exhaustive switch, a type
generated from a schema. A missing entry becomes a build error at the moment it is created.

> **A derived guard is only real once you have watched it fail.** Write the broken version
> on purpose, build it, and see the error. One first attempt at exactly this
> derived a union through a field typed `string`; the union widened to `string`,
> `Record<string, string>` accepted anything, and a deliberately broken build passed clean.
> The guard was decorative and would have been shipped as a guarantee.

**2. Publish it, so consumers read instead of copying.** Across a process or repository
boundary you cannot share a type, so serve the value and have clients render what arrives.
The owner gets an endpoint; the client keeps nothing.

> **A fallback list is the second copy in disguise.** When the published list cannot be
> reached, say so and offer the safe default. Do not "temporarily" hold a copy for that case
> — it will be the copy that goes stale, and it will look like it is working.

**3. Refuse the second copy with a check.** A lint or architecture test that rejects the
literal anywhere but its one home. Name known exceptions *in the rule, with their reason*, so
they stay visible and a genuinely new copy is still caught. An exception list that is empty
because you widened the pattern is not enforcement.

## When you find one, inventory the rest

The copy someone noticed is rarely the only one. Sweep both directions — same repo and across
repos — for the same shape: option lists mirroring a backend union, status vocabularies,
limits written twice. Report each as *what / where each copy is / do they agree today*, and
fix the ones that have already drifted. For copies you are not fixing now, **write them down
somewhere durable with a reason.** A known duplicate is survivable; an unknown one is what
reaches a user's screen.

### An action catalogue follows the object across surfaces

An action list shared across phone and laptop is still duplicated knowledge if each page,
row, preview or window chooses its own subset. Give the object's actions one catalogue and
controller; presenters choose frequent shortcuts, while their visible menu exposes the complete
applicable set. Editing windows load their own required metadata and destinations when opened,
rather than depending on another page having loaded them. Enforce known entrances and reject
local action catalogues with the existing architecture check. Observe the missing action on the
installed surface, then exercise each entrance and an independent open observer after repair.

Example: in one app, several object actions existed only inside one inline panel, while list
rows offered a smaller menu and detached windows lacked the destinations those actions need. A
single action-list rule already covered one surface; the missing invariant was the same object on
different surfaces.

## An illustration

A service adopted a new model version. The web app's new-session picker went on offering the
previous two model names, because the same list was written out in the web app twice: once as the
picker's options, once as the display names.

Sweeping for others found three more the same drift had already reached, none of them the picker
that was reported: a local runtime that **defaulted to a retired model**, so unlike the display
bugs it changed which model actually ran; a provider dropdown several options behind its backend;
and an event-kind list missing an entry, so one kind of update never refreshed a cached view. Two
of the three were invisible from any screen.

**The lesson is not "remember to update the other place."** It is that a value with more than
one home will drift, the drift will be invisible until someone notices the wrong thing on a
screen, and the copies you find by looking are usually worse than the one that was reported.

See `references/inventory-method.md` for how to run the sweep.
