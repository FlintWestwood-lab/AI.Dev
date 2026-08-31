# Idea to ship

The abstract flow. It does not know or care what this app is, so it works on
the first feature and the hundredth.

Every step is a skill from the `mattpocock-skills` plugin. If you forget the
map, type `/ask-matt` and it will route you.

---

## Step 0, once per repo

Already done for you. `CLAUDE.md`, `CONTEXT.md`, `docs/agents/` and `.scratch/`
are in place, which is exactly what `/setup-matt-pocock-skills` writes.

Re-run `/setup-matt-pocock-skills` only if you want to switch issue trackers
(this template is set to local markdown) or start the config over.

---

## The main flow

Keep steps 1 to 3 in **one unbroken context window**. Do not `/clear` or
`/compact` until after `/to-tickets`, so the interview, the spec and the
tickets are all built on the same thinking.

### 1. Sharpen the idea

```
/grill-with-docs
```

A relentless interview. It works your idea as a design tree, asking every
question whose prerequisites are settled in one numbered round, each with a
recommended answer, then waits for you. Facts are its job, decisions are yours.

It stops when the frontier is empty: every branch visited, nothing silently
assumed. Along the way it writes terms into `CONTEXT.md` and hard-to-reverse
decisions into `docs/adr/`.

Use it **every time** you want to make a change, not just big ones.

> Not in a working directory? `/grill-me` is the same interview with no paper
> trail. Inside this repo, `/grill-with-docs` is strictly better.

### 2. Does a question need a runnable answer?

If a question is about a state model, a piece of business logic, or a UI you
have to actually see, do not settle it on paper. Detour:

```
/handoff          # write a portable file describing the question
                  # open a fresh session in a prototype directory
/prototype        # throwaway code that answers exactly that question
/handoff          # bring the answer back, reference it from the idea thread
```

Throwaway is a rule about how the code is written, not a promise to delete it.
Keep the prototype on a `prototype/<name>` branch as a primary source, and
point at it from the issue.

### 3. Is this more than one session of work?

**No.** Stay here:

```
/implement
```

**Yes.** Split it:

```
/to-spec          # collapse this conversation into a spec, no new interview
/to-tickets       # break the spec into tracer-bullet tickets with blocking edges
```

Locally that writes `.scratch/<feature>/spec.md` plus one file per ticket under
`.scratch/<feature>/issues/`. Work blockers-first.

Then, **one ticket per session**:

```
/implement        # then /clear before the next ticket
```

Each ticket is self-contained, so the previous one's context is disposable.
That `/clear` is the whole point of splitting.

### What `/implement` does

It drives `/tdd` internally at seams you agreed up front, runs typechecking and
single test files as it goes, runs the full suite once at the end, then closes
out with `/code-review` before committing.

`/tdd`'s rules, worth knowing because they constrain what it will accept:

- Red before green. Failing test first, then only enough code to pass it.
- One vertical slice at a time. One seam, one test, one implementation.
- No test at an unconfirmed seam. It will ask you which seams first.
- Refactoring is not in the loop. It belongs to review.

---

## On-ramps

Work that arrives from somewhere else, then merges onto the main flow.

**Bugs and requests piling up** → `/triage`. Moves issues through the five
roles in `docs/agents/triage-labels.md` and produces agent-ready briefs that
`/implement` later picks up.

> Only for issues **you did not create**. Tickets from `/to-tickets` are
> already agent-ready. Do not triage them.

**Something is broken** → `/diagnosing-bugs`. For the hard ones: the
intermittent flake, the regression that crept in between two known-good states.
It refuses to theorise until it has a tight feedback loop that already goes red
on *this* bug. Build the loop and the bug is 90 percent fixed.

**A greenfield build or a feature too big to hold in one session** →
`/wayfinder`. Charts a map of decision tickets and resolves them one at a time
until the fog clears. It produces **decisions, not deliverables**, then hands
off to `/to-spec`. The densest flow in the set, so do not reach for it on a
well-scoped feature.

---

## Upkeep

```
/improve-codebase-architecture
```

Run it every few days. It surveys for deepening opportunities and hands you
candidates as an HTML report. Picking one generates an idea you take back into
`/grill-with-docs` at step 1.

It is a survey, not a rescue. It finds the candidates; it will not untangle the
mud for you.

---

## Reach for these any time

| When | Skill |
|---|---|
| A message did not land. Fire it immediately | `/wait-what` |
| The blocker is in someone else's head | `/to-questionnaire` |
| You need reading legwork, cited, in the background | `/research` |
| Only a human can do the next step (credentials, dashboards, cutover) | `/wizard` |
| You are mid merge or rebase conflict | `/resolving-merge-conflicts` |
| The words are wrong, not the process | `/domain-modeling` |
| A module's shape is the question | `/codebase-design` |
| You want to learn something over several sessions | `/teach` |
| You are writing a skill or an AGENTS.md | `/writing-for-agents` |

---

## At a phase boundary

Between the interview and the build, or the build and the QA, you have five
options. Ruled out first, not last: **Continue**, because staying put costs
nothing and loses nothing.

- **Continue**: nothing to pay.
- **`/clear`**: nothing here matters to what is next.
- **`/handoff`**: only for a new harness, a new directory, a colleague, or
  forking a side task mid-phase. What it buys is portability.
- **Subagent**: a tightly-scoped task in its own window, reporting back.
- **`/compact`**: the default, at the bottom of the tree rather than the first
  reach.

Make the call **at** a boundary. Mid-phase, either continue or split the
remainder into subagents.

Watch the **smart zone**, roughly 150k tokens, past which the model stops
reasoning sharply. If a session approaches it before `/to-tickets`, do not push
on degraded. `/compact` at the nearest boundary and carry on.
