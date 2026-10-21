---
name: harvest-fixture
description: >-
  Turn a piece of writing that needed correcting into a fixture for the
  write:check detector: the draft as it was, the reader, Mike's verdict in his
  own words, and what the detector must or must not flag. Names, ticket ids and
  other identifiers are replaced consistently before anything is saved. Use
  when the user says "/write:harvest-fixture", "make this a fixture", "harvest
  this", or "save this as a test case", or after Mike rejects or rewrites a
  draft and wants the checker to learn from it.
user-invocable: true
---

# Harvest a fixture

The detector in `write:check` is only as good as the fixtures it is measured
against, and the best fixtures are real drafts that a real reader could not
follow. This skill captures one while the evidence is still in the session:
the draft before, the verdict, and, when there is one, the version that worked.

## 1. Collect the evidence, verbatim

- **The draft as it was rejected.** Copy it from the session or from what Mike
  pastes. Never reconstruct it from memory or tidy it. A paraphrased draft
  tests nothing. If you cannot get the exact text, stop and say so.
- **The reader.** One sentence: who they are, what they already own, what they
  do not have, and the surface constraints ("reads it once, on a phone"). The
  detector's judgement scales to this sentence, so write it as `write:check`
  describes.
- **The verdict, in Mike's words.** Quote his correction exactly, with the
  date. The phrases he named are what the fixture tests.
- **The version that worked**, if it exists: the message that was sent, the
  ticket that was accepted. It can become a second fixture, a control.

## 2. Decide what the fixture tests

- **A case** — a draft that failed. Each phrase Mike named becomes a `require`
  entry, written as a list of the ways the detector might quote it
  (`["the 0001 edit", "0001 edit"]`), because the test is whether it noticed
  the reference, not how it quoted it.
- **A control** — writing that worked. Spans that resolve for this reader go in
  `forbid`. Real ambiguity the surface does not need fixed goes in `tolerable`.

Anything flagged that is in none of the lists shows up as unclassified when
scored, so leave a list empty rather than guessing.

## 3. Replace identifiers

The fixtures are public. Before writing any file, list every identifier in the
draft, the reader sentence and the verdict:

- people's names and handles, except Mike's own
- companies, private repository names, and URLs into private systems
- ticket ids, local paths, session ids, email addresses

The map lives at `~/.config/write-plugin/pseudonyms.json` (or
`$WRITE_PSEUDONYMS`). It is the key that undoes the substitution, so it never
goes into a repository. Read it, and reuse an existing entry for anyone or
anything already in it. That is what keeps one person the same stand-in across
every fixture.

For each new identifier, propose a stand-in of the same shape: a full name for
a full name, with the same pronouns the draft uses; a ticket keeps its number
and changes its prefix, written as a `pattern` entry so later tickets from the
same tracker are covered too. Show Mike the new entries as a table and wait
for his approval. Then add them to the map.

If nothing needs replacing, say so and let Mike confirm.

## 4. Write the fixture

Work in Mike's checkout of this plugin's repository, `Jollyhrothgar/write-plugin`.
Its path is in `~/.config/write-plugin/checkout`. If that file is missing, ask
once and write the answer there. Create a branch.

In `skills/check/fixtures/`:

- `NN-<surface>-<slug>.prompt.txt`, numbered after the highest existing
  fixture, in the existing format:

  ```
  READER: <sentence>

  DRAFT:
  <draft verbatim>
  ```

- An entry in `expectations.json` with `require`, `forbid`, `tolerable` and a
  `verdict` that quotes Mike and gives the date.

Then replace identifiers and confirm nothing is left:

```bash
P="${CLAUDE_PLUGIN_ROOT}/skills/harvest-fixture/scripts/pseudonymize"
python3 -I "$P" apply --dry-run skills/check/fixtures   # read this first
python3 -I "$P" apply skills/check/fixtures
python3 -I "$P" check .                                 # must print "clean"
```

`check` matches without regard to case and also reads file names. Do not
commit while it reports anything.

## 5. Run it

Run the new fixture three times. Output varies between runs, so one run
proves nothing:

```bash
T="$(mktemp -d)"
cp skills/check/fixtures/NN-*.prompt.txt "$T/"
python3 -I -c 'import json,sys; e=json.load(open(sys.argv[1])); json.dump({sys.argv[2]: e[sys.argv[2]]}, open(sys.argv[3], "w"), indent=1)' \
  skills/check/fixtures/expectations.json NN-<surface>-<slug> "$T/expectations.json"
WRITE_CHECK_FIXTURES="$T" skills/check/scripts/run-fixtures -n 3 -o "$T/results"
WRITE_CHECK_FIXTURES="$T" skills/check/scripts/score-fixtures "$T/results"/*
```

A case the detector already catches every time still belongs in the suite,
because it guards against regressions. A case it misses is the valuable kind;
say so plainly.

## 6. Land it

Append a dated line to the Log in `skills/check/DEV.md`: which fixture, what it
tests, and how the three runs scored. Commit, push the branch, and open a pull
request. Run `pseudonymize check .` once more before pushing.

## Stop conditions

- The draft cannot be recovered verbatim.
- Mike has not approved the new map entries.
- `pseudonymize check` reports anything.
