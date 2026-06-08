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

## Repository Structure

```
claude-magic-skills/
├── README.md
├── context-saver.skill       ← ZIP: context-saver/SKILL.md
└── prompt-enhancer.skill     ← ZIP: prompt-enhancer/SKILL.md
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
