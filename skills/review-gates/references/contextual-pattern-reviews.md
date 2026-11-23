# Contextual pattern reviews

Use this reference when a documented design pattern or readiness question could inform a
specific change. Keep the existing review level, scope and acceptance authority. This is
an agent worksheet within that review, not a new service, global gate or requirement to
implement every pattern.

## Find the source, then select

Read the original request and any existing synthesized note, product requirements and
relevant specialist guidance. Link their authoritative homes rather than creating another
pattern collection. Distinguish research evidence, an authored principle, an open
question, a requirement, an example and a possible implementation. Preserve tentative
language: an example or an attractive pattern does not become a requirement by appearing
in a checklist. A stated requirement cannot be waived by calling it a pattern exception.

| Context | Existing authority to consult |
| --- | --- |
| System structure and alternatives | [architecture-foundations](../../architecture-foundations/SKILL.md) and the project's behavior contracts |
| Interface choices and their reasons | the project's design notes |
| Screen space and placement | the project's responsive-design guidance and browser checks for observed geometry |
| State, recovery and exchanges | [stateful-shapes](../../stateful-shapes/SKILL.md) and [protocol-design](../../protocol-design/SKILL.md) |
| Release evidence and review depth | [review-gates](../SKILL.md) and the project's test guidance |

These route to existing sources; they do not require loading every skill for every feature.

For the affected behavior, ask which pattern addresses a real user objective or consequential
failure. Write why it applies, why now, and what an answer can change. Do not enumerate the
whole library as a ritual. Reuse an existing verified control's evidence where sufficient.

For each selected question, record these facts in the existing design or review artifact:

| Fact | What to record |
| --- | --- |
| Source and status | Exact source link and whether this is research, principle, question, requirement, example or candidate implementation |
| Context and reason | The affected behavior, applicability condition, and user benefit or hazard |
| Question and timing | One answerable question; design decision, readiness pause, or execution step |
| Evidence | Expected and observed behavior, environment/candidate identity, evidence link and limitations |
| Disposition | Satisfied, change needed, unknown, not applicable, or justified exception |
| Reason and next action | Evidence supporting that disposition; who acts next if unresolved and what change would reopen it |

These are useful facts, not a demand for a second form when the existing artifact already
contains them. Unknown is not a pass or automatically a defect. Not applicable means the
trigger is absent. An exception means it is present but a different solution better meets
the objective: state the tradeoff, remaining risk and evidence. Binding policy and product
requirements still follow their actual authority; this worksheet cannot override them.

Keep the original requirements visible even when no catalog pattern matches. Distinguish
an alternative that satisfies the purpose from a deliberate decline or deferral that leaves
a tradeoff unresolved; the latter needs the existing decision authority. Explain exclusions
for plausible consequential candidates, not every unselected entry in the collection.

## Put the question where it can be answered

- At design time, compare alternatives and decide what capabilities and interactions belong.
- At readiness review, confirm implemented behavior against agreed requirements and evidence.
- During deployment or recovery, follow the existing execution runbook; do not turn the
  review into an improvised sequence of production actions.

Do-confirm means doing the work and then verifying critical conditions; read-do means
performing actions as the checklist directs. Challenge-and-response describes coordination,
not a third mutually exclusive checklist type. A second agent reading the same account is
not independent observation of the behavior. The proof requirements in [review-gates](../SKILL.md) apply.

## Examples of contextual selection, not universal requirements

| Candidate pattern | When to ask | Evidence and possible exception |
| --- | --- | --- |
| Explain recommendations and learn from feedback | The feature makes recommendations | Inspect whether the user can understand why this suggestion appeared and provide relevant feedback. A fixed explicit choice is not a recommendation; don't add ranking or feedback machinery to it. |
| Make secondary actions available without crowding the main task | Multiple relevant actions compete for space | Identify essential visible actions and demonstrate how secondary ones are reached at actual screen sizes. Frequent or urgent actions may deserve direct exposure; hiding every action is not the goal. |
| Separate capability from presentation | A requested capability is useful in different contexts or views | Inventory existing operations, then show what is exposed here and why. Reuse what exists. A familiar product is a reference for expectations, not authorization to build its entire feature set. |
| Review recovery and admitted work | The change alters a stateful operation's failure or restart behavior | Use the existing state and protocol and operational readiness guidance and a scoped recovery observation. A wording-only change does not trigger a new recovery exercise. |

Question usefulness is unproven until used. After an actual review, retain what caught an
omission or clarified a decision; revise or retire repetitive or unhelpful questions in their
existing authority. Do not claim reduced bugs or faster delivery without comparative evidence.

For recommendation feedback, distinguish receiving a correction from retaining and using it
in later recommendations. Only the latter supports a claim that recommendations improve.
For capability claims, identify intended outcome, implemented behavior, actual user or agent
access route, and presentation separately. Hidden functionality still costs maintenance;
an internal action without usable access does not prove the intended capability is available.

## Provenance and evidence boundaries

The practice of feeding design patterns into operational-readiness checklists, with discernment
about why and when each applies, is adapted from existing public practice rather than being proof
of an installed readiness-review engine:

- [AWS ORR iteration](https://docs.aws.amazon.com/wellarchitected/latest/operational-readiness-reviews/iteration.html) describes workload-specific templates, challenge, feedback and incident-driven curation.
- [Google launch checklist](https://sre.google/sre-book/reliable-product-launches/) describes practical questions and curation against checklist growth.
- Atul Gawande's *The Checklist Manifesto* is the usual reference for do-confirm and read-do checklists.

The workflow above is an adaptation. Neither these sources nor healthcare checklist results
establish its causal benefit for coding agents.
