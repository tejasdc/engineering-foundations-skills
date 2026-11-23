---
name: review-gates
description: Use when deciding how much review a change needs or before running one — whether a change needs a review, requesting an independent review, SHIP or NO-SHIP verdicts, review until GO, plan review, high-risk classification, scope creep from review, a reviewer who keeps adding edge cases, rapid mode (deploy first, test after). Covers the real operating profile, the minimum sufficient system, one whole delivery (no phases), Focused / Independent / High-risk levels, the three-verdict cap, choosing a fresh independent reviewer, giving the reviewer the requester's original words, requirement-fidelity and overlap findings, and rapid mode (which reviews move after the deploy).
---

# Review gates — proportional scope and review

This is the full authority on how much review a change needs and how to run it. Project
instruction files (for example AGENTS.md) may keep a short summary of the levels.

### Proportional Scope and Review

**Engineer for the real operating profile.** Before choosing architecture, identify the facts that materially affect it: expected users/operators and load, public exposure, data sensitivity, consequence of loss or downtime, reversibility, compliance obligations, and available time. Infer these from the project when clear and state the assumption when it changes the design. Never silently assume internet scale, many operators, formal compliance, or high availability. A personal or hobby project still gets strong controls at a real trust boundary, but that boundary does not make the entire system enterprise infrastructure.

**Build the minimum sufficient system.** Every new service, database, credential, background worker, migration, recovery protocol, or abstraction must be justified by an explicit acceptance criterion, an existing project invariant, or a concrete observed failure. Otherwise keep it out of the current change and record it as possible future work. Prefer the smallest reversible design that meets the requirement and preserves existing behavior.

**Design, implement, and review each requested change as one whole.** A design must describe the complete implementation as one delivery unit; it must not divide the feature into phases, stages, waves, partial activations, or phase-specific acceptance or review gates. Complete the full approved design before implementation review, then review the entire diff and its evidence together. Do not run phase reviews, ship partial implementation phases, or reset the review budget for components or subparts. If a request cannot be safely delivered as one coherent change, stop and ask the user to reduce or redefine its scope; agents and reviewers may not invent a phased program. This governs delivery structure, not targeted tests or correction passes within the same whole change.

A foundation that spans several subsystems is not an exception to this rule but is easy to misread against it. It is delivered as dependency-ordered steps where each step is itself a complete change with its own real-system acceptance check, carrying its own protocol, compatibility and cleanup documentation; what the rule forbids is activating half of one feature. The honest form is complete deliveries in dependency order, each gated by what the user can see; a gate may be removed only when every kind of work it protected is recoverable, not when the first kind is.

**Optimize for changeability, not imagined completeness.** A design is good when a later requirement can replace or extend one part without forcing unrelated rewrites. Introduce an interface or abstraction when it isolates a known variation or ownership boundary; do not build speculative generality for hypothetical futures. Under uncertainty, prefer a small vertical slice with clear seams and a cheap rollback over a comprehensive platform.

**Scope is owned by the user, not the reviewer.** Review feedback does not authorize new product behavior or adjacent projects. If a proposed fix adds a subsystem, credential, persistent state, deployment path, or materially broader contract, classify it as scope expansion and ask before proceeding. When review reveals worthwhile unrelated work, preserve it separately without making it a ship blocker.

Select review depth by risk:

When documented design patterns or a readiness checklist inform the change, use
[contextual pattern reviews](references/contextual-pattern-reviews.md) to select relevant
questions, explain applicability and exceptions, and inspect evidence at the right moment.
This supplies questions within the selected review level; it adds no gate or review session.

| Level | Typical work | Required gate |
| --- | --- | --- |
| **Focused** | Read-only investigation, prose/docs, tests, formatting, small reversible internal changes with no sensitive boundary | Self-review plus the smallest relevant checks. No independent review by default. |
| **Independent** | Meaningful feature or fix that changes runtime behavior but remains bounded and reversible | One fresh-context review of the actual diff and focused/full tests, with an explicit `SHIP` or `NO-SHIP` verdict. |
| **High-risk** | Auth/authorization, secrets, public ingress, destructive or hard-to-reverse data changes, deployment/lifecycle ownership, billing, legal/compliance, or broad blast radius | One primary independent review plus only the targeted specialist reviews needed for the named risks. Run independent specialist reviews in parallel when practical, then synthesize once. |

A project-defined real-environment acceptance gate for a trusted single-operator system replaces the Independent review gate for behavior that gate exercises. Sandbox-only credentials justify a named secret-exposure check, not automatic High-risk treatment of locks, process or browser ownership, or the entire sandbox lifecycle. Outside a named trust risk, review is advisory and must not return `SHIP` or `NO-SHIP`. A source-constructible but unobserved scenario is not blocking without a reproduction, deterministic failing check, observed incident, documented upstream-contract violation, direct contradiction of requested behavior, or user-named trust risk.

A conditional review skill may supply checks inside the selected level; it may not escalate the level, spawn a review swarm, create follow-up work, or broaden the acceptance contract merely because the skill supports those actions.

Plan review is required only when implementation depends on a consequential architecture choice, introduces a long-lived component or credential, crosses ownership boundaries, or performs a difficult-to-reverse operation. A bounded change with an obvious implementation path should go directly to implementation and diff review.

Reviewer findings must identify the violated requirement or invariant, concrete evidence, severity, and the smallest adequate correction. Reviewers judge the approved scope and regression surface; they do not redesign toward an idealized platform. Suggestions and future hardening are non-blocking unless the current change would otherwise be unsafe or fail an acceptance criterion.

For ordinary work, allow at most three review verdicts: the initial review, one re-review after the first correction pass, and one final re-review after the second correction pass. Each re-review checks prior blockers and regressions caused by their fixes; it must not introduce unrelated blockers without new critical evidence. If the third verdict is still `NO-SHIP`, stop the review loop. The responsible agent separates demonstrated in-scope blockers from speculative or out-of-scope suggestions, corrects the demonstrated defects, and checks the final candidate against the requested behavior and real acceptance path. It records how each blocker was resolved and decides whether the evidence supports release. Do not ask the requester to authorize another review: that is not a decision reserved to them. If a concrete acceptance failure or named trust risk remains, keep the release held, report the exact evidence gap and next agent action, and redesign the approach as needed. Ask the requester only when their words admit materially different irreversible outcomes or when they have reserved the action to themselves. A technical consultation with another model can help settle uncertainty but cannot substitute for a missing acceptance check or override an unresolved trust risk. The three-verdict cap applies to high-risk work too; it is not a reason to waive its trust gate. “Review until GO” means resolve justified in-scope blockers, not apply every suggestion indefinitely.

**Prefer bounded production evidence over speculative review when the real environment is the unknown.** Exercise the smallest safe vertical slice against the actual service account, supervisor, credential mechanism, route, or provider early when the probe is reversible, observable, and cannot expose secrets, corrupt user data, or create unacceptable external effects. Define success, failure, blast radius, and rollback before the probe. Use staging or simulation when those conditions cannot be met. Do not keep extending a design review to predict behavior that a contained production preflight can answer directly.

Stop planning and reassess scope when planning/review effort approaches the likely implementation effort, the plan grows to cover more subsystems than the request, reviewer feedback repeatedly adds theoretical edge cases, or the proposed solution becomes less reversible than the problem requires. Separate useful ideas into future work and return to the smallest design that satisfies the acceptance criteria.

**Reviewer provider.** Route required design and implementation reviews to a fresh, independent reviewer: preferably a strong model from a different provider than the author, or at least a fresh session with no memory of authoring. Attach the complete original source, design or actual diff, and verification evidence when the reviewer cannot inspect the machine itself; a reviewer that can read the repository or run commands should be given read-only access. The user's explicit provider, model and effort choices take precedence. The reviewer must be fresh and must not have authored the design or implemented the change. Fresh context is required; a different provider is not. Disclose when author and reviewer share a provider rather than claiming provider independence.

If the preferred reviewer is unavailable (for example, it is rate-limited or out of quota) and the reviewer must inspect the repository or run commands itself, use another fresh read-only reviewer and declare the exception and its reason; a single-provider default must not deadlock already-authorized work. This is not permission to silently change an explicit human reviewer choice or to invent an automatic fallback chain. One rejected model does not establish exhaustion: preserve the actual usage response and attempts.

Record the actual reviewer provider, model, session identity, selected route and reason, and any unavailability evidence in the project's gitignored review-artifacts directory (for example `tmp/reviews/`); review artifacts are not committed. Verify the actual runner's capabilities rather than assuming them. If the selected reviewer cannot complete, preserve the source packet and partial output, report the exact blocker and next recovery action, and keep the required gate unsatisfied. Independent preparation may continue. An unavailable reviewer or partial attempt never counts as a completed review, and changing providers does not waive source completeness, requirement fidelity, concrete acceptance evidence, or a named trust gate.

Every review prompt must state the real operating profile, approved acceptance criteria, explicit non-goals, selected review level, and relevant diff or artifact. It must tell the reviewer that hypothetical scale, adjacent redesigns, and future hardening are non-blocking unless they demonstrate a current requirement, invariant, or safety violation. Ask for the smallest adequate correction rather than an idealized redesign.

**Original source is a review prerequisite.** Give the reviewer the user's complete original request or spoken transcript, the original attachments carrying requirements, and all task-relevant later user corrections and scope decisions, in chronological order and separately from the design or implementation artifact. Read this source before assessing the solution. Preserve wording and source identities; an author's requirements list, plan, handoff, search snippet, or cumulative summary cannot replace it. Retrieve exact requester records from the session transcripts or the original supplied transcript and attachments. Cancellation is not evidence that a user withdrew a requirement.

Preserve a review copy in the review-artifacts directory with source session/message/file identifiers and the expected byte lengths of supplied text files. Deliver long source as complete files or ordered parts, and verify that the reviewer received and read it without truncation before issuing its assessment. Copying integrity does not prove upstream completeness. If a source is missing, truncated, or cannot be established as complete, explicitly report a source-completeness blocker and recover the original or ask the user for it. Do not review around the gap or treat requirement fidelity as verified. Redact secret values and disclose the redaction without changing the requirement's meaning.

**Requirement fidelity is a first-class finding class.** First map requested outcomes and constraints to the design or implementation. Report myopic interpretation, omitted or substituted requirements, and silent scope drift as requirement-fidelity findings in their own right. Each finding must cite exact user wording and its source, identify the artifact's mismatch, state severity and consequence, and give the smallest adequate correction. A plan's approval or an author-written non-goal does not establish that the user withdrew an original requirement; require an explicit user scope decision. A demonstrated mismatch is a requested-behavior violation, not a style preference. If the original request supports materially different interpretations, identify the ambiguity and obtain clarification rather than inventing a requirement.

**Overlap with existing mechanisms is a first-class finding class, checked before quality.** A reviewer given only the diff judges the diff; it cannot see that the change rebuilds something that already runs. So every design and implementation review brief has a required section, **Existing mechanisms**:
- The author lists each responsibility the change adds: a watcher, restart, notification, retry, repair, agent launch, store, route or schedule.
- Beside each, the author names what already does it, or says that nothing does and how they searched.
- The reviewer verifies that list instead of trusting it. It searches:
  - the owning repository;
  - sibling repositories that deploy to the same machine;
  - their agent-instruction files and architecture docs;
  - the installed units, timers and scheduled jobs (for example `systemctl list-units` and `systemctl list-timers` on Linux).
- Each overlap is reported as a finding with its evidence, and with one of three outcomes: merge into the existing owner; delete the duplicate; or keep both, with the boundary between them stated in one sentence.

A brief without this section is incomplete in the same way as one without the original source. Example: an outside supervisor passed three high-risk reviews while most of its checks duplicated what the init system already did. Every brief gave the reviewers the diff and the safety questions, and none asked what already did the job; the designer had also skipped the inventory of owners that [architecture-foundations](../architecture-foundations/SKILL.md) requires.

### Rapid mode: review after the deploy

Rapid mode is a special mode the requester switches on, never the default: every other release keeps
its reviews and checks before release. When the requester is using a deployed product live and asks
for rapid mode, the level still decides *what* review a change gets, but not *when*: Focused and
Independent reviews, and any test that takes more than a few minutes, run after the change is
live, in the background, against the live build; a finding becomes the next fix, shipped the same
way. Only what a fast follow-up cannot undo is reviewed before the deploy: High-risk work that
changes stored data, its deletion or migration, sign-in or secrets, and the release path itself.
Never batch fixes to save a review. The principle: reviews and slow tests happen in the background; testing is not skipped, it is done after the deploy. A project adopting
rapid mode needs the same refusal in its own release command (for example, one that
requires each release to declare which checks ran or that it is a rapid release, and refuses a change to stored data or
sign-in without an explicit data-review line); this paragraph alone is not enforcement.

### Running the selected review

Use a file-backed prompt and the complete original-source packet with the selected route above. Give the reviewer only the read access needed to inspect the work; save its full output and actual session identity under the review-artifacts directory, and have the parent read the saved result. Track background processes and apply the applicable review budget. Label the review with the provider that actually ran it. Reviewer selection, explicit human choices, and unavailable-provider behavior are governed by **Proportional Scope and Review** above.

A verdict of SHIP on behavior needs acceptance evidence: what the user would see, observed by something other than the actor. A reviewer who sees only saved records or the acting tab returns NO-SHIP for that claim.
