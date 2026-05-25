# CMS Developers AI Playbook
![Version](https://img.shields.io/badge/version-v1.0-blue)

This builds on the broader work in the
**[Offshorly AI Excellence Playbook](https://github.com/maekooffshorly/ai-excellence-playbook)**. Where the linked repo covers the company's full system, this repo contains specific guidance for the Offshorly CMS Developers.

## Building WordPress Sites on the Offshorly Boilerplate with AI Assistance

> **Audience:** CMS Developers building client sites on the Offshorly WP Boilerplate (Bedrock + Timber + ACF Pro + UIkit 3) using Claude Code's AI assistance.
>
> **Goal:** A repeatable, AI-driven workflow that turns a Figma file into a production-ready WordPress site.
>
> **What this doc covers:** the steps *you* take as a developer — what to prep, what to prompt, what to check. The AI handles boilerplate conventions internally (see the separate **Boilerplate Standards** doc for those).

---

## 1. The Mental Model

The boilerplate handles the boring parts: WordPress install, Composer deps, Timber wiring, asset compilation, ACF registration. **What you build per project** is:

1. A brand-specific **design system** (tokens in SCSS).
2. A library of **ACF block modules** that match the Figma design.
3. **Pages assembled** from those modules with real content.

Your job in this workflow:

- Provide the right inputs (Figma URLs, brand decisions, content).
- Verify outputs against the design.
- Approve anything destructive or that touches shared state.

Treat the AI like a junior dev with good instincts but no project-specific context. You give it Figma, it gives you working code. You catch the visual mismatches.

---

## 2. The Build Loop

```
┌──────────────────────────────────────────────────────────────────┐
│  1. Design tokens     →  populate _variables.scss + _typography  │
│  2. Per Figma section →  AI builds an ACF block module           │
│  3. Per Figma image   →  AI extracts + imports via WP-CLI        │
│  4. Per page          →  AI assembles modules into a page        │
│  5. Iterate           →  AI tweaks individual modules            │
│  6. Verify            →  you check in browser                    │
└──────────────────────────────────────────────────────────────────┘
```

Each step has a standard prompt pattern. Sections 4–9 walk through each one.

---

## 3. Prerequisites (Before You Open the AI)

A few things need to be running locally before the AI can help. Knock these out in order — none of them take long, but skipping one will make later prompts fail in confusing ways.

### 3.1 Boot the local environment

The AI can't debug Docker issues over a chat, so get Lando up yourself first:

```bash
# from the repo root
lando start
lando composer install

# from the theme folder: web/app/themes/[theme-name]
lando composer install
npm install
npm run build
```

Confirm the site loads at `https://[project].lndo.site`. Fix any Lando errors before continuing.

### 3.2 Open Figma Desktop and connect the MCP

The AI pulls designs and images from Figma through the **Figma MCP** (Model Context Protocol) server, which runs locally at `http://localhost:3845`. This only works when all of the following are true:

1. **Figma Desktop is open and running** — the browser version of Figma doesn't expose the local MCP server.
2. **The project file is open in Figma Desktop** — the MCP can only see files that are currently open in a tab.
3. **The Figma MCP integration is enabled in your AI tool.**
   - In Claude Code, run `/mcp` to list connected servers. You should see `figma` (or `claude_ai_Figma`) listed.
   - If it's not listed, follow the Figma MCP setup instructions (one-time per machine) before continuing.

Quick sanity check: paste a Figma node URL into the AI and ask "can you get a screenshot of this?". If it returns one, the MCP connection is healthy. If it errors with "can't reach localhost:3845" or similar, close and reopen Figma Desktop, then re-check `/mcp`.

> **Why this matters:** every prompt in this playbook that mentions a Figma URL depends on the MCP. If the MCP is down, the AI will either fail silently or hallucinate design details.

### 3.3 Confirm WP admin access

Log into `https://[project].lndo.site/wp/wp-admin` once before you start so you can spot-check ACF syncs, page renders, and media uploads as you go. The default boilerplate admin credentials are in the project README.

### 3.4 Quick check on `CLAUDE.md`

The boilerplate ships with a `CLAUDE.md` file at the repo root. This is the AI's onboarding document. Before starting, open it and confirm:

- Project name and theme folder name are correct.
- Local dev URL matches your Lando setup.
- Any brand-specific notes are accurate.

If you change the theme structure later, update `CLAUDE.md` to match.

---

## 4. Step 1 — Establish the Design System

**Do this before building any block.** Every module references these tokens. Skipping ahead means refactoring hardcoded values later.

### Prompt

```
Here's the Figma file: [FIGMA_URL_TO_STYLE_GUIDE_PAGE]

Extract the brand tokens (colors, typography sizes, spacing, font families)
and populate the global SCSS files.

Use semantic names matching the brand (e.g. $brick, $leaf), not generic
ones like $primary / $secondary.
```

### What you check

- [ ] Token names reflect the **brand**, not their role.
- [ ] All Figma colors are present.
- [ ] Global typography for `h1`–`h6`, `p`, and `.eyebrow` is set.
- [ ] No values are duplicated across the variables and typography files.

---

## 5. Step 2 — Building a Block Module

The core loop. Each Figma section → one ACF block module. You'll do this dozens of times per project.

### Prompt

```
Here's the Figma node for the [BLOCK_DESCRIPTION] section:
[FIGMA_URL_WITH_NODE_ID]

Build it as an ACF block module called [block-name-kebab].
Use existing global tokens from _variables.scss.
```

### What you check

- [ ] Block appears in the ACF Sync screen and syncs cleanly.
- [ ] Block can be added to a page via the editor.
- [ ] Visual matches Figma at desktop **and** mobile breakpoints.
- [ ] Content fields are editable and render correctly.
- [ ] Background color toggle works as expected.

If the block doesn't appear in the ACF sync screen, jump to Section 12 (Troubleshooting).

---

## 6. Step 3 — Importing Figma Images

Figma images don't move into WordPress automatically. The AI handles the extraction → import flow, but **you** need to know when to step in.

### Prompt

```
This block uses the image at Figma node [FIGMA_URL_WITH_NODE_ID].

Extract it, save to temp-assets, import via WP-CLI, and assign the
attachment ID to the [field_name] ACF field on the [page/block].
```

### When image extraction fails

If a Figma asset is built from multiple layers or grouped elements, the AI will stop and tell you something like:

> ⚠️ **Image extraction failed for [asset name].** This asset is made up of multiple layers and cannot be auto-downloaded as a single file.

**Don't try to fix this in code.** It's a Figma problem. Options:

1. Ask the designer to flatten the asset in Figma and re-export.
2. Manually export the asset as PNG/JPG yourself and drop it into `web/app/themes/[theme]/temp-assets/`, then tell the AI to import that file instead.

### What you check

- [ ] Image appears in the WP Media Library.
- [ ] Image renders on the page (not a broken `<img>` tag).
- [ ] Image is correctly sized and not blurry on retina displays.

### Tell your designers up front

- **Flatten all images and logos before handoff.** No layered groups, no masks, no live effects.
- **Use web-optimised, correctly-sized images.** Don't ship 4K hero images that should display at 1200px.

Flagging this on day one of the project saves hours later.

---

## 7. Step 4 — Assembling Pages

Once you have a library of modules, pages become assembly.

### Prompt

```
Build the [PAGE_NAME] page using existing modules.
Figma reference: [FIGMA_URL_FOR_PAGE]

Map each Figma section to an existing module. If a section needs a
module that doesn't exist yet, STOP and tell me before creating
anything new.
```

### Why "stop and tell me" matters

Without it, the AI may auto-create a near-duplicate of an existing module when a styling toggle would have done the job. You want to make that call yourself — extend an existing module, or genuinely build a new one. Either is fine; the decision should be conscious.

### What you check

- [ ] Page renders without PHP errors or missing modules.
- [ ] All modules pulled real content from Figma (not lorem ipsum).
- [ ] All images attached.
- [ ] Page slug, title, and meta are correct.

---

## 8. Step 5 — Iterating on Existing Modules

First builds are never final. When you need a tweak:

### Prompt

```
Update the [block-name] module — change [SPECIFIC_THING].
Don't touch anything else in the module.
```

### Why "don't touch anything else" matters

AI sometimes "improves" code it thinks looks wrong while making your requested change. Explicit scoping prevents drive-by refactors and surprise regressions. If you want broader changes, ask in a separate prompt.

### Adding a new ACF field

```
Add a [FIELD_TYPE] field called [field_name] to the [block-name] module.
Then update the Twig template to render it.
```

### What you check

- [ ] Only the thing you asked about changed (run `git diff`).
- [ ] The block still appears in the ACF sync screen.
- [ ] No unrelated styling broke.

---

## 9. Step 6 — Verifying AI Work

The AI cannot reliably verify its own visual output. **This step is non-negotiable.**

### What the AI can verify on its own

- Syntax errors, lint failures.
- Whether files were written.
- Whether ACF fields are registered.

### What only you can verify

- Visual fidelity to Figma (spacing, alignment, typography hierarchy).
- Responsive behaviour at common breakpoints.
- Content overflow / text wrapping with real-length content.
- Hover and interactive states.
- Cross-browser quirks.

### The minimum verification workflow

1. `npm start` running in watch mode (or `npm run build` after each change).
2. Refresh the page in the browser.
3. Compare side-by-side with Figma at desktop and mobile widths.
4. If using Claude Code, you can ask `/verify` — it'll start the dev server and screenshot the change.

If the visual is wrong, go back to the iterate prompt (Section 8) and describe what's off. Don't try to fix CSS yourself if you can describe the fix to the AI faster.

---

## 10. What You Handle, Not the AI

Some decisions and actions stay with you. If the AI does any of these without asking, stop the session and review.

| Action | Why it's yours |
| :--- | :--- |
| Approving brand tokens | Subjective; brand-team sign-off |
| Approving copy / final content | Voice and tone judgement |
| Pushing to production / staging | Destructive |
| Deleting attachments or content | Destructive |
| Modifying `.env` or environment configs | Breaks other devs' setups |
| Calling out a Figma asset that needs flattening | AI flags it, you talk to the designer |
| Accepting "this is close enough" visual deviations | Designer's call |

---

## 11. Prompt Template Appendix

Copy-paste templates. Replace `[BRACKETS]` with real values.

### 11.1 Bootstrap a new project's design system

```
Here is the Figma file for [PROJECT_NAME]: [FIGMA_URL]

Extract the brand tokens — colors, typography, spacing — from the style
guide page and populate the global SCSS files.

Use brand-semantic variable names (e.g. $brick, $sage) not role-based
ones (e.g. $primary, $secondary).
```

### 11.2 Build a block module from Figma

```
Build an ACF block module named [block-name-kebab] from this Figma node:
[FIGMA_URL_WITH_NODE_ID]

Use existing global tokens from _variables.scss.

If any image in the design fails to extract because it's layered or
composite, stop and tell me which one.
```

### 11.3 Import a single Figma image

```
Extract the image at Figma node [FIGMA_URL_WITH_NODE_ID].
Save it to temp-assets with a descriptive filename, import it into WP,
and assign the attachment ID to the [field_name] ACF field on
[the page / block instance].
```

### 11.4 Assemble a page from existing modules

```
Build the [PAGE_NAME] page from this Figma reference:
[FIGMA_URL_FOR_FULL_PAGE]

Map each Figma section to an existing block module from /modules/.
If a section requires a module that doesn't exist, STOP and ask me
before creating it.

Generate a PHP script that creates the page programmatically.
Run it and confirm the page renders.
```

### 11.5 Add a field to an existing module

```
Add a [FIELD_TYPE] field called [field_name] to the [block-name] module.
Update the Twig template to render it.
Don't touch anything else in the module.
```

### 11.6 Tweak a module's styling

```
Update only the SCSS for the [block-name] module.
Change [SPECIFIC_THING — e.g. "the headline color to $brick on the
dark variant"].
Do not change the Twig template, fields.json, or any other module.
```

### 11.7 Get an image flattened by the designer

(Not for the AI — this is a Slack/email template for the designer.)

```
Hey [DESIGNER], the [ASSET_NAME] in [FIGMA_URL_TO_NODE] is made up of
multiple layers / groups / masks, so our tooling can't extract it as a
single image. Could you flatten it into one layer in Figma (or export
it as a PNG and send it over)? Thanks!
```

---

## 12. Troubleshooting

The pattern: **describe the symptom, name the file/block, let the AI fix it.** You don't need to know which rule was broken — that's in the standards doc.

- **Block doesn't appear in ACF sync screen** — Tell the AI: *"the block isn't appearing in the sync screen, please check the fields.json"*
- **Block appears but shows weird unrelated fields** — Tell the AI: *"the block's editor sidebar is showing fields from other groups, please check the styles group"*
- **Wrapper styles aren't applying** — Tell the AI: *"the block's wrapper styles aren't applying — check the wrapper classes"*
- **Image import failed** — Re-run the image prompt. If it still fails, check whether the Figma asset is layered (Section 6).
- **Figma URLs in chat suddenly return 404** — Reopen Figma Desktop, confirm the file is open, then re-paste the URL in a fresh prompt.
- **AI says it imported an image but it's not in the Media Library** — Tell the AI: *"that image isn't showing in the media library, please re-import it"*
- **Page renders but blocks are blank** — Tell the AI: *"the [block-name] block is rendering blank on [PAGE_NAME], please check the field key mappings"*
- **Module SCSS has hardcoded colors** — Tell the AI: *"there are hardcoded colors in the [block-name] SCSS, please refactor to use the global tokens"*
- **`/mcp` shows Figma is disconnected** — Quit and restart Figma Desktop, then run `/mcp` again. If still disconnected, re-run the Figma MCP setup.

---

## 13. Final Notes

The AI is a force multiplier, not a replacement for engineering judgement. Read its diffs. Verify visually. Ask questions when a decision feels load-bearing.

When in doubt, return to the build loop:

> **Tokens → Modules → Images → Pages → Iterate → Verify**
