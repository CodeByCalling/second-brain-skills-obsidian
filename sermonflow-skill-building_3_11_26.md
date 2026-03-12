# SermonFlow Skill-Building Notes — March 11, 2026

Reference notes from analyzing the `second-brain-skills-obsidian` repo and mapping its skills to the SermonFlow project.

---

## Source Repo: second-brain-skills-obsidian

A collection of Claude Code skills for knowledge work: presentations, brand systems, MCP integrations, video creation, SOPs, and skill development. Key pattern: **progressive disclosure** — skills load detailed instructions only when triggered.

### Skills Available

1. **PPTX Generator** — Programmatic slides via python-pptx with 16 layouts + 5 carousel layouts
2. **Brand Voice Generator** — Interactive brand identity + voice creation (outputs brand.json, config.json, brand-system.md, tone-of-voice.md)
3. **MCP Client** — Universal MCP server connector with on-demand tool schema loading
4. **SOP Creator** — Runbooks, playbooks, and technical docs
5. **Skill Creator** — Guide for building new Claude Code skills
6. **Remotion** — Programmatic video creation (NOT relevant to SermonFlow)

---

## PPTX Generator — Blueprint for custom-engine.ts (Phase 8)

### Current SermonFlow PPT System

| Aspect | Current State |
|--------|--------------|
| Library | PptxGenJS (JavaScript, runs in Node) |
| Layouts | 9 slide types, all follow the **same pattern**: background image + semi-transparent panel + text |
| Themes | 12 themes x 4 layouts = 36 combos (config-driven) |
| Visuals | Professional but repetitive — every slide is "panel over photo" |
| Branding | Church colors exist in DB but logo isn't integrated yet |
| Visual score | ~6.5/10 |

### What the PPTX Generator Skill Offers

| Aspect | Skill Approach |
|--------|---------------|
| Library | python-pptx (Python, runs via `uv`) |
| Layouts | 16 distinct layouts with **unique visual structures** per layout |
| Visuals | Floating cards with shadows, circular hero with trigonometry, stats with large numbers, diagonal shapes, quote styling |
| Branding | 10-color system + 3 fonts + design philosophy doc |
| Visual score | ~7.5-8/10 |

### Why You Can't Just Drop It In

1. **Different language** — Skill uses Python (`python-pptx`), SermonFlow runs Node.js (`PptxGenJS`). Need a Python sandbox (Docker/Lambda) to execute server-side.
2. **Different purpose** — Skill generates general presentations. SermonFlow generates sermon-specific slides with theological constraints.
3. **Different flow** — Skill is interactive (Claude picks layouts per session). SermonFlow is automated (Claude returns JSON, template engine renders deterministically).

### What to Borrow for Phase 8 (custom-engine.ts)

| Pattern | How it helps |
|---------|-------------|
| **Cookbook pattern** | Each sermon slide type becomes a self-contained python-pptx script with frontmatter documenting when/how to use it |
| **Visual-first decision tree** | Claude picks the best visual layout for each sermon section instead of uniform panel-over-background |
| **Batch + validate loop** | Generate 5 slides, validate, continue — prevents token blowouts on 20+ slide sermons |
| **Brand variable mapping** | `ChurchBrand` model maps to `BRAND_*` placeholder system |
| **Combine script** | Batch-combining code with background-fix handles the bug you'd hit merging slide batches |

### Layout Variety Mapping for Sermon Slides

| Sermon Slide Type | Current Look | Could Become |
|---|---|---|
| `point` (main sermon point) | Panel + big number + text | **Giant-focus style** — massive word/phrase, minimal text |
| `subpoint` (3-4 bullets) | Panel + bullet list | **Floating cards** — each bullet becomes a card with shadow |
| `scripture` | Panel + quote marks + text | **Quote slide** — elegant attribution, accent bar |
| `transition` | Panel + short phrase | **Bold diagonal** — dynamic shape with high-energy text |
| `application` | Panel + text | **Two-column** — "The Problem" vs "The Response" |
| `closing` | Panel + text | **Section break style** — dramatic, minimal, centered |

### Immediate Win (No Python Required)

Port the visual variety concept into existing PptxGenJS template engine. Give each of the 9 slide types a **unique visual structure** instead of the uniform panel approach. Add:
- Accent bars (thin colored lines for structure)
- Shadow layers behind cards (offset rectangles)
- Subtle dividers between sections
- Small geometric shapes as visual anchors

All achievable in PptxGenJS.

### Color System Upgrade Needed

| SermonFlow currently | Skill uses |
|---|---|
| 4 colors: `bg`, `primaryText`, `accent`, `secondary` | 10 colors: + `card_bg`, `card_bg_alt`, `accent_secondary`, `accent_tertiary`, `code_bg`, `background_alt` |

Adding 3-4 more color fields to `ChurchBrand` would unlock much richer PPT output — cards, stats, accent bars all need distinct colors.

---

## Brand Voice Generator — Level 2 Personalization

### The Connection

SermonFlow's Level 2 personalization (`/profile` page) asks pastors to describe their preaching voice. The Brand Voice Generator solves the same problem: **people don't know how to describe their own voice.**

### Voice Discovery Questions to Adapt

The skill asks 6 questions. Adapted for pastors:

1. **"Describe your preaching personality in 3 words"**
   - e.g., "Passionate, Scriptural, Practical" or "Gentle, Story-driven, Conversational"

2. **"Who are 2-3 preachers whose style you admire?"**
   - Helps Claude understand the vibe without the pastor having to articulate it

3. **"What words or phrases do you naturally reach for in sermons?"**
   - Pet phrases, intensifiers, transitions they use often

4. **"What kind of tone do you want to AVOID?"**
   - "Too academic", "prosperity gospel language", "overly emotional"

5. **"Do you prefer short punchy sentences or longer flowing explanations?"**
   - Sentence rhythm preference

6. **"When explaining something complex, how do you approach it?"**
   - Analogies? Stories? Walk through the Greek? Practical examples first?

### Preaching Voice Archetypes (from Voice Templates)

The skill has 5 voice templates. Adapted as preaching archetypes:

| Archetype | Description | Maps to Skill Template |
|---|---|---|
| **The Teacher** | Walks through text methodically, explains context and original language | Technical Educator |
| **The Shepherd** | Gentle, pastoral, focuses on comfort and application | Calm Authority |
| **The Exhorter** | Direct, challenging, calls to action and accountability | Builder's Perspective |
| **The Storyteller** | Weaves narratives, uses vivid illustrations, draws you in | Approachable Expert |
| **The Prophet** | Challenges assumptions, confronts cultural norms with Scripture | Contrarian Thinker |

### Implementation Path

- Add archetype selection to Level 2 profile (radio buttons, not freeform)
- Add 3-4 guided voice questions below the archetype
- Generate a structured voice profile from answers
- Feed into `buildManuscriptPrompt()` as structured data, not freeform text
- Store in `PastorProfile` model (new fields or JSON column)

---

## SOP Creator — Pre-Launch Operations Docs

### Priority SOPs to Create

**1. Payment Webhook Flow (HIGHEST PRIORITY)**
- Stripe webhook → credit transaction → wallet update
- PayMongo e-wallet webhook (GCash, Maya differences)
- Double-fire prevention
- Silent failure detection
- Credit refund when sermon generation fails mid-stream

**2. Denomination Management Runbook**
- Adding denomination #10: prompts, DB enums, onboarding UI, config
- Editing existing denomination theological framing
- Testing denomination-specific output

**3. Incident Response**
- Claude API down during sermon generation — pastor sees streaming stop
- localStorage recovery via `sermonflow_pending_sermon`
- Supabase storage fails during PPT upload
- Support contact flow

**4. Deployment Runbook**
- Vercel deployment checklist
- Supabase migration process
- Environment variable management
- Rollback procedure

### SOP Creator's Key Patterns to Follow

- **Definition of Done checklist FIRST** (at the top, not buried at the bottom)
- **Warnings BEFORE the dangerous step** (not after)
- **Specific numbers** instead of "as needed" or "regularly"
- **Action-first steps** (verbs, not descriptions)
- **Clear decision points** (if X, then Y — not "handle based on priority")

---

## Skill Creator — SermonFlow Dev Skills

### Skills to Build

**1. Sermon Prompt Engineering Skill**
- Knows prompt priority order: Anti-Prefs > Custom Theology > Denomination > Style > Background > Influences > Congregation > Books
- All 9 denomination profiles and theological boundaries
- Manuscript format rules (max word counts, slide extraction limits)
- Use case: "improve the Baptist prompt" → Claude already has full context

**2. Theme Creator Skill**
- Knows `ThemeConfig` type shape and `themes.ts` structure
- The 4 background layout types and base64 image system
- Color contrast requirements for readability
- Use case: "create a Lent theme" → generates full config + background data

**3. Payment Debugger Skill**
- Stripe and PayMongo webhook payload formats
- `CreditTransaction` model and status flows
- Common failure modes and fixes
- Test credentials and test card numbers

### Skill Anatomy (from Skill Creator)

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description, triggers)
│   └── Markdown instructions
└── Bundled Resources (optional)
    ├── scripts/     - Executable code
    ├── references/  - Documentation loaded as needed
    └── assets/      - Files used in output
```

Key principles:
- Only add context Claude doesn't already have
- Progressive disclosure: metadata always in context, body when triggered, resources as needed
- Set appropriate degrees of freedom (match specificity to task fragility)

---

## MCP Client — Dev Workflow (Post-Launch)

| Connection | What it does |
|---|---|
| **GitHub MCP** | Manage SermonFlow issues/PRs from Claude Code |
| **Zapier MCP** | Notifications: new pastor signup, payment webhook failure |
| **Gmail MCP** | Test Resend email templates, draft support responses |
| **Google Calendar MCP** | Schedule phase milestones and deadlines |

Setup: Copy `example-mcp-config.json` → `mcp-config.json`, add API keys, test each tool, document gotchas in CLAUDE.md.

---

## Implementation Priority

| Priority | Action | When | Impact |
|---|---|---|---|
| 1 | **SOP Creator** → Document payment webhook flow (Stripe + PayMongo) | Before launch | High — highest-risk area |
| 2 | **Brand Voice Generator** → Adapt voice discovery into Level 2 profile | When building /profile | High — solves "pastors can't describe their voice" problem |
| 3 | **PPTX Generator** → Port visual variety into existing PptxGenJS engine | Phase 8 prep | High — breaks the monotone panel pattern |
| 4 | **PPTX Generator** → Build custom-engine.ts using cookbook architecture | Phase 8 | High — premium PPT tier |
| 5 | **Skill Creator** → Build sermon prompt engineering skill | Ongoing | Medium — saves context per session |
| 6 | **MCP Client** → Zapier notifications for signups/payments | Post-launch | Low — convenience |

---

## Key Files Reference

### In second-brain-skills-obsidian

- `.claude/skills/pptx-generator/SKILL.md` — Full PPTX generation workflow
- `.claude/skills/pptx-generator/cookbook/*.py` — 16 layout templates (study floating-cards, circular-hero, stats for sermon adaptation)
- `.claude/skills/pptx-generator/brands/template/` — Brand template files to adapt
- `.claude/skills/brand-voice-generator/SKILL.md` — Voice discovery process
- `.claude/skills/brand-voice-generator/references/voice-templates.md` — 5 voice archetypes
- `.claude/skills/mcp-client/SKILL.md` — MCP client setup
- `.claude/skills/mcp-client/references/example-mcp-config.json` — Config template

### In sermonflow-site

- `src/lib/ppt/template-engine.ts` — Current PPT generation (PptxGenJS)
- `src/lib/ppt/custom-engine.ts` — Stub for premium tier (`NOT_IMPLEMENTED`)
- `src/lib/ppt/slide-types.ts` — 9 sermon slide type definitions
- `src/config/themes.ts` — 12 themes x 4 layouts
- `src/lib/ai/prompt-builders.ts` — Manuscript + slide JSON prompts
- `src/lib/ai/denomination-prompts.ts` — 9 denomination theological profiles
- `docs/PPT-ARCHITECTURE.md` — Full PPT system design
- `docs/PERSONALIZATION.md` — Level 1-3 personalization system
- `docs/THEOLOGY.md` — Theological positioning (non-negotiable requirements)

---

## Not Relevant to SermonFlow

| Skill | Why skip |
|---|---|
| **Remotion** | No video content needed — sermons are slides + manuscript |
| **LinkedIn Carousels** | Wrong audience — pastors aren't posting sermon carousels |
