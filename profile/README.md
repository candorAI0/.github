# Candor

**Process-based AI accountability for college writing assignments.**

Candor lets professors permit AI use in student work and see how it was used, instead of banning it and hoping. It records the writing process rather than guessing, after the fact, whether a finished piece of text looks AI-written.

---

## The problem

Students are already using AI. Colleges have two responses, and both fail:

- **Ban it.** Bans are unenforceable. Detection of AI-written prose is unreliable, so a ban punishes the honest and the careless while missing everyone else.
- **Ignore it.** The assignment stops measuring anything, and nobody says so out loud.

Underneath both sits a framing problem. Academic integrity policy treats AI use as cheating. In the professional world students are heading into, using AI well is expected and rewarded.

The result is a system that teaches students to hide their AI use, then sends them into jobs that ask them to showcase it. Candor closes that gap.

## Why detection fails

Tools that classify finished text as human or machine fail for structural reasons:

| Problem | Consequence |
|---|---|
| Classifiers are probabilistic | False accusations against real student writing |
| Known bias against non-native English writers | The students least able to contest it are hit hardest |
| Trivially defeated by paraphrasing | Sophisticated cheaters pass; clumsy honest students fail |
| Adversarial arms race | Every model release resets the detector's accuracy |
| Binary output | "87% AI" tells a professor nothing about how AI was used |

Detection also answers the wrong question. "Was AI used?" is the wrong question when the modern answer for most assignments should be yes, with permission. The useful question is how, and the final text alone cannot answer it.

## What Candor does instead

Candor records the process rather than analyzing the output.

Students write inside a dedicated desktop app with an AI assistant built into the window. As they work, the app captures:

- Keystrokes and draft evolution, timestamped
- Paste events, including the size of what was pasted
- Every AI prompt and every AI response, in full
- Time on task, gaps, and revision patterns

Professors review the finished work alongside the record of how it came to exist. That record is a sequence of events, not a score and not an accusation.

This turns an unanswerable forensic question into a factual one.

## How it works

**The professor sets the rules, per assignment.** AI policy is defined at the assignment level rather than the institution level, because the right policy depends on what the course teaches. Ten policies are available, grouped by how far Candor can stand behind them.

*Enforced. The window or the proxy prevents it.*

| Policy | What it does |
|---|---|
| `banned` | The AI pane never opens |
| `locked-until-draft` | The pane stays shut until the student has written a set number of their own words |
| `turn-limited` | A fixed number of exchanges, set per assignment |
| `socratic-only` | The assistant may only ask questions. It never produces prose |
| `mechanics-only` | Grammar, spelling, and punctuation only. No content and no structure |

*Observed. Recorded and flagged for a person to judge.*

| Policy | What it does |
|---|---|
| `allowed-brainstorm-only` | AI for outlining and idea generation, not for drafting final prose |
| `allowed-revision-only` | The assistant critiques the student's own draft rather than replacing it |
| `no-ai-prose` | Talk freely, but no AI text may enter the document |

*Procedural. Adds a requirement to the submission.*

| Policy | What it does |
|---|---|
| `allowed-with-log` | Use it freely, and the conversation forms part of the submission |
| `allowed-with-reflection` | Use it freely, and submit a short account of how it was used |

Two are worth singling out. `no-ai-prose` is the rule Candor can check mechanically, by comparing the assistant's responses against the text of the document. `locked-until-draft` is the one no detector can express at all: it makes the student think first and use AI second.

**The student writes in the app.** A distraction-limited editor sits on one side and an AI chat pane on the other. The assignment's AI policy is enforced in the app, and the student sees exactly what is being recorded.

**The professor reviews in the dashboard.** Each submission shows the final text, a compressed timeline of the writing process, the complete AI conversation log, and any flagged events.

**A flag is a question, not a verdict.** A large paste might be a student moving a draft in from another editor. Candor surfaces the event and leaves the judgment to a person. The system makes no claim about intent.

## Design principles

**Transparency over surveillance.** Students see their own process log. Everything captured is visible to the person it came from.

**Bounded capture.** Recording happens inside the app, on the assignment, while the student works on it. Capture covers keystrokes, paste events, and AI messages within the assignment window. It excludes webcam, microphone, screen recording, and activity outside the assignment.

**Support the intended pedagogy.** Professors decide what good AI use looks like in their course. Candor gives them the visibility to make that decision real.

**Teach AI as a tool for thinking.** The goal is graduates who can articulate how they used AI and why, which is what employers ask for.

## Architecture

Four repositories, TypeScript throughout. Private during development.

```
                    ┌──────────────────────┐
                    │    shared-types      │
                    │  User, Assignment,   │
                    │  Submission, Event   │
                    └──────────┬───────────┘
                               │ imported by all
          ┌────────────────────┼────────────────────┐
          │                    │                    │
┌─────────▼──────────┐  ┌──────▼───────┐  ┌─────────▼──────────┐
│ lockdown-browser   │  │ backend-api  │  │ professor-         │
│      -app          │─▶│              │◀─│   dashboard        │
│                    │  │   Express    │  │                    │
│  Electron desktop  │  │   Postgres   │  │  Next.js web app   │
│  Student writes    │  │   AI proxy   │  │  Professor reviews │
│  Captures events   │  │              │  │  Sets policy       │
└────────────────────┘  └──────┬───────┘  └────────────────────┘
                               │
                     ┌─────────▼──────────┐
                     │  Anthropic /       │
                     │  OpenAI API        │
                     │  (keys server-side)│
                     └────────────────────┘
```

| Repo | Stack | Role |
|---|---|---|
| `shared-types` | TypeScript | Single source of truth for core data shapes |
| `backend-api` | Node, Express, Postgres | Auth, event storage, AI proxying |
| `professor-dashboard` | Next.js | Assignment creation, policy config, submission review |
| `lockdown-browser-app` | Electron, TipTap | Student writing environment and capture |

Two deliberate architectural choices:

**AI keys stay off client devices.** The desktop app reaches Anthropic or OpenAI through the backend, which holds the keys server-side.

**Candor is a wrapper, not a model.** The product is the capture layer, the policy layer, and the review layer. The intelligence comes from Anthropic or OpenAI.

## Market

Two ladders run at once. One moves between institutions, the other between disciplines. The product stays the same. The assignments are what change.

**By institution**

| Segment | Why in this order |
|---|---|
| **Universities, first** | Faculty choose their own tools, budgets sit at department level, and students are adults, so consent and data handling stay comparatively simple. This is where the product gets proven. |
| **K-12, next** | Larger seat counts and district-level buying. Minors change the picture: parental consent, COPPA, and state student-privacy law all apply, so this comes after the university product is settled. The study cited above already covers this segment. |

**By discipline**

| Stage | Courses | Why |
|---|---|---|
| I | Writing and philosophy (ENG, PHIL) | The assignments are entirely text, and "how was this reasoned?" is not a side concern, it is the thing being assessed. |
| II | Business and marketing (BUS, MKTG) | Case studies, marketing plans, and memos. The same structure, in a context where professional AI fluency already carries explicit value. |
| III | Computer science | The long-term prize, and the hardest. Assistance is already normal there, so the question shifts from whether AI was used to whether the student can still reason about what it produced. |

The wedge: writing faculty absorb the most damage from AI today and have the fewest workable responses available. Every other segment is reached with the same product and a different assignment type.

## Status

In active development. MVP scope is read-only review, meaning Candor captures the process and lets professors look at it. Enforcement, scoring, and intent inference stay out of scope for now.
