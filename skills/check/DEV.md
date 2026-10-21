# write:check — development state

Working notes for whoever picks this up next. Not part of the skill; the skill
does not read this file.

## Where it is

A three-pass loop — draft, blind detect, informed repair, re-check — running
inside Claude Code. The two passes are the plugin's agents,
`write:clarity-detector` and `write:clarity-repair`, dispatched with
the Agent tool. Both default to `sonnet` with no effort set.

The detector is measured against nine fixtures drawn from real artifacts, with
the human verdict each received recorded in `fixtures/expectations.json`.
Names, companies and ticket ids in them are stand-ins; `harvest-fixture`
adds new fixtures the same way.
Seven are cases it must flag; two are controls it must largely pass.

```
scripts/run-fixtures -n 3                 # sonnet, default effort
scripts/run-fixtures -n 3 -m opus -e high # any model and effort
scripts/score-fixtures                    # newest run; or pass a directory
```

`run-fixtures` runs the detector as `claude -p --tools ""` from an empty
directory, so in the benchmark it is blind by enforcement. In the skill it is
blind only by instruction — the registered agents have every tool.

Before this was a plugin the passes ran under opencode on
`microsoft-foundry/gpt-5.6-sol`, chosen as a non-Anthropic model so the checker
would not share the drafting model's habits. The 79% / 86% recall figures from
that era were measured on that model and do not carry over.

## Where it should be

The end state is that Mike stops correcting agent prose before it can be sent.

1. **Compression covered end to end.** Detection and repair work; not yet
   proven in day-to-day use.
2. **Padding, form, and task identification** — specified, not built. Build
   them only after real use shows they matter more than the known limits below.
3. **Invoked without being remembered.** The intended home is inside the skills
   Mike already invokes by name, sequenced after the standalone loop has earned
   it.

## Known limits

Measured 2026-10-15 on Sonnet 5.5 and Opus 5.5, three runs each at medium and
high effort.

**Git vocabulary passes as resolved.** "the three-dot diff" in
`01-slack-413-rejected` is one of the three phrases Mike rejected by hand.
Opus at both efforts and Sonnet at high missed it 3/3; Sonnet at medium caught
it 2/3. gpt-5.6-sol caught it 3/3. The likely cause is that a Claude reader
knows the git term and treats it as resolved — the shared-habits risk the
original model choice guarded against.

**The human-written control draws findings.** `02-slack-413-sent` was sent and
worked. Every Claude configuration flags `migration 0001` there, which the
expectations forbid, and produces 2–4 findings per run where gpt-5.6-sol mostly
produced none.

**Output varies run to run.** Total findings across the nine fixtures ranged
59–73 per run. Only the intersection across runs is signal.

**A term cannot be distinguished from a quotation of that term.** A document
about unclear writing scores badly for the wrong reason, and the repair pass
will expand a quoted example into its definition. Naming the quotation in the
evidence block — "these are quoted as examples; leave them exactly as written"
— makes repair leave them alone. A workaround, since the author has to know
which spans are quotations.

## Open questions

- **Does three-dot need a prompt rule or a different model?** A line in the
  detector saying git terms are unresolved unless the reader sentence says the
  reader uses git that way is the cheap experiment. Re-run the fixtures to see
  whether it fixes `01` without new false positives.
- **Should the detector run on a non-Anthropic model again?** Only possible
  outside Claude Code's model list. The fixtures say Claude trades one miss
  (three-dot) for two catches (Class A, `#485`); that is not yet a case for
  the extra machinery.
- **Where does the evidence block come from when this runs inside another
  skill?** Nobody has designed that.

## Methodological caveats

The fixtures and the detector prompt were written by the same agent — a
tautology risk. The `require` lists were revised after seeing output, to
accept alternative spans for the same defect. Three runs per configuration
cannot separate configurations whose pass counts differ by one or two.

## Log

Dated observations. Append; do not rewrite.

**2026-10-15** — Benchmarked the detector on Claude after the move to a
plugin. Fixture runs passed, of 27 (nine fixtures × three runs):

```
gpt-5.6-sol (2026-08-05)  19   missed 9   false positives 1
sonnet 5.5, medium        20   missed 4   false positives 5
sonnet 5.5, high          20   missed 5   false positives 2
opus 5.5, medium          21   missed 3   false positives 4
opus 5.5, high            19   missed 6   false positives 3
```

A wash between configurations; Opus and higher effort buy nothing measurable.
Kept `sonnet` at default effort — Opus costs more and is the model drafts are
written on.

Two old known limits were specific to gpt-5.6-sol and are gone on Claude:
**terms defined after first use** ("Class A" in `07-pr45-description`, missed
3/3 on gpt, caught 11/12 on Claude) and **bare identifiers** (`#485` in
`01-slack-413-rejected`, never flagged on gpt, flagged 2/3 on every Claude
configuration). Surface-dependent blindness for "three-dot" flipped: Claude
catches it in the vault note and misses it in the Slack draft, the reverse of
gpt.

`run-fixtures` and `score-fixtures` were rewritten for Claude Code in the same
change; the opencode versions and `write-loop` are retired.

**2026-10-21** — Replaced identifiers in the fixtures with stand-ins, so the
plugin can be public: people, the company and its repository, and ticket ids.
Fixture 06 was renamed to match. Dropped the annotated Slack transcript in
`fixtures/corpus/`, which no fixture or script read. Same detector, Sonnet at default effort, three runs each:

```
original fixtures     23/27   262 findings
stand-in fixtures     23/27   246 findings
```

The 3/3 failures are the same (`migration 0001` and `the hash` in
`02-slack-413-sent`). The one-run failures moved: `three-dot` in
`08-vault-note` before, `Class A` in `07-pr45-description` after. Stand-ins
drew findings at the originals' rates (`acme/platform` 3/3, `PLAT-4841` 3/3),
and no run flagged a person's name. `results/` holds the stand-in run.
