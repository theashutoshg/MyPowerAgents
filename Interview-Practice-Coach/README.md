# Interview Practice Coach

Runs a mock behavioral/technical interview from a pasted job description (JD). The agent plays interviewer — one question at a time — and coaches each answer into a tight, defensible STAR-format story.

See [SKILL.md](SKILL.md) for the full workflow definition (source of truth) and [CLAUDE.md](CLAUDE.md) for folder-specific conventions.

## How to install the skill

The skill is a single `SKILL.md` file placed in opencode's global skills directory:

```
~/.config/opencode/skills/interview-practice-coach/SKILL.md
```

### Quick setup (copy from this repo)

```bash
# From the repo root:
mkdir -p ~/.config/opencode/skills/interview-practice-coach
cp Interview-Practice-Coach/SKILL.md ~/.config/opencode/skills/interview-practice-coach/SKILL.md
```

opencode auto-discovers skills from `~/.config/opencode/skills/<name>/SKILL.md` — no registration, no config changes needed.

### Verifying it's loaded

After copying, restart opencode (or start a new session), then ask:

> "What skills do you have available?"

The `interview-practice-coach` skill should be listed with its description.

## How to run the skill

Paste a job description and ask to be interviewed. The agent recognizes the intent and loads the skill automatically:

```
> Here's the JD for a Senior PM role at Acme — interview me for it

> Give me a mock interview based on this job posting

> Practice my STAR answers against this JD

> Ask me interview questions like a hiring manager would
```

The skill triggers on: *interview me*, *mock interview*, *practice interviewing*, *prepare for an interview*, *ask me questions like an interviewer*, *build/refine STAR answers*, *rehearse behavioral answers* — you don't need to name the skill explicitly, and you don't need to repeat the JD in every message once it's been shared earlier in the conversation.

### What happens

1. **JD analysis** — the agent extracts core responsibilities, must-have qualifications, and 4-8 competency themes to build questions around.
2. **One question at a time** — starts with background/motivation, then one behavioral question per competency, sequenced the way a real panel would.
3. **Coaching after every answer** — what worked, what's missing (quantification, a clear bridge to the role, STAR gaps), a rewritten version using only the user's real details, and a likely follow-up question.
4. **Mismatch detection** — if an answer doesn't actually fit the competency being tested, the agent says so and redirects rather than force-fitting it.
5. **Dictation cleanup** — silently fixes obvious transcription errors, confirms ambiguous or domain-specific terms before locking them into a final answer.
6. **Momentum** — after each coaching pass, offers to redo the answer, drill the follow-up, or move on; advances as soon as the user signals readiness.

### Example prompts

| Prompt | What happens |
|---|---|
| `Interview me for this role` (JD pasted) | Extracts competency themes, opens with a background question |
| *(after an answer)* `redo` | Re-asks the same question for another attempt |
| *(after coaching)* `hit me with the follow-up` | Asks the pressure-test follow-up question drafted during coaching |
| `next` / `move on` | Advances to the next competency question |
| `did you mean latency or legacy?` *(agent asking back)* | Confirms an ambiguous dictated term before finalizing the STAR answer |

## Notes

- Model-agnostic: the workflow doesn't depend on which underlying model runs it (see [CLAUDE.md](CLAUDE.md)).
- Never fabricates specifics (numbers, names, outcomes) the user hasn't provided — asks clarifying questions instead when a story is too thin to draft.
- Source of truth for behavior is always [SKILL.md](SKILL.md); if the live skill is updated in a session, check the update back into this file's sibling before they drift.
