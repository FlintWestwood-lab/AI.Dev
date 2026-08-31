# Issue tracker

This directory **is** the issue tracker for this repo. It is committed, not
ignored, despite the name.

Layout, written by `/to-spec`, `/to-tickets`, `/triage` and `/wayfinder`:

```
.scratch/
└── <feature-slug>/
    ├── spec.md                    ← /to-spec
    └── issues/
        ├── 01-<slug>.md           ← /to-tickets, numbered from 01
        └── 02-<slug>.md
```

Each issue file carries a `Status:` line near the top holding one of the roles
from `docs/agents/triage-labels.md`, a `Blocked by: NN, NN` line when it has
blocking edges, and a `## Comments` heading at the bottom where conversation
appends.

`/wayfinder` adds a `map.md` alongside `issues/` for the effort it is charting.

## Switching to GitHub or Linear later

Rewrite `docs/agents/issue-tracker.md`, or re-run
`/setup-matt-pocock-skills` and pick a different tracker. Nothing else in the
repo needs to change: every skill reads that one file to decide whether to run
`gh issue create` or write a file here.
