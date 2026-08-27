# Candor

**Process-based AI accountability for college writing assignments.**

Candor lets professors *permit* AI use in student work — and actually see how it was used — instead of banning it and hoping. It records the writing process rather than trying to guess, after the fact, whether a finished piece of text "looks AI-written."

---

## The problem

Students are already using AI. Constantly. Colleges have two responses, and neither works:

- **Ban it.** Unenforceable. There is no reliable way to detect AI-written prose, so the ban punishes the honest and the careless while missing everyone else.
- **Ignore it.** The assignment stops measuring anything, and nobody says so out loud.

Underneath both is a framing problem. Academic integrity policies assume AI use is *cheating*. But in the professional world students are heading into, using AI well is expected, rewarded, and increasingly part of the job description.

So we've built a system where students learn to **hide** their AI use, then graduate into jobs where they're asked to **showcase** it. That gap is the problem Candor exists to close.

## Why detection doesn't work

Tools that classify finished text as human or machine fail for structural reasons, not fixable ones:

| Problem | Consequence |
|---|---|
| Classifiers are probabilistic | False accusations against real student writing |
| Known bias against non-native English writers | The students least able to contest it are hit hardest |
| Trivially defeated by paraphrasing | Sophisticated cheaters pass; clumsy honest students fail |
| Adversarial arms race | Every model release resets the detector's accuracy |
| Binary output | "87% AI" tells a professor nothing about *how* AI was used |

Most importantly: **detection answers the wrong question.** "Was AI used?" isn't useful when the honest answer for most assignments should be "yes, and that's allowed." The useful question is *how* — and no amount of staring at the final text can answer that.

## What Candor does instead

Candor doesn't analyze the output. It records the **process**.

Students write inside a dedicated desktop app with an AI assistant built into the window. As they work, the app captures:

- Keystrokes and draft evolution, timestamped
- Paste events, including the size of what was pasted
- Every AI prompt and every AI response, in full
- Time on task, gaps, and revision patterns

Professors review the finished work *alongside* the record of how it came to exist. Not a score. Not an accusation. The actual sequence of events.

This turns an unanswerable forensic question into a straightforward factual one.

## How it works

**The professor sets the rules, per assignment.** AI policy is defined at the assignment level, not the institution level, because the right policy depends on what's being taught:

- `banned` — no AI assistance
- `allowed-with-log` — use it freely; the conversation is part of the submission
- `allowed-brainstorm-only` — AI for outlining and idea generation, not for drafting final prose

**The student writes in the app.** Distraction-limited editor on one side, AI chat on the other. The assignment's AI policy is enforced in the app, and the student can see exactly what's being recorded — no hidden capture.

**The professor reviews in the dashboard.** Each submission shows the final text, a compressed timeline of how it was written, the complete AI conversation log, and any flagged events.

**A flag is a question, not a verdict.** A large paste might be a student moving their own draft in from another editor. Candor surfaces it and lets a human decide. The system never renders a judgment about intent.

## Design principles

**Transparency over surveillance.** Students see their own process log. Everything captured is visible to the person it was captured from.

**Bounded capture.** Recording happens inside the app, on the assignment, while the student is working on it. Candor is not a proctoring tool: no webcam, no microphone, no screen recording, no monitoring of anything outside the assignment.

**Support the intended pedagogy, don't invent one.** Professors decide what good AI use looks like in their course. Candor gives them the visibility to make that decision real.

**Teach AI as a tool for thinking, not a shortcut around it.** The goal isn't catching students. It's producing graduates who can articulate how they used AI and why — which is what employers are asking for.

## Architecture

Four repositories, TypeScript throughout. *(Private during development.)*

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

**Two deliberate architectural choices:**

*AI keys never touch a client device.* The desktop app doesn't call Anthropic or OpenAI directly — it goes through the backend, which holds the keys server-side.

*Candor is a wrapper, not a model.* There is no custom AI here, and no plan for one. The product is the capture layer, the policy layer, and the review layer.

## Market

**Start:** college composition and writing courses. The AI problem is most acute there, the assignments are almost entirely text, and "how was this written?" is the central pedagogical question rather than a side concern.

**Expand:** business and marketing courses — case studies, marketing plans, memos. Same structure, and closer to the professional context where AI fluency is explicitly valued.

**Then:** any writing-heavy discipline.

The wedge is that composition instructors are the faculty currently absorbing the most damage from AI, and have the least workable response available to them.

## Status

🚧 In active development. MVP scope is **read-only review** — capture the process and let professors look at it. No automated enforcement, no scoring, no intent inference.
