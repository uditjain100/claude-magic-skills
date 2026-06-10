# claude-magic-skills

A personal collection of custom skills for [Claude Code](https://claude.ai/code) — packaged as `.skill` files and ready to install. Each skill extends Claude Code with a new slash command, giving it structured, repeatable behavior for common workflows.

---

## What Are Skills?

Skills are Claude Code's plugin system. A `.skill` file is a ZIP archive containing a `SKILL.md` — a markdown file with a YAML frontmatter header (name + description) and a body that serves as a system-level instruction set for Claude. When a skill is installed, Claude Code surfaces it as a slash command (e.g. `/context-saver`, `/prompt-enhancer`) and follows its instructions exactly when invoked.

**Anatomy of a skill:**
```
my-skill.skill           ← ZIP archive
└── my-skill/
    └── SKILL.md         ← markdown with YAML frontmatter + instructions
```

**SKILL.md frontmatter:**
```yaml
---
name: my-skill
description: >
  One or more sentences that describe when Claude should auto-trigger this skill.
  Claude reads this to decide whether to invoke the skill automatically.
---
```

---

## Installation

### Option 1 — Install a single skill via Claude Code

```bash
claude skill install <path-to-skill-file>
```

### Option 2 — Install all skills in this repo

Clone the repo and install everything at once:

```bash
git clone https://github.com/uditjain/claude-magic-skills.git
cd claude-magic-skills
for skill in *.skill; do claude skill install "$skill"; done
```

### Verify installed skills

```bash
claude skill list
```

---

## Skills

### 1. `context-saver`

**File:** [context-saver.skill](context-saver.skill)

Saves the entire current conversation to a structured Markdown file, and reloads it in a future session — giving Claude persistent memory across completely separate chats.

#### Modes

**SAVE mode** — triggered when you say things like:
- `"save this chat"`
- `"save the context"`
- `"export this conversation"`
- `"I want to continue this later"`
- `"checkpoint this session"`
- `"save our progress"`

Claude will:
1. Reconstruct the full conversation from the current context window (messages, code, decisions, artifacts)
2. Build a structured `.md` file using a fixed template
3. Write it to `/mnt/user-data/outputs/context_<topic>_<YYYYMMDD>.md`
4. Present the file for download
5. Tell you exactly how to reload it

**LOAD mode** — triggered when you upload a previously saved context file and say things like:
- `"load context from file"`
- `"restore session"`
- `"continue from file"`
- `"pick up where we left off"`

Claude will:
1. Read the file fully
2. Verify it uses the context-saver format
3. Summarize what was restored (topic, date, key decisions, pending items)
4. Resume the session naturally — no re-explaining needed

#### Reload Prompt (copy-paste this when starting a new chat)

Upload your saved `.md` context file and use this prompt to trigger a structured restore:

```
I'm resuming a previous session. I'm uploading a context file saved by the context-saver skill.

Please do the following:
1. Read the entire file carefully before responding
2. Confirm what topic/session you've restored by stating:
   - The original topic
   - Date it was saved
   - Current status (In Progress / Completed / Paused)
3. Summarize the KEY FACTS & DECISIONS in 3-5 bullet points
4. List any PENDING ACTION ITEMS
5. Then ask me: "What would you like to work on next?" OR if the next step is obvious from the file, suggest it directly and ask if I want to proceed.

Treat everything in the KEY FACTS & DECISIONS section as ground truth — do not re-derive or question established decisions. Pick up naturally as if the conversation never stopped.
```

> **Why use this prompt?** Without it, Claude may partially read the file or respond generically. This prompt forces a structured restore — confirmed topic, date, status, decisions, and next steps — before anything else.

#### Saved File Structure

```markdown
# SAVED CONTEXT — {TOPIC}

## SESSION METADATA      ← topic, date, goal, status
## KEY FACTS & DECISIONS ← dense TL;DR — what Claude needs to immediately re-orient
## PENDING ACTION ITEMS  ← checkboxes for unfinished work
## ARTIFACTS PRODUCED    ← table of files/code/docs created
## CONVERSATION HISTORY  ← full reconstructed turn-by-turn transcript
## RELOAD INSTRUCTIONS   ← copy-paste instructions for the next session
```

#### Quality Guarantees

- Code blocks are never truncated or paraphrased — always verbatim
- Numbers, filenames, URLs, and technical specs are preserved exactly
- For very long conversations (50+ turns), middle turns are grouped into labeled summaries; first 5 and last 10 turns are always kept verbatim
- Sensitive data (passwords, API keys, PII) is automatically redacted with a `[REDACTED]` marker

---

### 2. `prompt-enhancer`

**File:** [prompt-enhancer.skill](prompt-enhancer.skill)

Takes a rough, vague, or unstructured prompt and rewrites it into a clean, detailed, production-quality version — then immediately runs the enhanced prompt so you see the result right away.

#### Trigger Phrases

- `"improve my prompt"`
- `"enhance this prompt"`
- `"rewrite this prompt"`
- `"make this prompt better"`
- `"fix my prompt"`
- `"restructure my prompt"`
- `"turn this into a good prompt"`
- Pasting any prompt and asking for help with it

Works for all prompt types: system prompts, LLM API prompts, task prompts (coding, writing, research), and general-purpose Claude/ChatGPT prompts.

#### Output Format

Every invocation produces three sections in order:

**1. Enhanced Prompt**
The fully rewritten, clean, copy-ready prompt. No meta-commentary inside it.

**2. Optional Additions**
3–6 bullets of things you *could* add to push the prompt further — kept separate so the main output stays clean. Examples: adding few-shot examples, defining a JSON output schema, adding chain-of-thought instructions, setting response length constraints.

**3. Output**
The enhanced prompt is run immediately after being shown. You get both the improved prompt and its result in one shot.
- If the prompt requires external input (e.g. "paste your article here"), Claude skips execution and prompts you to provide it.
- If it's a system prompt (configures AI behavior, not direct output), Claude skips execution and explains how to use it.

#### Rewriting Principles

| Principle | What it means |
|-----------|---------------|
| Preserve intent | Never changes what you're fundamentally asking for |
| Eliminate ambiguity | Replaces vague words ("good", "nice", "proper") with specific, observable criteria |
| Be explicit | If the original assumes Claude will "just know" something, spells it out |
| Positive instructions | "Do X" instead of "Don't not do X" |
| One goal per prompt | Flags multi-task prompts and suggests splitting them |
| Match prompt type | System prompts get `You are…` framing; API prompts get precise input/output contracts; task prompts lead with the task |

#### Edge Cases

- **Already well-written prompt:** Makes minimal changes; explains what was preserved and why
- **Too vague to enhance:** Asks one clarifying question (the single most important missing piece) before rewriting
- **Multi-task prompt:** Rewrites as-is, but flags in Optional Additions that splitting would improve reliability
- **Non-English prompt:** Rewrites in the same language

---

### 3. `project-analyzer`

**File:** [project-analyzer.skill](project-analyzer.skill)

Recursively walks every file and folder in a tech project, then produces a structured quick-start analysis report — so you can immediately understand any codebase and start making changes without reading it file by file.

#### Trigger Phrases

- `"analyze this project"`
- `"explain this codebase"`
- `"what does this project do"`
- `"help me understand this repo"`
- `"I want to work on this project"`
- `"give me an overview of this code"`
- `"dive into this directory"`
- Uploading a project zip or pointing to a folder and asking what it does
- Pasting a file tree or list of project files

> Does **not** trigger for single-file code review — only multi-file projects.

#### How It Works

**Step 1 — Locate the project root**
Accepts a directory path, an uploaded zip (auto-extracted), or a path you type.

**Step 2 — Recursive directory walk**
Runs `tree` (with noise excluded: `node_modules`, `.git`, `__pycache__`, `dist`, `build`, `.next`, etc.) to map the full structure. Falls back to `find` if `tree` is unavailable.

**Step 3 — Surgical file reading**
Reads files in strict priority order — not everything, just what matters:

| Priority | What gets read |
|----------|----------------|
| 1 — Identity | `README.md`, `package.json`, `pyproject.toml`, `requirements.txt`, `.env.example` |
| 2 — Entry points | `main.py`, `app.py`, `index.ts`, `server.js`, `manage.py`, etc. based on detected framework |
| 3 — Core logic | Files with names suggesting core functionality (`agent.py`, `pipeline.py`, `services/`, `api/`) |
| 4 — Infra (skim) | `Dockerfile`, `docker-compose.yml`, CI workflows, bundler configs |

Test files, generated files, and lock files are skipped unless specifically relevant.

**Step 4 — Structured analysis report**
Outputs a fixed, scannable report covering:

- **What It Does** — plain-English TL;DR paragraph
- **Architecture at a Glance** — overall pattern (e.g. "Next.js + FastAPI + PostgreSQL")
- **Annotated Folder Structure** — top-2-level tree with every line annotated
- **Tech Stack** — table by layer (language, framework, AI/ML, database, infra)
- **Entry Points** — exactly which files start the app and why
- **Key Files to Know** — table of the files you'll touch most, with their roles
- **Notable Dependencies** — only the important/non-obvious ones
- **Data / Request Flow** — 1–3 sentences on how data moves through the system
- **How to Run** — install + configure + run commands (from README or inferred)
- **Gotchas / Things to Know** — 2–4 non-obvious architectural decisions or footguns
- **Good Starting Points for Changes** — specific files to open depending on what you want to change

The report ends with: `Ready — what changes do you want to make?`

#### Behavior Guarantees

- Every section uses **actual names from the project**, never placeholder text
- Infers purpose from code when there's no README
- Omits sections that don't apply rather than writing "N/A"
- Entire output is designed to be skimmable in under 2 minutes

---

### 4. `ats-scorer`

**File:** [ats-scorer.skill](ats-scorer.skill)

Performs a precise, multi-dimensional ATS (Applicant Tracking System) score analysis on a resume against a job description. Produces a numeric score (0–100), category-level breakdown, keyword gap analysis, format audit, and a ranked fix list — so you know exactly what to fix before submitting.

#### Trigger Phrases

- `"check ATS score"`
- `"scan my resume"`
- `"how ATS-friendly is my resume"`
- `"will my resume pass ATS"`
- `"ATS check"` / `"ATS analysis"`
- `"keyword match"` / `"will I get filtered out"`
- `"what keywords am I missing"`
- `"is my formatting ATS-safe"`
- `"optimize for ATS"`
- Pasting a resume + job description and asking how well they align

#### Scoring Model

Scores across 8 weighted categories — research-calibrated, not arbitrary:

| Category | Weight | Why It Matters |
|----------|--------|---------------|
| Hard Skills & Keyword Match | 30% | Single biggest driver of ATS ranking |
| Job Title & Seniority Alignment | 18% | Title match is the highest-impact ATS signal |
| Format & Parseability | 17% | Parse failure invalidates all downstream scoring |
| Skills Section Coverage | 15% | Dedicated section materially improves keyword extraction |
| Quantification & Impact | 8% | AI-native ATS and all recruiters weight this |
| Education Match | 7% | Required-degree filters are hard knockouts in enterprise ATS |
| Career Trajectory & Signals | 3% | Progression matters to AI-native ATS |
| Soft Skills (contextual) | 2% | Credited only when used naturally with evidence |

#### Output

Each run produces:
- **Final score** (0–100) with band label (Excellent / Good / Fair / Weak / Poor)
- **Category breakdown** with score, max, and status for each of the 8 categories
- **Parsed resume snapshot** — what an ATS parser would actually extract
- **Keyword match analysis** — matched (exact / acronym / semantic) and missing (critical / recommended / predicted)
- **Format audit** — 11-point checklist with PASS/PARTIAL/FAIL and specific fixes
- **Platform simulation** — score estimates for Taleo/SuccessFactors (strict), Workday/iCIMS (standard), and Greenhouse/Lever (modern)
- **Priority fix list** — up to 8 ranked fixes, each with estimated points gained and exact instruction
- **Score potential** — current, after top 3 fixes, after all fixes, realistic ceiling

#### Platform Detection

If you provide an application URL, the skill detects the ATS platform (Workday, Taleo, Greenhouse, Lever, iCIMS, etc.) and applies that platform's specific parsing rules rather than showing a generic simulation.

#### Key Principle

> A score near 100% via keyword stuffing is worse than a score of 75% with clean, natural text. The target sweet spot is **75–85%** — strong match without sacrificing recruiter readability.

---

### 5. `resume-tailor`

**File:** [resume-tailor.skill](resume-tailor.skill)

Takes your existing resume and a target job description, and produces a fully tailored, ATS-optimized, recruiter-ready version — rewriting bullets, reordering sections, aligning keywords, and adjusting the summary — without fabricating any experience.

#### Trigger Phrases

- `"tailor my resume"`
- `"customize resume for this JD"`
- `"update my resume for this role"`
- `"help me apply for this job"`
- `"make my resume match this posting"`
- `"optimize my resume"` / `"rewrite my resume for"`
- `"highlight relevant experience"`
- `"add keywords from JD"`
- `"make my resume ATS-friendly"` for a given job
- Pasting a job description alongside your resume and asking for help

#### How It Works

**Step 1 — Analysis:** Before rewriting, shows a brief analysis block — role level, domain match strength, gap areas, hidden strengths, ATS risks, and whether sections need reordering.

**Step 2 — Tailoring:** Applies a systematic set of rules:
- **Keyword alignment** — injects JD keywords naturally into bullets and summary using exact JD phrasing
- **Bullet rewriting** — uses the STAR-L formula (verb + what + how + result + scale); directly relevant bullets get JD language, tangential bullets get reframed, irrelevant bullets get deprioritized
- **Summary rewrite** — mirrors the JD's language for the ideal candidate; opens with role title alignment
- **Skills reordering** — JD-matching skills move to the top; JD terminology is used verbatim
- **Section ordering** — reorders top-level sections based on role type (engineering, research, new grad, hybrid)
- **Projects elevation** — JD-aligned projects move to the top; descriptions rewritten with JD keywords

#### Output

```
📄 TAILORED RESUME — [Job Title] at [Company]
[Full resume — ready to copy into a doc]

🔍 CHANGES MADE
Summary, keywords added, bullets rewritten, skills/sections reordered

⚠️  GAP REPORT
Honest list of JD requirements the resume can't cover, each with severity (critical / minor / nice-to-have) and a suggested mitigation
```

#### Hard Constraints

The skill never fabricates. It will not invent job titles, companies, degrees, skills, certifications, or impact numbers. It reframes, reorders, rewords, and reemphasizes — but every claim must trace back to something the user actually did.

#### Edge Cases Handled Automatically

Detects and adjusts for: career change, new grad, senior/staff/lead roles, research roles, overqualified candidates, employment gaps, multiple roles at the same company, and international resume formats.

---

### 6. `fact-checker`

**File:** [fact-checker.skill](fact-checker.skill)

Performs deep, line-by-line web research on any text or document to verify whether each factual claim is true, false, unverifiable, or uncertain — then produces a per-claim confidence score and a full visual verification report.

#### Trigger Phrases

- `"fact-check this"`
- `"verify this text"` / `"is this accurate"`
- `"check if this is true"` / `"verify these claims"`
- `"audit this content"` / `"research and validate this"`
- Pasting text and asking whether it's correct
- Uploading a document (resume, article, report, bio, LinkedIn excerpt) and asking for fact verification

#### How It Works

**Phase 0 — Parse the Input**
Extracts every verifiable factual claim (names, dates, numbers, credentials, events, statistics, titles, institutions) from the pasted text or uploaded file. Each claim is numbered `[F1]`, `[F2]`, …

**Phase 1 — Web Research**
For every claim, runs a targeted web search and fetches full pages when needed. Never relies on training data alone — all claims are actively researched.

**Phase 2 — Score Each Claim**
Assigns a Confidence Score (0–100) and a verdict:

| Verdict | Score | Meaning |
|---------|-------|---------|
| ✅ VERIFIED | 80–100 | Confirmed by credible source(s) |
| ⚠️ LIKELY TRUE | 60–79 | Supported by indirect/partial evidence |
| ❓ UNVERIFIABLE | 40–59 | No reliable source found either way |
| 🔴 LIKELY FALSE | 20–39 | Evidence contradicts the claim |
| ❌ FALSE | 0–19 | Directly refuted by credible source(s) |
| 🤷 UNKNOWN | — | Cannot determine; more research needed |

#### Output

Each run produces (in order):

1. **Extracted Claims** — numbered list of all verifiable facts found
2. **Research & Verification Table** — claim, verdict, score, evidence summary, and source URL for every claim
3. **Visual Confidence Report** — color-coded bar chart/card grid with per-claim scores, summary stats (total claims, verified, false, unknown), and an Overall Document Score
4. **Final Verdict** — 3–5 sentence plain-English summary: overall reliability, critical issues, and what to do with the content

#### Edge Cases Handled Automatically

- **Resume / CV input:** Treats job titles, dates, institutions, and degree names as individual facts; résumé fraud detection mode
- **News articles:** Focuses on named events, statistics, and untraced quotes
- **Technical docs:** Verifies API names, version numbers, and library behaviors against official documentation
- **Very long input (20+ claims):** Chunked into groups of 10, then combined into a single final report
- **Private/internal claims** (e.g., "I led a team of 12"): Marked UNKNOWN — not verified or fabricated

---

## Repository Structure

```
claude-magic-skills/
├── README.md
├── context-saver.skill       ← ZIP: context-saver/SKILL.md
├── prompt-enhancer.skill     ← ZIP: prompt-enhancer/SKILL.md
├── project-analyzer.skill    ← ZIP: project-analyzer/SKILL.md
├── ats-scorer.skill          ← ZIP: ats-scorer/SKILL.md + references/ats-platforms.md
├── resume-tailor.skill       ← ZIP: resume-tailor/SKILL.md
└── fact-checker.skill        ← ZIP: fact-checker/SKILL.md
```

---

## Adding a New Skill

1. Create a directory named after your skill:
   ```bash
   mkdir my-skill
   ```

2. Write `my-skill/SKILL.md` with the required frontmatter:
   ```markdown
   ---
   name: my-skill
   description: >
     One or more sentences describing when this skill should be triggered.
     Be specific — Claude reads this to decide auto-invocation.
   ---

   # My Skill

   [Instructions for Claude go here...]
   ```

3. Package it as a `.skill` file (ZIP with directory):
   ```bash
   zip -r my-skill.skill my-skill/
   ```

4. Install and test:
   ```bash
   claude skill install my-skill.skill
   ```

5. Add it to this repo with a section in the README covering: trigger phrases, what it does, and output format.

---

## License

MIT
