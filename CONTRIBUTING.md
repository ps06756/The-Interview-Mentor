# Contributing to The Mentor

Thank you for helping make interview preparation more accessible! This guide explains how to contribute new skills, improve existing ones, and maintain quality standards.

---

## Quick Start for Contributors

1. **Fork** the repository
2. **Copy** the template:
   ```bash
   cp -r templates/skill-template agents/{category}/{skill-name}-interviewer
   ```
3. **Fill in** the template (see Quality Standards below)
4. **Test** your skill with `claude --plugin-dir ./agents/{category}`
5. **Submit** a pull request

---

## Skill Categories

| Directory | What goes here |
|-----------|---------------|
| `agents/swe-i/` | Entry-level coding (Easy to Easy-Medium) |
| `agents/swe-ii/` | Mid-level algorithms (Medium to Medium-Hard) |
| `agents/data-engineer/` | Data engineering roles |
| `agents/systems-design/` | System design interviews |
| `agents/devops-sre/` | DevOps and SRE roles |
| `agents/ml-engineer/` | Machine learning engineering |
| `agents/ai-pm/` | AI product management |
| `agents/debugging/` | Debugging and incident response scenarios |
| `agents/behavioral/` | Behavioral and leadership interviews |
| `agents/meta/` | Meta-skills (problem decomposition, approach selection) |

---

## Skill Structure

Every skill must have this directory structure:

```
skill-name-interviewer/
├── SKILL.md                      # Main interviewer instructions
└── references/
    ├── problems.md               # Problem bank with walkthroughs
    └── remotion-components.md    # Animation components (Remotion)
```

---

## Quality Standards

### Required Sections in SKILL.md

Every SKILL.md must include ALL of these sections:

1. **YAML frontmatter** with `name:` (must match directory name) and `description:`
2. **## Persona** with ### Communication Style
3. **## Activation** — "immediately begin Phase 1" instruction
4. **## Core Mission** — numbered list of focus areas
5. **## Interview Structure** — 4 phases with timing
6. **### Adaptive Difficulty** — how to adjust for candidate level
7. **### Scorecard Generation** — end-of-interview evaluation instruction
8. **## Interactive Elements** — at least 1 ASCII diagram
9. **## Hint System** — at least 3 problems, each with 4 hint levels
10. **## Evaluation Rubric** — table with `| Area | Novice | Intermediate | Expert |` header
11. **## Resources** — Essential Reading, Practice Problems, Tools to Know
12. **## Interviewer Notes** — guidance for the AI interviewer
13. **## Additional Resources** — links to references/problems.md and references/remotion-components.md

### Content Quality Checklist

- [ ] Persona is distinct and memorable (not generic "friendly interviewer")
- [ ] 3-4 problems with realistic interview difficulty
- [ ] All 4 hint levels are truly progressive (Level 1 doesn't give away the approach)
- [ ] Follow-up constraints on each problem ("Now do it in O(1) space")
- [ ] At least 2 ASCII diagrams
- [ ] Evaluation rubric uses Novice/Intermediate/Expert format
- [ ] Resources reference real books, tools, and practice problems
- [ ] No emojis in section headers
- [ ] `name:` in frontmatter matches directory name

### Testing Your Skill

Before submitting a PR, test your skill:

```bash
# Load your skill category
claude --plugin-dir ./agents/{category}

# Then ask Claude to use your skill
> "Use the {skill-name}-interviewer skill and start my mock interview."
```

Verify that:
- The interviewer starts Phase 1 immediately (no preamble)
- The persona voice is consistent throughout
- Hints are accessible when asked
- The scorecard is generated at the end

---

## Pull Request Template

When submitting a PR, include:

1. **Skill name** and category
2. **Target role** and difficulty level
3. **What's covered** (3-5 bullet points)
4. **Testing notes** (how you verified it works)
5. **Screenshot or transcript** of a test session (optional but helpful)

---

## Improving Existing Skills

Found a bug, inaccuracy, or gap in an existing skill? We welcome improvements:

- **Technical errors**: Fix and submit a PR with the correction
- **Missing topics**: Add problems or hint levels
- **Weak personas**: Suggest a more distinctive character
- **Better diagrams**: Replace or add ASCII visualizations

---

## Code of Conduct

Be respectful, constructive, and focused on making interview prep better for everyone.
