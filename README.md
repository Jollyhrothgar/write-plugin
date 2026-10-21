# write

A Claude Code plugin that checks and repairs prose before it leaves the
session. A blind subagent finds references the reader cannot resolve; an
informed one replaces them and returns a change log.

| Skill or agent | What it does |
| --- | --- |
| `/write:check` | Draft, detect, repair and re-check a piece of writing |
| `/write:harvest-fixture` | Turn a draft that needed correcting into a detector fixture, with identifiers replaced |
| `write:clarity-detector` | Blind pass: reports spans the stated reader cannot resolve |
| `write:clarity-repair` | Informed pass: replaces them from an evidence block |

## Install

```bash
claude plugin marketplace add Jollyhrothgar/personal_ai_config
claude plugin install write@jollyhrothgar
```

To try a working copy without installing: `claude --plugin-dir .`

## Fixtures

`skills/check/fixtures/` holds real drafts with the verdict each received.
Measure the detector against them:

```bash
skills/check/scripts/run-fixtures -n 3
skills/check/scripts/score-fixtures
```

`skills/check/DEV.md` records where the detector stands and its known limits.

Names, companies and ticket ids in the fixtures are stand-ins. The map from
real to stand-in lives outside the repository, at
`~/.config/write-plugin/pseudonyms.json`, and `harvest-fixture` reuses it so a
person keeps the same stand-in across fixtures. Before committing:

```bash
python3 -I skills/harvest-fixture/scripts/pseudonymize check .
```
