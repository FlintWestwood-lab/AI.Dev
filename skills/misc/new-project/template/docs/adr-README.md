# Architecture Decision Records

One file per hard-to-reverse decision, numbered from `0001`, named
`NNNN-short-slug.md`.

Written by `/domain-modeling` (usually reached through `/grill-with-docs`) at
the moment a decision is actually made, not upfront. If this directory is
empty, that is the correct starting state.

The shape:

```markdown
# NNNN. Title in the present tense

## Status

Accepted | Superseded by ADR-NNNN

## Context

What forced a decision. The constraints as they stood at the time.

## Decision

What was chosen, stated plainly.

## Consequences

What this makes easy, and what it makes hard or expensive to undo.
```

A decision earns an ADR when reversing it later would be expensive: a database
choice, an auth model, a boundary between modules. Routine choices do not.
