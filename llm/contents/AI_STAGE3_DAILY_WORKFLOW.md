# AI in Daily Workflow (Stage 3 Deep Dive)

> **Companion to:** [AI_ROADMAP.md](./AI_ROADMAP.md) — this file is the full Stage 3 deep-dive.
> **Prerequisites:** [AI_STAGE1_FOUNDATIONS.md](./AI_STAGE1_FOUNDATIONS.md) and [AI_STAGE2_PROMPT_ENGINEERING.md](./AI_STAGE2_PROMPT_ENGINEERING.md) — the vocabulary and prompt skills you'll apply here.
> **Audience:** Software developers who want AI to be muscle memory across the workday, not a curiosity.
> **Time:** Ongoing — start integrating today, never "complete" it.
> **Approach:** Pick 3 workflows, integrate deeply, ignore the rest until those 3 are habits.

---

## Table of Contents

1. [Why this stage matters (the leverage is real)](#1-why-this-stage-matters-the-leverage-is-real)
2. [The integration mindset: dabbler vs integrator](#2-the-integration-mindset-dabbler-vs-integrator)
3. [Anatomy of an AI-augmented workflow](#3-anatomy-of-an-ai-augmented-workflow)
4. [The IDE: your highest-leverage AI surface](#4-the-ide-your-highest-leverage-ai-surface)
5. [Chat tools: ChatGPT vs Claude vs Gemini](#5-chat-tools-chatgpt-vs-claude-vs-gemini)
6. [Research & summarization workflows](#6-research--summarization-workflows)
7. [Writing workflows (emails, docs, PRDs)](#7-writing-workflows-emails-docs-prds)
8. [Data analysis workflows](#8-data-analysis-workflows)
9. [Meeting capture & knowledge workflows](#9-meeting-capture--knowledge-workflows)
10. [Diagrams, slides, image, and video](#10-diagrams-slides-image-and-video)
11. [The "pick 3" exercise: choose what to integrate](#11-the-pick-3-exercise-choose-what-to-integrate)
12. [Common anti-patterns (the dabbler trap)](#12-common-anti-patterns-the-dabbler-trap)
13. [Hands-on exercises](#13-hands-on-exercises)
14. [Interview questions companies actually ask](#14-interview-questions-companies-actually-ask)
15. [Self-check: are you done with Stage 3?](#15-self-check-are-you-done-with-stage-3)

---

## 1. Why this stage matters (the leverage is real)

Stages 1 and 2 were about *understanding* and *speaking* the language of AI. Stage 3 is about **becoming faster at your existing job**. Not "interesting demos" — measurable time saved every single day.

### The honest numbers

For a typical senior software developer in 2026, a serious Stage-3 integration delivers (rough self-reported ranges from surveys and personal experiments):

| Workflow | Before AI | With AI integrated | Time saved |
|---|---|---|---|
| Writing PR descriptions | 10–15 min | 2–3 min | ~80% |
| First-draft design doc | 4 hours | 1 hour | ~75% |
| Debugging an unfamiliar stack trace | 30–60 min | 5–15 min | ~70% |
| Email triage (50 emails) | 45 min | 10 min | ~75% |
| Meeting notes + action items | 20 min | 1 min | ~95% |
| Researching an unfamiliar topic | 2 hours | 30 min | ~75% |
| Generating boilerplate code | 30 min | 5 min | ~85% |
| Quick code review on a small PR | 20 min | 5 min | ~75% |

Add it up across a week. You're talking about **5–10 hours/week reclaimed** for someone who actually integrates. That's a whole engineering day.

### What companies actually pay for

This stage doesn't show up on a JD as "Stage 3." It shows up indirectly:

- **"Productive with AI tools"** — every JD in 2026 mentions some variant.
- **"Self-directed, high-output"** — code for "we've noticed our AI-leveraged engineers ship 2x what their peers do."
- **Live demos in interviews** — many companies now ask candidates to do a coding/design exercise *with* AI tools, and watch how you use them. Sloppy use is a red flag.
- **Internal AI advocacy** — senior engineers who can mentor teammates on AI workflows are valuable beyond their own output.

### Why this is the hardest stage to actually do

Reading this doc takes an hour. **Building these habits takes months.** The friction isn't intellectual — it's behavioral. You'll keep falling back to your old workflow because it's automatic. The whole game is making the new workflow automatic.

The fix: **pick 3 high-frequency tasks** and ruthlessly route them through AI for 4 weeks. Skip the other 50 things in this doc until those 3 are reflex.

---

## 2. The integration mindset: dabbler vs integrator

There are two kinds of AI users right now. Identify which you currently are. Then close the gap.

```
                  DABBLER                              INTEGRATOR
              ────────────────                     ──────────────────

   Opens ChatGPT when stuck.            ←→        Routes specific recurring tasks
                                                  through AI by default.

   Different tools every week.          ←→        Settled on 3 deep, ignores rest.

   Re-types the same prompt              ←→        Has a prompt library;
   from scratch each time.                         pastes/composes templates.

   Treats AI as a magic 8-ball:         ←→        Treats AI as a junior teammate:
   ask, accept whatever it says.                   verify, push back, iterate.

   No measurement.                      ←→        Knows roughly how much time
                                                   AI saves per task.

   "AI gave me a wrong answer            ←→        "I used AI to draft, then verified
    so I won't use it."                             the 2 things AI commonly gets wrong."

   Demos cool tools to friends.         ←→        Has integrated AI so deeply
                                                   it barely comes up in conversation
                                                   anymore — it's just how they work.
```

The transition is not about learning new tools. It's about **changing default behaviors** for a few specific tasks. Pick one. Make it stick. Then pick the next.

---

## 3. Anatomy of an AI-augmented workflow

Every effective AI-augmented workflow follows the same shape. Internalize it; you'll re-use it everywhere.

```
            ┌─────────────────────────────────────────────────┐
            │  1. TRIGGER                                     │
            │     A real, recurring task you do anyway.       │
            │     (PR description, weekly status, debugging…)  │
            └────────────────────┬────────────────────────────┘
                                 ▼
            ┌─────────────────────────────────────────────────┐
            │  2. CONTEXT GATHER                              │
            │     Pull in: the artifact, the constraints,     │
            │     the audience, prior examples.               │
            └────────────────────┬────────────────────────────┘
                                 ▼
            ┌─────────────────────────────────────────────────┐
            │  3. AI DRAFT                                    │
            │     Use a saved template (from Stage 2 library) │
            │     to generate a first cut. Cheap, fast.       │
            └────────────────────┬────────────────────────────┘
                                 ▼
            ┌─────────────────────────────────────────────────┐
            │  4. HUMAN VERIFY + EDIT                         │
            │     Spot-check facts, fix tone, cut filler.     │
            │     YOU are still the author. AI was the typist. │
            └────────────────────┬────────────────────────────┘
                                 ▼
            ┌─────────────────────────────────────────────────┐
            │  5. SHIP                                        │
            │     Send it / commit it / post it.              │
            └────────────────────┬────────────────────────────┘
                                 ▼
            ┌─────────────────────────────────────────────────┐
            │  6. LEARN (the part most people skip)           │
            │     Did the draft need heavy editing? Update     │
            │     the template. The next iteration is better. │
            └─────────────────────────────────────────────────┘
```

The two phases people botch:

- **Step 2 (context gather)** — they paste the bare question with no context. Garbage in, garbage out. Always give the AI the actual artifact, the audience, the constraints.
- **Step 6 (learn)** — they don't update their template when the draft needed heavy editing. So next month they're doing the same heavy editing again. Treat your prompt library as a living thing.

### The "AI as junior teammate" frame

The single most useful frame I've found: **treat AI like a smart junior teammate.** Specifically:

- You'd give a junior teammate full context, not a one-line question.
- You'd review their work, not blindly ship it.
- You'd push back when their first answer is off.
- You'd give them feedback so they do better next time.
- You wouldn't ask them to do things you can do in 30 seconds yourself.
- You wouldn't ask them anything irreversible without supervision.

Apply this to AI. It's a near-perfect model.

---

## 4. The IDE: your highest-leverage AI surface

You spend most of your day in an editor. AI in your IDE is, by hours-per-week, your single biggest leverage point. Treat it as a first-class skill, not "something that happens."

### The three modes inside modern IDEs

```
                   TAB COMPLETION (Copilot/Cursor inline)
                   ──────────────────────────────────────
   What:   The IDE suggests the next line as you type.
   When:   Mechanical code: boilerplate, types, repetitive patterns.
   Speed:  Zero-friction; happens automatically.
   Risk:   Low individually, but blind acceptance compounds bugs.

                   CHAT (sidebar, with file/repo context)
                   ──────────────────────────────────────
   What:   You ask a question or give an instruction in natural language.
   When:   Explanations, multi-file edits, exploration, refactor planning.
   Speed:  Seconds. Iterative.
   Risk:   Medium. You see the diff before applying.

                   AGENT MODE (multi-step, autonomous)
                   ─────────────────────────────────────
   What:   You give a goal. The agent reads files, writes edits,
           runs commands, iterates.
   When:   Multi-file changes with a clear scope. Boilerplate features.
           Mechanical migrations.
   Speed:  Minutes per task.
   Risk:   High if unsupervised. Always review the diff and run tests.
```

### A pattern for each mode

**Tab completion — let it autocomplete, but never blindly:**

```python
# You type:
def calculate_tax(amount: float, rate: float) ->
# IDE suggests:
def calculate_tax(amount: float, rate: float) -> float:
    return amount * rate
```

Accept. But scan the suggestion *before* hitting Tab. A bad habit is to Tab-Tab-Tab through a function and not realize you accepted a subtle off-by-one.

**Chat — for surgical changes with full understanding:**

```
@file:user_service.py

Refactor the `create_user` function to:
1. Validate the email format using the regex pattern in `utils/validators.py`
2. Hash the password using bcrypt (already imported in `auth.py`)
3. Return a typed Pydantic model instead of a dict
4. Add a docstring explaining the side effects (DB write, audit log)

Don't change the function signature.
```

Notice: explicit file references, clear constraints, a "don't break this" guard. This is Stage 2 prompt engineering applied inside the IDE.

**Agent mode — for "do this whole feature":**

```
GOAL: Add a "soft delete" feature to the User model.

REQUIREMENTS:
- Add a `deleted_at` nullable DateTime column to the User model
- Generate a database migration
- Update the UserRepository:
   - `find_by_id` should ignore deleted users by default, with `include_deleted=False` flag
   - Add `soft_delete(user_id)` method
- Update the API endpoints DELETE /users/<id> to call soft_delete
- Add tests in test_user_repository.py

CONSTRAINTS:
- Don't change any existing test (they should still pass)
- Match the existing code style in the repo
- Don't add new dependencies
```

Notice the structure: clear goal, enumerated requirements, explicit constraints. **Agent mode without constraints is how you lose an afternoon to a runaway agent.** Always set fences.

### Rules files: codify your style once

Cursor, Copilot, Claude Code — they all let you put a "rules" file at the repo root (e.g., `.cursor/rules/*.mdc`, `AGENTS.md`, `CLAUDE.md`). This file is sent with every AI request in that repo. Use it.

```
# Coding Conventions

- Python 3.12. Use type hints everywhere.
- Use `httpx` for HTTP, never `requests`.
- All DB access goes through `app/db.py`. Never import psycopg directly.
- Tests use pytest. Test files live next to the file under test, named `test_<name>.py`.
- Logging: use `structlog`. Never print().
- Errors: define typed exceptions in `app/errors.py`. Don't raise bare Exception.

# Style

- Functions ≤ 40 lines. Refactor if longer.
- No comments that just restate the code. Comments explain *why*.
- Prefer composition over inheritance.

# Don'ts

- Don't add new dependencies without asking.
- Don't write tests for trivial getters/setters.
- Don't use deprecated APIs (e.g., `datetime.utcnow()`).
```

Now you stop re-explaining "we use httpx not requests" five times a day. The AI knows. Update this file as the repo evolves.

### The IDE-AI patterns worth naming and reusing

These are the patterns that, after using them dozens of times, you'll wonder how you ever worked without:

| Pattern | What it looks like in chat |
|---|---|
| **Explain this code** | "Explain `@file:foo.py` in plain English. What does it do? What are the edge cases?" |
| **Find usage** | "Where is `function_x` called from? List call sites and what they do." |
| **What would break if…** | "If I change the signature of `process()` to add a `dry_run` flag, what files would I need to update?" |
| **Generate tests** | "Generate pytest tests for `@file:auth.py` covering: happy path, expired token, invalid signature, missing claim." |
| **Mechanical refactor** | "In `@file:user.py`, rename `getUser` to `get_user` and update all callers in this repo." |
| **Debugging buddy** | "I'm seeing this stack trace: `<paste>`. The relevant code is `@file:x.py`. What's likely wrong?" |
| **Code review my diff** | "Here's my diff: `<paste git diff>`. What's wrong with it? Be specific and harsh." |
| **Write the PR description** | "Summarize the diff `<paste>` into a PR description: 1-sentence summary, bullet list of changes, test plan." |

### The biweekly reflection ritual

Every two weeks, spend 15 minutes:

1. What AI moves saved me the most time in the last 2 weeks?
2. What AI moves backfired (had to redo / debug AI's output)?
3. What patterns am I doing manually that I should be routing through AI?
4. Update my `AGENTS.md` / rules file with anything I had to re-explain repeatedly.
5. Add 2 new prompts to my library.

This compounds. After 3 months, your workflow is unrecognizable from where you started.

---

## 5. Chat tools: ChatGPT vs Claude vs Gemini

You will use these every day, often in parallel. Each has personality and strengths. Knowing which to reach for is worth 30% in efficiency.

### The three families

```
   ┌─────────────────────────────────────────────────────────────┐
   │ CHATGPT (OpenAI)                                            │
   │   Strengths: Broadest tooling, best ecosystem, GPT-5 is     │
   │              the strongest "general" frontier model in '26. │
   │              Best image generation built-in. Best plugins.  │
   │   Use for:   General writing, code, brainstorming, data     │
   │              analysis (Advanced Data Analysis is killer),   │
   │              quick searches, image gen.                     │
   │   Weakness:  Can be over-verbose. Some safety reflexes      │
   │              trigger on benign requests.                    │
   ├─────────────────────────────────────────────────────────────┤
   │ CLAUDE (Anthropic)                                          │
   │   Strengths: Best for long-form writing, coding, careful    │
   │              instruction-following, large contexts, nuanced │
   │              analysis. The "thoughtful colleague" feel.     │
   │   Use for:   Long docs, complex code, design reviews,       │
   │              anything requiring careful reasoning, writing  │
   │              that needs voice (less robotic by default).    │
   │   Weakness:  Smaller plugin ecosystem. No native image gen. │
   ├─────────────────────────────────────────────────────────────┤
   │ GEMINI (Google)                                             │
   │   Strengths: Massive context (1M+ tokens routinely), tight  │
   │              integration with Google Workspace (Gmail,      │
   │              Docs, Drive, Sheets), strongest multimodal     │
   │              (video/audio understanding).                   │
   │   Use for:   Working with very long documents, codebases,   │
   │              video/audio analysis, Google Workspace tasks.  │
   │   Weakness:  Default tone is dry; sometimes flat reasoning. │
   └─────────────────────────────────────────────────────────────┘
```

### Practical "reach-for" heuristic

- **First draft of a document, or rewrite for voice** → Claude
- **Quick answers, image gen, casual brainstorm** → ChatGPT
- **Analyzing a massive PDF, codebase, or video** → Gemini
- **Data in a CSV/Excel, with charts** → ChatGPT (Advanced Data Analysis)
- **Translating a doc with nuance** → Claude
- **Web-search-y questions ("what's the latest on X")** → Perplexity (or ChatGPT with search)
- **Q&A over your own files** → NotebookLM
- **Voice conversation while walking** → ChatGPT voice mode

You don't need accounts on all three on day one. Start with one. After a month, add a second. The differences are real but not so dramatic that you can't get 90% there with any one of them.

### The "ask the same question to two models" trick

For consequential answers (a design decision, an architecture question, an interpretation of contractual text), paste the same question into two different models. Compare. If they agree, you have more confidence. If they disagree, the disagreement itself is informative — it tells you the answer isn't obvious.

This is the AI equivalent of asking two coworkers to sanity-check you. Costs nothing extra. Catches a surprising amount.

---

## 6. Research & summarization workflows

Research is one of the highest-leverage targets for AI. A 2-hour research task collapses to 30 minutes if you do it right.

### The four tools and when to use which

| Tool | When |
|---|---|
| **Perplexity** | Quick, current, web-research-style questions. Returns answer + citations. Default for "what's the latest on…?" |
| **ChatGPT with search** | When you want a more conversational, deeper exploration with iteration. |
| **NotebookLM** | Q&A over a fixed set of *your* sources — papers, PDFs, transcripts. Stays grounded in what you provide. Excellent for studying. |
| **Claude Projects / ChatGPT Projects** | A persistent workspace: upload N reference docs once, ask many questions in the same session over time. |

### The research workflow that actually works

```
   1. CLARIFY the question
       "What am I really trying to learn? What decision does this inform?"

   2. ASK a strong open-ended question
       "Give me an opinionated overview of <topic>. What are the main
        approaches, what are the tradeoffs, who's doing what well in 2026?"

   3. PRESSURE-TEST with follow-ups
       "What are 3 things that would change this overview if I told you
        my use case is <constraint>?"
       "What's the strongest counterargument to your recommendation?"

   4. VERIFY the load-bearing claims
       Take 2–3 specific claims and Google them. AI summaries can be
       confidently wrong on niche facts.

   5. CAPTURE the synthesis
       Paste the final summary into your notes, with a date and the
       sources you verified.
```

### NotebookLM in particular is a hidden gem

NotebookLM lets you upload up to ~50 sources (PDFs, Google Docs, websites, YouTube transcripts) and then Q&A against them. The model is constrained to those sources, so hallucination is dramatically reduced — citations come with line-level highlights.

Use cases:
- Studying for a topic: upload 10 papers and ask "explain the disagreements between these authors."
- Learning a codebase: upload key design docs + the architecture markdown.
- Onboarding: upload your company's wiki pages on a new system.
- Distilling a long video lecture: paste the YouTube link, ask "give me a study guide with timestamps."

You can also generate "audio overviews" — a podcast-style discussion of your sources by two AI hosts. Sounds gimmicky; is actually useful for commute learning.

---

## 7. Writing workflows (emails, docs, PRDs)

The writing leverage is real and easy to underestimate.

### The general writing flow

```
   ┌──────────────────────────────────────┐
   │ 1. You: write a bullet-point dump    │  ← 90% of your time is HERE
   │    of what you want to say           │
   └─────────────┬────────────────────────┘
                 ▼
   ┌──────────────────────────────────────┐
   │ 2. AI: turn bullets into prose       │  ← 1 minute
   │    matching audience & tone          │
   └─────────────┬────────────────────────┘
                 ▼
   ┌──────────────────────────────────────┐
   │ 3. You: read, edit, cut filler,      │  ← 10% of your time
   │    fix what AI got wrong             │
   └─────────────┬────────────────────────┘
                 ▼
              SHIP
```

The mistake: people ask AI to write *the whole thing*, including the actual ideas. Garbage in, garbage out. Your bullets are the ideas. AI is a translator from your bullets to prose.

### Templates for the most common writing tasks

**Email reply (3-line response to a long thread):**

```
TASK: Draft a concise reply to the email thread below.

CONTEXT:
- Audience: [name + relationship: "my manager Sam, who values directness"]
- Constraint: 3 sentences maximum, no pleasantries, no "I hope you're well"
- Outcome I want: [confirm the meeting / decline politely / ask for clarification]

THREAD:
<paste>

Draft 3 versions: (1) neutral, (2) slightly warm, (3) firm.
```

**PR description from a diff:**

```
TASK: Write a PR description for the diff below.

FORMAT:
- One-sentence summary (what this PR does)
- "Why" paragraph (2-3 sentences of context)
- Bulleted "Changes" list (one bullet per logical change)
- "Test plan" section: checklist of how I verified this

DIFF:
<paste>

Keep it under 200 words. Skip filler.
```

**Design doc first draft:**

```
TASK: Write a first-draft design doc for the system described below.

AUDIENCE: senior engineers who don't know this codebase yet.

STRUCTURE:
1. TL;DR (3 sentences)
2. Problem
3. Goals & non-goals
4. Proposed design (high level)
5. Alternatives considered
6. Open questions
7. Rollout plan

CONSTRAINTS:
- 2–3 pages. Don't pad.
- Each section should be skimmable.
- Flag uncertainty explicitly with "🟡 OPEN: ..." inline.

INPUT (my rough notes):
<paste>
```

**Weekly status update:**

```
TASK: Write my weekly update from the bullets below.

AUDIENCE: my engineering manager. Direct, no fluff. Lead with risk.

STRUCTURE:
- Did this week: 3-5 bullets, outcome-oriented
- Doing next week: 3 bullets
- Blocked / at risk: anything they need to act on
- Asks: explicit asks of my manager, if any

BULLETS:
<paste>
```

Save these to your prompt library. After three uses each you'll have them memorized.

---

## 8. Data analysis workflows

For a developer, the single biggest underused superpower is **ChatGPT's Advanced Data Analysis** (or Claude with code interpreter, or Gemini's equivalent).

You drop a CSV/Excel/JSON file in, and ask questions in natural language. It writes and runs Python in a sandbox to answer. Charts come for free.

### What this replaces

```
   BEFORE (manual)                      AFTER (AI-augmented)
   ───────────────                      ────────────────────

   1. Open CSV in Excel                 1. Drop CSV in ChatGPT
   2. Notice it's 50k rows              2. "Summarize this dataset.
   3. Pivot tables                          What columns are there?
   4. Mess with formulas                    Distribution of column X?
   5. Realize you need Python               Outliers? Missing values?"
   6. Open Jupyter                      3. Read the answer in 30 seconds.
   7. Pandas, matplotlib, ...           4. "Now plot Y over time grouped
   8. Two hours later, a chart              by category, log scale."
                                        5. Chart appears.
                                        6. "Now run a t-test between..."
                                        7. Done in 5 minutes.
```

### When this shines

- Any one-off analysis (you don't need a reusable script).
- Quick exploration of a new dataset.
- Sanity-checking numbers before a meeting.
- Generating quick charts for a slide deck.
- Converting between data formats (CSV ↔ JSON ↔ Excel).
- "Clean this messy CSV: remove duplicates, normalize dates, drop rows with X."

### The hidden killer feature: code disclosure

Most data analysis tools let you see the Python code the AI ran. **Read it.** Two reasons:

1. **Verify.** It might have grouped by the wrong column.
2. **Learn.** You're seeing real, working pandas/matplotlib for your dataset. It's a free tutor on data analysis. After a month of this, you'll be way better at pandas yourself.

### Caveats

- For sensitive data, check your provider's data-handling policy. Many enterprise plans don't train on uploaded data. Free tiers may.
- For massive datasets (>~50MB), the sandbox can struggle. Sample or pre-aggregate first.
- For repeated, scheduled analysis, write the code yourself (or get AI to write you a reusable script in your IDE). Don't ChatGPT a chart every Monday.

---

## 9. Meeting capture & knowledge workflows

The "I'll just remember what we discussed" era is over. AI meeting tools are mature, and they save 15–30 minutes per meeting in note-taking and follow-up.

### The tools, briefly

| Tool | Sweet spot |
|---|---|
| **Granola** | Mac-native, takes notes as you take *your own* notes — combines your notes with auto-summary. The "respects your taste" option. |
| **Fireflies** | Joins calls as a bot, transcribes everything, integrates with calendars and Slack. Most enterprise. |
| **Otter** | Mature transcription; weaker AI summary than competitors. |
| **Tactiq / tldv / Read** | Browser-extension based, varying quality. |
| **Native (Zoom AI Companion, Google Meet "Take notes for me", Microsoft Copilot)** | Built into the meeting platform. Often "good enough" if you don't want a 4th tool. |

### What you actually want from these

Don't pick by "most features." Pick by what you'll actually read after the meeting:

```
                THE GOOD MEETING DOC

   ┌────────────────────────────────────────────┐
   │ TL;DR (2 sentences)                        │  ← always read
   ├────────────────────────────────────────────┤
   │ DECISIONS                                  │  ← always read
   │  - Decided X because Y                     │
   │  - Decided NOT to do Z                     │
   ├────────────────────────────────────────────┤
   │ ACTION ITEMS (by owner, with date)         │  ← always read
   │  [ ] Alice: send Q1 numbers by Friday      │
   │  [ ] Bob:   write design doc by next Tue   │
   ├────────────────────────────────────────────┤
   │ OPEN QUESTIONS                             │  ← read once
   ├────────────────────────────────────────────┤
   │ FULL TRANSCRIPT                            │  ← rarely read; reference
   └────────────────────────────────────────────┘
```

Anything that gives you the top 3 sections, well-structured, is good. The transcript itself is mostly a reference.

### The post-meeting workflow

```
   1. Meeting ends.
   2. Tool generates summary (auto).
   3. You: 90-second skim. Edit anything misattributed.
   4. You: copy the action items into your tracker (Jira/Linear/Things).
   5. Done. Move on.
```

The thing to *not* do: try to read every transcript. The transcript is for the rare "wait, what exactly did we agree to?" lookup, not daily reading.

### Knowledge workflows beyond meetings

The same family of tools applies to:

- **Lecture / talk notes** — drop a YouTube link into NotebookLM or ChatGPT, get a structured summary.
- **Book notes** — paste a chapter, ask for the 5 key claims and where you'd push back.
- **Personal journaling search** — embed your journal entries, search semantically.
- **Code review knowledge capture** — when you make a non-obvious decision in a PR, paste the discussion into a prompt that turns it into a 1-paragraph ADR (Architecture Decision Record).

---

## 10. Diagrams, slides, image, and video

The "creative output" surface is where AI gives you superpowers without you being a designer.

### Diagrams from text

For a developer, **Mermaid + AI** is the killer combo. You describe what you want; AI gives you Mermaid; Mermaid renders in GitHub, Notion, Obsidian, almost anywhere.

```
You: "Generate a Mermaid sequence diagram showing user login: browser
      → API → auth service → DB → back. Include token issuance."

AI:  sequenceDiagram
       participant B as Browser
       participant A as API
       participant Auth as Auth Service
       participant DB as Database
       B->>A: POST /login {email, password}
       A->>Auth: verify(email, password)
       Auth->>DB: SELECT user WHERE email=?
       DB-->>Auth: user record
       Auth->>Auth: bcrypt.compare(password, hash)
       Auth->>Auth: issue JWT
       Auth-->>A: { token, user }
       A-->>B: 200 { token }
```

Other options:
- **Excalidraw AI** — hand-drawn style, great for design docs.
- **Eraser** — purpose-built for architecture/sequence/flowchart diagrams from text.
- **Whimsical / Lucidchart AI** — for slides-with-diagrams use cases.

### Slides

| Tool | When |
|---|---|
| **Gamma** | Fast "first-draft deck from a prompt." Excellent default. |
| **Tome** | Visual storytelling; more design-forward. |
| **Beautiful.ai** | Template-heavy; good for sales-y decks. |
| **PowerPoint Copilot / Google Slides Duet** | Inside your existing slide tool, slightly weaker. |

The workflow: bullet-dump your content → AI generates first deck → you edit. Same flow as writing.

### Image generation

Three reasons developers care:

1. **Mockups in design docs.** Replace "imagine a dashboard with X" with an actual image. Massive comprehension boost.
2. **Illustrations for blog posts / READMEs.** Free professional-looking visuals.
3. **Social media / talks.** Slide decks just look better with images.

Tools: Midjourney (best quality, Discord-based), DALL·E (inside ChatGPT, easiest), Stable Diffusion (local, customizable), Sora / Veo / Runway (video).

### Video and audio

In 2026 this is real, but the production polish is still uneven. Use cases that genuinely work today:

- **Talking-head demos** from a script (HeyGen, Synthesia).
- **B-roll footage** for technical videos (Runway, Pika).
- **Voiceovers** in your own voice or a synthetic one (ElevenLabs).
- **Transcribing + cleaning** podcast audio (Descript).

For a developer, the most-used of these is probably ElevenLabs for voiceover or Descript for editing recordings. Skip the rest until you have a need.

---

## 11. The "pick 3" exercise: choose what to integrate

This is the most important part of this whole document. Do not skip it.

### Step 1: Audit your week

For one week, keep a log (rough is fine). Every time you do a task that lasts more than 10 minutes, write down:

- What was the task?
- How long did it take?
- How frequently does it recur (daily, weekly, monthly)?

You'll end up with 20–40 entries. Maybe like:

```
   - Wrote PR descriptions (4 times this week, ~15 min each) = 60 min/wk
   - Triage email inbox (daily, ~20 min) = 100 min/wk
   - Wrote a design doc (once, 4 hours) = 240 min this month
   - Code review on a teammate's PR (3 times, ~25 min) = 75 min/wk
   - Researched "how does X library work?" (twice) = 90 min/wk
   - Wrote weekly status update (1 time, 30 min) = 30 min/wk
   - Investigated a flaky test (2 hours, once) = 120 min this month
   - ...
```

### Step 2: Rank by leverage

For each task, score:

- **Frequency** (how often does this recur?)
- **Time cost** (how long does it take now?)
- **AI fit** (does AI seem like it can help? — easy: writing, summarization, boilerplate, exploration; hard: novel architecture, calibrated judgment, sensitive interpersonal)

Leverage ≈ frequency × time × AI-fit. The top 3 are your targets.

### Step 3: Commit and integrate

For each of your top 3, do this:

```
   ┌─────────────────────────────────────────────────────────┐
   │ TASK: <name>                                            │
   │                                                         │
   │ Tool to use:    <Cursor agent / ChatGPT / NotebookLM>   │
   │ Template:       <link to your prompt-library entry>     │
   │ Commitment:     Next 4 weeks, use AI for this every time │
   │ Reflection:     End of each week, was it faster? What    │
   │                 needed editing? Update template.         │
   └─────────────────────────────────────────────────────────┘
```

Stick three of these on the wall (literally or metaphorically). Refer to them when you catch yourself doing the old way.

### Step 4: Add the next 3 after a month

Don't try to do 10 at once. Do 3. Once they're reflex, add 3 more. After 6 months you've integrated 18 workflows and you're operating at a different level.

### A reasonable "Pick 3" for a senior backend dev

If you don't want to audit and just want a default, here's a high-yield starter set:

1. **All PR descriptions go through an AI template** (saved in your prompt library).
2. **Every research task starts with Perplexity or NotebookLM**, not Google.
3. **All non-trivial commits start in your IDE with chat or agent mode**, with a clear scope and constraints.

These three alone reclaim 5–8 hours/week for most engineers.

---

## 12. Common anti-patterns (the dabbler trap)

### Anti-pattern 1: Tool churn

You read a hot-take, switch to Tool X. Two weeks later, you switch to Tool Y. You never get deep with any of them. **Fix:** pick a tool. Use it for 2 months before switching. The marginal differences between top tools are smaller than the gains from depth in one.

### Anti-pattern 2: One-line questions

You type "best way to design a rate limiter?" into ChatGPT. You get a generic response. You blame ChatGPT. **Fix:** apply Stage 2's six-part skeleton. Context, constraints, output format — every time.

### Anti-pattern 3: Blind acceptance

You let Copilot autocomplete a function. You don't read the suggestion. Three days later, production has an off-by-one. **Fix:** read every accepted suggestion. The cost is one second of attention. The value is your job.

### Anti-pattern 4: Demoing instead of integrating

You show coworkers cool AI tricks at lunch. You don't actually route any of your work through AI consistently. **Fix:** stop demoing. Start integrating.

### Anti-pattern 5: Skipping the verify step

You ask AI for a quote. You paste it into your doc. It's fabricated. **Fix:** for any factual claim that matters, verify. The 30-second cost of verification is the difference between trustworthy work and embarrassing yourself.

### Anti-pattern 6: Treating AI as the writer

You ask AI to write your design doc from scratch. The output is bland, generic, and your reviewers can tell. **Fix:** YOU write the bullets. AI translates. The ideas are yours.

### Anti-pattern 7: Forgetting it's not the only tool

For some tasks, AI is genuinely slower than the old way. Quick keyword search? `grep` is faster than chat. Simple arithmetic? Calculator. Don't romanticize AI; use the right tool.

### Anti-pattern 8: Ignoring the IDE

You diligently use ChatGPT in a browser. You never set up Cursor / Copilot agent / Claude Code. You're leaving the biggest leverage on the table. **Fix:** the IDE is where you spend your day. Optimize it first.

### Anti-pattern 9: Letting agent mode run unsupervised

You give Cursor's agent a vague goal. You go get coffee. You come back to 47 modified files, half of them broken. **Fix:** narrow scope, explicit constraints, always-review-diff. Agent mode is power tools — gloves on.

### Anti-pattern 10: Not updating your templates

Your "PR description" prompt was written 6 months ago. Your team has new conventions. Your prompts still produce old-style PRs. **Fix:** biweekly reflection ritual (§4). Templates rot. Maintain them.

---

## 13. Hands-on exercises

These are different from the labs in Stage 1 and 2 — they're **workflow-building drills**, not concept tests. Each one takes a real task in your week and routes it through AI.

### Exercise 1: The audit (60 min, once)

Run the §11 audit. Spend an hour with a notebook open. Write down every task you do this week. Score them. Pick your 3. Commit them to a doc.

**Deliverable:** a `pick-3.md` file in your notes with the three workflows and the templates you'll use.

### Exercise 2: PR description automation (1 PR)

Next time you open a PR:

1. Don't write the description manually.
2. Run `git diff origin/main..HEAD` and copy the output (or use the IDE's "summarize diff" feature).
3. Paste it into a saved "PR description" prompt template (see §7).
4. Edit the output. Note what needed fixing.
5. Ship.

**Deliverable:** the prompt template, refined based on what you had to edit.

### Exercise 3: Research workflow drill (1 topic)

Pick a topic you need to learn this month (a library, a pattern, a concept). Do the research workflow from §6:

1. Clarify the question.
2. Strong open-ended ask in Perplexity or Claude.
3. Three pressure-test follow-ups.
4. Verify 2 load-bearing claims.
5. Capture the synthesis in your notes.

Time-box it: 30 minutes total.

**Deliverable:** a research note in your knowledge base.

### Exercise 4: IDE rules file (60 min)

For your main project, create or update a rules file (`.cursor/rules/*.mdc`, `AGENTS.md`, or `CLAUDE.md`).

Include:
- Tech stack and versions
- Code style conventions
- Common imports/utilities to prefer
- Things to never do
- Testing conventions

**Deliverable:** committed rules file. Notice how chat/agent suggestions improve over the next week.

### Exercise 5: Data analysis warm-up (30 min)

Find any CSV or dataset (your bank statement export, GitHub commit history, a public CSV). Drop it into ChatGPT Advanced Data Analysis or Claude with code execution.

Ask:
1. "Summarize this dataset."
2. "Show me the distribution of <column>."
3. "Identify any outliers or anomalies."
4. "Plot <X> over time, grouped by <Y>."

Read the Python code it produces. Notice what you learn.

**Deliverable:** a short note on what you learned about the dataset *and* about pandas/matplotlib.

### Exercise 6: Meeting capture for one week

For a full week, use a meeting capture tool (Granola, Fireflies, native Zoom AI). After each meeting:

1. Read the auto-summary.
2. Correct any misattributions in 60 seconds.
3. Move action items to your task tracker.
4. Don't read the full transcript.

End of week: did you spend less time on meeting notes? Were the action items captured better than before?

**Deliverable:** an honest reflection, kept or dropped.

### Exercise 7: The biweekly reflection (15 min, recurring)

Two weeks from now, do this:

1. What AI moves saved you the most time?
2. What backfired?
3. What manual habits should you route through AI next?
4. Update rules file.
5. Add 2 new prompts to library.

**Deliverable:** schedule it as a recurring calendar event. Forever.

### Exercise 8 (optional): The "all-AI day"

Pick one workday. Try to route *every* task that could plausibly use AI through AI. Be deliberate. Note what worked and what was friction.

This isn't sustainable — you wouldn't do it every day — but it's a calibration exercise. You'll discover workflows you weren't using and tools that don't fit your style.

**Deliverable:** a list of "keepers" (workflows you'll continue) and "drop list" (didn't help).

---

## 14. Interview questions companies actually ask

For an experienced developer in 2026, Stage 3 questions show up in interviews as practical or behavioral, not theoretical. Companies want to know you've *integrated* AI into how you work, not just heard about it.

### Conceptual / behavioral

1. **"Walk me through how AI is integrated into your daily workflow."**
   *What they want:* specifics, not buzzwords. Name 3 concrete workflows you've changed. Give time-saved estimates. Show you have prompt templates. Show you reflect and iterate. *Red flag answer:* "I use ChatGPT sometimes."

2. **"How do you decide when to use AI for a task vs do it yourself?"**
   *Good answer:* Three filters: (1) Is it a recurring task where investing in a prompt pays back? (2) Is it the kind of task AI does well — writing, summarization, exploration, boilerplate — or does it need calibrated judgment / sensitive interpersonal / novel architecture? (3) Is the cost of verification cheap relative to the time saved? If all three are yes, AI. Otherwise, manual.

3. **"What's the difference between using AI well and badly?"**
   *Good answer:* The dabbler-vs-integrator frame. Bad: one-line questions, blind acceptance, tool churn, no templates, no measurement. Good: rich context, verification, prompt library, biweekly reflection, "AI as junior teammate" mindset.

4. **"How do you avoid AI-induced bugs in your code?"**
   *Good answer:* Read every accepted suggestion, even completions. Run tests after agent-mode changes — always. Set explicit constraints when delegating multi-file work. Use rules files to enforce repo conventions. Never accept code for security-sensitive areas without manual review. Treat AI suggestions as a junior PR — review before merge.

5. **"Have you ever had AI lead you astray? What happened?"**
   *Good answer:* Tell a real story. Be specific about how the failure happened, how you noticed, what you do differently now. (Showing you can fail and learn is a senior trait.) Don't pretend you've never been burned — that's the bigger red flag.

### Practical

6. **"You're new to a 500k-line codebase. How would you use AI to ramp up?"**
   *Good answer:* (1) Drop the architecture docs / READMEs into NotebookLM and Q&A against them. (2) Use IDE chat with repo context for "explain this module," "where is X used," "what's the data flow for Y." (3) Use agent mode for guided refactors with explicit constraints, to learn the conventions by doing. (4) Build a personal note: list of weird patterns, gotchas, where things live. (5) Ask AI to generate a sequence diagram of a key flow from the code — fastest way to internalize an unfamiliar system.

7. **"How would you set up a junior engineer for success with AI tools?"**
   *Good answer:* (1) Pair on real tasks, not on demos — show them in the flow. (2) Set up the rules file together so the AI knows the team conventions. (3) Build a starter prompt library: PR descriptions, code review, test generation. (4) Insist on the verify-and-edit step early; that's the biggest skill gap. (5) Biweekly reflection ritual — what worked, what backfired.

8. **"Your team is debating whether to standardize on Cursor, Copilot, or Claude Code. How would you decide?"**
   *Good answer:* The differences are marginal compared to depth of use. Run a 2-week pilot of each with 2–3 engineers. Measure: subjective survey (1–5 on speed, accuracy, friction), objective metric (PRs/eng/week if measurable), agent-mode reliability on a standardized task set. Decide and stop debating — the org-wide cost of indecision exceeds the difference between tools.

9. **"How do you stay current as new AI tools come out every month?"**
   *Good answer:* (1) Maintain a small set of trusted sources (Latent Space, Simon Willison's blog, two newsletters). (2) Try new tools quickly but evaluate slowly — 1-week trial, but don't switch defaults unless materially better. (3) Reflection ritual catches "huh, this old workflow is suddenly worse than the new way." (4) Accept you'll never be at the bleeding edge; focus on depth, not breadth.

10. **"How do you handle AI tool usage with sensitive data (customer PII, code IP)?"**
    *Good answer:* Know your org's policy and the data-handling terms of each tool. For sensitive data: prefer enterprise plans with no-training agreements, run locally via Ollama for highly sensitive cases, redact PII before sending, use approved tools only. For code IP: most enterprise plans are fine, but the policy depends on legal. Don't be the engineer who pasted prod secrets into a free ChatGPT account.

### Senior-level

11. **"How would you measure the ROI of AI tools across an engineering org?"**
    *Hint:* Be careful — most measurement here is noisy. Combine: subjective survey (do engineers feel faster? on what tasks?), ecosystem metrics (PRs/eng/week, cycle time, lead time), targeted A/B (a few teams without the tool for a quarter — politically hard but cleanest signal), spend-vs-time-saved heuristic. Avoid vanity metrics like "lines generated."

12. **"Design a process for rolling out a new AI tool to a 200-engineer org."**
    *Hint:* (1) Champions: 5–10 engineers who pilot for a month. (2) Workshop: half-day intro for everyone, with hands-on templates. (3) Rules files: build per-team rules together. (4) Office hours: weekly drop-in for questions, run by champions. (5) Measurement: subjective survey at 1mo and 3mo. (6) Pay attention to the "AI skeptics" — they often have real concerns (security, accuracy). Their feedback shapes the rollout. (7) Don't mandate adoption; mandate trying for 4 weeks.

13. **"A senior engineer on your team refuses to use AI tools, citing accuracy concerns. How do you handle it?"**
    *Hint:* This is a behavioral/leadership question. Don't dismiss them. Their concerns are often legitimate — AI does hallucinate, blind use does cause bugs. Pair with them on a real task. Show them the integrator's frame (verify, narrow scope, templates). Let them keep their veto on high-risk areas (security, critical paths). The goal isn't conformity; it's that they make an informed choice. Often they end up using AI for the low-risk tasks and earning the team's respect for being deliberate about the high-risk ones.

14. **"How does AI integration change how you do system design now?"**
    *Hint:* You explore more options faster (ask 3 designs, compare). You catch issues earlier (ask for the strongest counterargument). You generate diagrams from text mid-discussion. You research unfamiliar tech in 30 min instead of 3 hours. But the actual design decisions — tradeoffs, judgment calls — remain yours. AI accelerates the *thinking*, doesn't replace it.

15. **"What's a recent example of AI changing the way you do a task you'd been doing for years?"**
    *Open-ended. Tell a specific story.* The interviewer wants to see (a) self-awareness, (b) curiosity, (c) ability to change. Almost any honest specific story is a better answer than a polished generic one.

---

## 15. Self-check: are you done with Stage 3?

Stage 3 is never "done" — it's a habit. But you've reached a competent baseline when you can:

- [ ] List the 3 workflows you've consciously integrated, with templates saved.
- [ ] Estimate how much time AI saves you per week, with specifics.
- [ ] Demonstrate using AI inside your IDE (chat + agent mode) on a real task in <10 min.
- [ ] Show a current rules file (`AGENTS.md` / `.cursor/rules/*` / `CLAUDE.md`) for at least one repo.
- [ ] Show a personal prompt library with 10+ entries.
- [ ] Name when you'd use ChatGPT vs Claude vs Gemini vs Perplexity vs NotebookLM.
- [ ] Describe the "verify" habit and when you apply it.
- [ ] Recite at least 3 anti-patterns and how you avoid them.
- [ ] Have a biweekly reflection cadence (calendar event exists).
- [ ] Have onboarded at least one other person to one AI workflow (mentoring is the senior signal).

8+ checked → you're competent at Stage 3. Move on to Stage 4 (RAG) — but **don't stop the daily integration**. Stages 4–7 are about building; Stage 3 is about how you work. They run in parallel forever.

If you can check 4 or fewer, your problem isn't reading more. Your problem is habit-building. Go back to §11 ("Pick 3"). Commit. 4 weeks. Then come back.

---

## Further reading (optional)

- **[Simon Willison's blog](https://simonwillison.net/)** — best ongoing practical commentary on real-world LLM use.
- **[Every's "Chain of Thought"](https://every.to/chain-of-thought)** — knowledge worker AI workflows, well-written.
- **[Latent Space podcast](https://www.latent.space/)** — engineering-focused interviews with AI practitioners.
- **[Geoffrey Litt's blog](https://www.geoffreylitt.com/)** — thoughtful pieces on AI-augmented workflows.
- **[Hamel Husain's blog](https://hamel.dev/)** — practical AI engineering, including workflow tooling.

For meeting capture and writing tool specifics, vendor blogs are fine but not where the lasting insights live.

---

## What's next

Stage 3 is your daily operating system. It runs in the background forever, getting refined every two weeks.

The next stages are where you start *building*. Stage 4 (RAG) is your first real engineering project: making AI answer questions over your own data. It's where this stops being about consumption and starts being about creation.

See [AI_ROADMAP.md](./AI_ROADMAP.md) for the full path, or jump to [AI_STAGE4_RAG.md](./AI_STAGE4_RAG.md) (coming next).
