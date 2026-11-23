# Two independent models converging on one design

Used between two different models (one writes the first design, the other reviews and
co-designs), ideally on different machines. Three rounds; one agreed document. What worked is
below; what cost time is at the end.

## Roles

- **Author**: writes the first design from a brief, read-only, labelling every claim Confirmed
  (with `path:line` or URL), Likely or Unknown, and listing the probes it could not run.
- **Reviewer**: a different model, on a different machine when the machines differ (each can
  probe what only its machine can). Gets the author's artifact, the original brief, and the
  requester's verbatim words, separately.
- **Human**: decides only what evidence cannot settle, after three rounds.

## The reviewer's first pass, before any message

1. Re-verify every Confirmed citation against current code (delegate the mechanical part to a
   cheaper model with the exact list; read the deciding lines yourself for the claims that drive
   the design). Record TRUE / PARTLY / FALSE with the current `path:line`.
2. Fetch the canonical pages for every documentation claim; label anything that turned out to be
   a third-party write-up. Keep "could not fetch" explicit.
3. Run the harmless probes on your machine that the author could not; ask the author for the ones
   only its machine can run (signed-in provider, device, app).
4. Measure the numbers the design rests on (restart time, per-call cost) on the real system.
5. Load the relevant skills and any reading the requester has collected; note which changed your view.
6. Write your findings to a file before sending anything.

## The exchange

- Message 1: who you are, what you have read, the probes you want, the first challenges with the
  evidence behind each, numbered. Ask for partial replies per batch so probe results arrive as
  they come.
- Each later message: fold the other's reply explicitly ("we agree on…; I still contest…;
  new fact…"); never re-argue a settled point; attach the integrated document as a file rather
  than describing it.
- Final message: the integrated document, a request for "agreed as written" or exact contested
  passages with evidence.
- Every message carries a stable id; the reply closes the request with a typed disposition.

## Rules that kept it honest

- A disagreement that evidence cannot settle is written into the document verbatim with each
  side's evidence under each name. Never averaged, never dropped.
- Attribution is checked: when the author attributes a proposal to the reviewer that the reviewer
  did not make, the reviewer says so and the author retracts it in writing.
- A measured fact beats a cited type declaration beats a documentation page beats a blog.
- Scope stays the requester's: the reviewer may not add a subsystem the request did not name; a useful
  idea goes in the changelog as future work.
- Three rounds. If the third still disagrees, the human decides among named alternatives.

## What cost time, and the fix

- A "Confirmed" claim can come from a stale checkout (read at commit N while main was at N+1).
  Fix: the reviewer states the exact commit it verified against in the first message; the author
  re-reads at that commit.
- Three documentation claims were third-party write-ups surfaced by web search as "official".
  Fix: fetch the canonical page; cite pinned source commits for behaviour claims.
- A probe can be blocked by a command filter on one machine and pass on the other. Fix: record
  the refusal as evidence, rerun with a command the filter does not match, and say so.
- The design document was only drafted after all inputs; a skeleton with marked gaps after the
  first round would have let the author read the integrated structure a round earlier.
