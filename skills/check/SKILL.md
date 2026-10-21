---
name: check
description: >-
  Check and repair prose that leaves the session — Slack messages, Linear issues
  and comments, pull request descriptions, pull request review comments, vault
  notes. A blind pass finds references the reader cannot resolve; an informed
  pass replaces them and returns a change log. Use when the user says
  "/write:check", "check this before I send it", "is this clear", "draft this",
  "write this up", or when a draft has been rejected as unclear. Do NOT use for
  code comments or for prose that stays inside the current session.
user-invocable: true
---

# Write check

Mike delegates writing to agents. The recurring cost is that he has to correct
the result before it can be sent. This skill removes that correction.

**Deliver a repaired draft, not a list of findings.** The name says "check"
because that is what Mike reaches for, but stopping at the findings hands the
correction back to him, which is the cost this exists to remove. A finding says
"the 0001 edit is unclear". The deliverable says "the five-line comment that
pull request 413 in `acme/api` added to
`db/migrations/0001_foundation.sql`". Run
the loop to the end and hand over prose plus a change log.

There is one case where findings alone are the right output: you do not have the
facts behind a flagged reference and nobody has given them to you. Then say which
facts you need. Never guess — a wrong expansion reads as authoritative.

## Why a separate detector exists

During a long session an agent builds short labels for things it has been
reasoning about. "The five-line comment that pull request 413 in `acme/api` added
to `db/migrations/0001_foundation.sql`" becomes "the 0001 edit". To the
agent that label now reads as an ordinary noun. It then writes "the 0001 edit"
to someone who was never in the session.

**The agent cannot catch this by re-reading its own draft.** The label resolves
from inside its context, so any self-check passes. On 2026-08-05 an agent wrote
the phrase "working-state compression" inside a paragraph apologising for
inventing shorthand. Self-inspection is not available.

So the check runs in a subagent with no tools, which is stuck exactly where the
real reader would be stuck. An agent that could search the repository would look
up "the 0001 edit", understand it, and pass the draft.

## Before drafting

Answer these. If you cannot, ask — once, before writing, not midway through.

**What is the objective?** One sentence: what this writing is for and what
changes once it lands. A request for a message is sometimes a request to think,
using a message as the container. If the decision the writing announces is
unsettled, say so and name the open question instead of drafting. Drafting
against an undecided plan turns every draft into rework.

**What does the reader want?** What are they trying to do when they open this —
approve or reject, unblock themselves, decide whether to reply?

**What does the writer want back?** A decision, a handoff accepted, a record
that still holds up in six months.

Most failures are a mismatch between the last two: writing to persuade a reader
who only needed to be informed, or writing a record for a reader who needed to
act.

**What is the interaction contract?** Mike usually states it — *send this, don't
check in first*, or *draft this, then show me*. Follow it exactly. Checking in
against an explicit instruction not to check in is a failure, not caution. If it
is not stated, infer it when you can say what he will do with the output the
moment he gets it, and ask once when you cannot.

## The loop

Three passes on three different footings.

1. **Draft.** You, in this session, with all the context.
2. **Detect** — `clarity-detector`, on a different model from the one drafting, no
   tools, no evidence. It gets the draft and one sentence describing the reader, and
   returns the spans it cannot resolve. It has to be blind: an agent that can
   look a reference up resolves it and passes the draft.
3. **Repair** — `clarity-repair`, same model choice, no tools. It gets the draft, the
   findings, and an evidence block you write. It returns the corrected draft
   plus a change log. It has to be informed, because the detector never knew
   what the terms meant.
4. **Re-check.** Run the detector again on the repaired draft.

Detection blind, repair informed. That split is why this is three passes rather
than two: under-explaining is only detectable without the evidence and only
fixable with it.

**Writing the evidence block is your job and it is the load-bearing part.** The
detector names spans; you supply what each one actually referred to. If you
cannot supply a fact, leave it out — the repair pass will mark it `UNRESOLVED`
rather than guess, which is the correct outcome.

### Repair may restructure

Naming a thing costs more words than labelling it, so a sentence carrying three
or four expansions becomes unreadable — an unresolvable reference traded for a
sixty-word sentence, which is the same defect wearing different clothes. Repair
is therefore allowed to split sentences, break paragraphs into lines, and add a
lead-in phrase such as "Exact steps:". It may not add content, change a claim,
reorder the argument, or shift register.

It reads the `READER` line for shape, not only for vocabulary. "Reads it once,
on a phone" is an instruction about line length. Each restructuring gets its own
line in the change log, so the reviewer can see that shape changed as well as
words.

### Writing the reader sentence

The detector's judgement is only as good as this sentence, because what counts
as resolvable is scaled to the reader. Say who they are, what they already own,
and what they do not have. "A colleague on the deployment team who has GitHub
access but was not in this session" produces a different and more useful result
than "a colleague".

The sentence also carries the artifact's shape constraints, because the repair
pass reads it for shape and will otherwise restructure freely. A resume that
must stay at two pages needs: "each bullet must remain a single compact bullet;
expansions are the fewest words that carry the referent; no bullet may grow by
more than roughly fifteen words; do not split bullets." Without that, the repair
turns a bullet into three and the page count with it.

### Structured sources

The detector reads prose. When the artifact is generated from structured data —
a resume rendered from YAML, a report from JSON — flatten it to the prose the
reader will see (headings, bullets, skills lines), run the loop on that, then
apply the accepted repairs back to the source by hand and re-render. Do not
feed the detector the YAML: keys and quoting produce findings about the format.

### Applying the repair

The change log is the reviewable artifact, and it has to be reviewed against
the source of truth, not just against the draft. Two things to check per line:

- **The repair copies the evidence's wording.** If the evidence says "code
  output" where the source says "code velocity", the repaired draft now says
  "code output". Write the evidence block in the source's own terms.
- **The re-check can surface invented facts.** An expansion the detector still
  cannot resolve is sometimes unresolvable because it was never true: on
  2026-08-28 the re-check flagged "a two-person team operates no
  infrastructure" in a resume whose source said only "the team". Treat a
  re-check finding on a span you wrote as a prompt to look it up in the source.

## Running the loop

The two agents ship in this plugin and register as the subagent types
`write:clarity-detector` and `write:clarity-repair`. Each pass is
one dispatch with the Agent tool.

**Model.** The checker must not be the model that wrote the draft — that is
the one property the model choice protects. Both agents default to `sonnet` in
their frontmatter. If the session is drafting on Sonnet, override the model on
the dispatch.

**Blindness is by instruction, not enforcement.** The agents are registered
with all tools. Open every dispatch with "Do not use any tools. Do not read
files, search, or browse. You are standing in for a reader who has only the
text below." and put nothing in the prompt except the reader sentence and the
draft. A detector that can read
the repository will resolve the reference and pass the draft.

**Dispatch shape.**

1. Detector: `subagent_type: write:clarity-detector`, prompt = no-tools line, then
   `READER: <sentence>`, then `# DRAFT` and the draft verbatim.
2. Write the evidence block from the source of truth.
3. Repair: `subagent_type: write:clarity-repair`, prompt = no-tools line, then
   `READER:`, `FINDINGS:` (the detector's output verbatim), `EVIDENCE:`, then
   `# DRAFT`.
4. Re-check: detector again on the repaired draft. Expect roughly a third of
   the first pass's count; what remains is either as precise as the source
   allows or a fact to verify.

If the registered agent fails to launch, fall back to a `general-purpose`
subagent with a model override and the agent body inlined in the prompt — the
body is the instruction set; the frontmatter is only routing.

## Rules the repair must respect

**Length is not a criterion.** Ten words is fine if ten words do it. The
criteria are precision, word choice, and readability. The test runs both ways —
take a word out and ask whether the reader still gets what they came for, put
one in and ask whether it earned its place.

The correction for an unresolvable reference is almost always longer than what
it replaces. In the case that
motivated this skill, the human-written message was roughly four times the
length of the rejected draft and described the same event. Never treat brevity as evidence of quality.

**Show, don't tell.** When the claim is about a piece of writing, quote the
writing. A reader who cannot see the text has to take your word for it and
learns nothing.

**Content follows the surface.** What a document is for determines what may be
compressed. A handoff message compresses the derivation and keeps the plan. A
corpus for training a writing skill keeps the derivation and compresses nothing.
Before writing, ask where this will be placed and what a reader in that
environment needs.

Specific rules Mike has given, all still standing:

- Linear is permanent and company-wide. No references to his vault or notes, no
  local file paths or worktree names, no agent session provenance, no history of
  previously-held incorrect beliefs. Only what a new person needs to work the
  ticket.
- Vault notes record what currently exists and the traps a new session will hit
   — not a history of things that turned out to be wrong.
- Hyperlink every reference that has a canonical URL. A bare `PROJ-1234` or `#57`
  is a defect.

## Not built yet

**The form check** — "does this content belong in this artifact?" — is a
separate check needing the artifact's rules rather than the reader's ignorance.

**The objective check** — refusing to draft against an unsettled decision — has
no test case. Capturing one requires a live session where the correct output is a
question rather than prose.
