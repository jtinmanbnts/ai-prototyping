# Prototype → Figma → Prototype: The Full Loop

## What This Gives You

A workflow where designers, developers, and stakeholders can iterate on a live prototype without anyone starting from scratch — and without designers or clients ever touching a terminal or a Git repository. Design changes flow into code automatically. Feedback on the live prototype flows back into Figma automatically. Claude is the bridge in every direction.

**The governing principle: Vercel is the scratchpad, Figma is the record.**

Fast changes and client feedback happen directly on the live Vercel preview. Once something is approved, Figma gets updated to reflect what was decided. Figma becomes the authoritative record of design decisions — not a spec document that has to stay perfectly ahead of the code.

---

## Starting Points

You don't need to build a prototype first. There are four valid entry points:

| Starting point | When to use | Phase 2 (Design System) |
|---|---|---|
| **Existing Figma design system** | Client or team already has a component library in Figma | Skip Phase 2 entirely |
| **Client's public website URL** | New client engagement — extract their real design system from the live site | See caveat below |
| **HTML prototype file** | You already have a working prototype and want to iterate on specific screens | Extracted from file |
| **GitHub repository** | Team project with an existing codebase | Extracted from code |

### Existing Figma design system — the cleanest starting point

If a design system already exists in Figma, Claude can read it directly via the Figma MCP — components, variables, text styles, and effect styles — and start building prototype frames immediately. No extraction step, no token mapping, no guesswork.

Just point Claude at the Figma file:
> *"Build a dashboard prototype for Maria Chen using our existing design system at figma.com/design/TaFgKARVKCKhx2PEImuJ2A"*

### Public website — a caveat on CSS extraction

Claude can fetch public HTML pages, but **raw CSS values are not reliably extractable** via web fetch (the tool converts HTML to markdown, losing computed styles). In practice:

- Typography can often be inferred from rendered markup
- **Brand colors and exact hex values often need to be provided directly** — ask the client for a brand palette screenshot or style guide PDF
- If the site uses a design system package (e.g. Tailwind config in the repo), that's a better source than scraping the live site

Authenticated portals (dashboards, logged-in experiences) are fully blocked from external capture — share the HTML file or screenshots of the authenticated screens instead.

---

## Phase 1: One-Time Setup

**Who does this:** Developer, once. Nobody else ever needs to touch it.

### Step 1.1 — Create and connect the repo

```
1. Create a GitHub repo (github.com/new)
2. Install the Vercel GitHub App: github.com/apps/vercel
   → Grant access to the new repo
3. In Vercel dashboard → Add New → Project → Import the repo
4. Vercel assigns a stable URL: yourproject.vercel.app
```

Every push to `main` auto-deploys to that stable URL within ~30 seconds.

> **Common gotcha:** In the Vercel project settings, set **Output Directory** to `.` (a single dot) for root-level HTML projects with no build step. The default expects a `public/` or `dist/` folder and will fail without this.

### Step 1.2 — Add `vercel.json` to the repo root

Required for root-level HTML with client-side routing:

```json
{
  "outputDirectory": ".",
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

### Step 1.3 — Write `CLAUDE.md` at the repo root

This is the most important file in the repo. It tells Claude how to behave consistently across every session — file locations, conventions, token references, Figma details:

```markdown
# CLAUDE.md

## Project
Client prototype. Figma file key: TaFgKARVKCKhx2PEImuJ2A
Node IDs tracked in figma.config.js

## Stack
React, inline styles only, no Tailwind, no CSS modules
Design tokens at top of each component file — never hardcode hex values
Screens in src/screens/<Name>/component.js

## Figma → Code
1. Check figma.config.js for the node ID
2. Write component to src/screens/<Name>/component.js
3. Push to main — Vercel deploys automatically
4. Post the Vercel URL as a comment on the Figma frame

## Vercel feedback → Code + Figma
1. Read pinned comments on the Vercel preview (or Figma comments via API)
2. Implement approved changes in code
3. Push to main
4. Update the corresponding Figma frame to match
```

### Step 1.4 — Create `figma.config.js`

Node IDs should never live only in conversation history. Commit them so Claude can find and re-sync any screen in a future session without context:

```js
export const FIGMA = {
  fileKey: "TaFgKARVKCKhx2PEImuJ2A",
  screens: {
    MariaDashboard:  "120:11491",
    BountoosExpDesign: "13:2",
  }
};
```

### Step 1.5 — Store your Figma Personal Access Token

Claude can read Figma comments directly via the REST API, but needs a token to do so. Generate one once:

```
Figma → Settings → Security → Personal Access Tokens → Generate new token
```

Store it in your shell config so it's available every session:

```bash
# ~/.zshrc or ~/.bash_profile
export FIGMA_TOKEN="figd_xxxxxxxxxxxxxxxxxxxx"
```

Then Claude can call `$FIGMA_TOKEN` without you pasting it each session. Without this, you'll need to paste the token manually whenever comment-reading is needed.

**That's the entire setup.** The developer steps back. From here, the loop runs between designer, Claude, and Vercel.

---

## Phase 2: Extract or Build the Design System

**Goal:** Get a Figma style guide and design tokens that reflect the real visual language.

> **If you're starting from an existing Figma design system, skip this phase entirely.**

### Option A — From a client's public website

Give Claude the URL. Claude will attempt to extract brand colors, typography, and spacing. In practice:

- **What works reliably:** font families, rough spacing scale, component structure from markup
- **What often needs manual input:** exact hex color values (provide a brand palette screenshot), custom font weights
- **What doesn't work:** CSS from authenticated pages, dynamically injected styles, CSS-in-JS computed values

The most reliable pattern: provide the URL for structure + a brand palette image for colors.

### Option B — From an HTML prototype file

Share the file in the conversation. Claude reads the DOM, extracts inline styles and class references, and builds Figma frames and a style guide in one pass.

### Option C — From an existing codebase

Point Claude at the repo. It reads component files, token files, and existing patterns, then maps those to Figma variables and styles.

---

## Phase 3: Push Screens to Figma

**Goal:** Get prototype screens into Figma as proper frames, built using the style guide tokens.

### Step 3.1 — Tell Claude which screen to push
> *"Build the Bounteous Experience Design page in Figma file 3ttgkTMLrtszJ2DAW6Z553"*

### Step 3.2 — Claude builds the frame

Using the Figma MCP, Claude:
- Creates a frame on the specified page
- Lays out the screen using real colors, fonts, and spacing from the style guide
- Names every section clearly (Nav, Hero, Stats Bar, Offerings, etc.)
- Logs the node ID to `figma.config.js`

### Step 3.3 — Validate section by section

For long pages, **screenshot each section individually**, not just the full page. Full-page screenshots at reduced resolution hide:
- Text overlap (headline and body sharing the same vertical space)
- Clipped sections (frame height set to 100px when content is 500px tall)
- Grid overflow (cards extending beyond the frame width)

Fix each issue before moving to the next section.

> **Common Figma gotcha:** Sections built with `layoutMode: "VERTICAL"` (auto-layout) block manual x/y positioning of children. If a child's position isn't sticking, the parent is auto-layout. Set `frame.layoutMode = "NONE"` before repositioning.

---

## Phase 4: Implement Figma Frames as Code

**Goal:** React components that match the Figma frames 1:1.

### No-build-step approach (recommended for fast prototyping)

For a single `index.html` with React + Babel CDN:

```html
<script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
<script type="text/babel">
  // All JSX must be inline here — Babel standalone cannot load external files
  function MyComponent() { return <div>Hello</div>; }
  ReactDOM.createRoot(document.getElementById('root')).render(<MyComponent />);
</script>
```

> **Critical:** Babel standalone ignores `<script type="text/babel" src="external.js">`. All component code must be inlined inside the `<script type="text/babel">` tag. Don't try to split components into separate `.js` files with this setup.

### Multi-screen routing

With Vercel's rewrite rule sending everything to `index.html`, pathname routing works cleanly:

```jsx
function App() {
  const path = window.location.pathname;
  if (path.startsWith("/bounteous")) return <BountoosExpDesign />;
  return <MariaDashboard />;
}
```

Each screen gets its own stable URL: `yourproject.vercel.app/bounteous`

### Font naming precision

Font style names must match Figma exactly — and they vary by family:

| Font family | Correct style name | Wrong |
|---|---|---|
| Inter | `"Semi Bold"` | `"SemiBold"` |
| Inter | `"Extra Bold"` | `"ExtraBold"` |
| Nunito Sans | `"SemiBold"` | `"Semi Bold"` |
| Nunito Sans | `"ExtraBold"` | `"Extra Bold"` |

When unsure: run `figma.listAvailableFontsAsync()` in a `use_figma` call to see the exact style strings.

---

## Phase 5: The Iteration Loop

This is the day-to-day workflow once setup is done. It runs in three directions.

---

### Direction 1: Figma → Vercel

**Who drives it:** Designer

The designer iterates on a frame in Figma and marks it **"Ready for dev"** using Figma's Dev Mode status flag. That's the only action required.

```
Designer iterates frame in Figma
            │
            │  Marks "Ready for dev" in Dev Mode
            ▼
Figma API: devStatus = READY_FOR_DEV
            │
            │  Claude reads frame via Figma MCP
            │  Updates component in repo
            │  git push → main
            ▼
Vercel auto-deploys in ~30s
            │
            │  Claude posts Vercel URL as Figma comment
            ▼
Designer reviews at stable URL
```

> **When Dev Mode isn't needed:** If Claude is doing both the design work (building Figma frames) and the development work (writing the React component), the handoff step is skipped — Claude already has full context. Dev Mode is most valuable when a human designer is creating frames and signaling readiness to Claude or a developer.

---

### Direction 2: Stakeholder Feedback → Code + Figma

**Who drives it:** Designer or stakeholder

There are two places stakeholders can leave feedback:

**On the Vercel prototype** — Vercel's feedback toolbar lets anyone leave pinned visual comments on the deployed preview. No account needed for viewers.

**In the Figma file** — Stakeholders with view access leave comments directly on frames. Claude reads these via the Figma REST API.

```
Stakeholder leaves comment
  (in Vercel preview toolbar OR pinned to a Figma frame)
            │
            │  "Move the carousel to the right half of the section"
            ▼
Claude reads comments
  (Vercel: from the feedback dashboard)
  (Figma: via REST API using FIGMA_TOKEN)
            │
            │  Implements changes in code + Figma frame
            │  git push → main → Vercel redeploys
            ▼
Vercel: updated live       Figma: frame reflects change
```

> **Figma comment reading requires the REST API** — the Figma MCP's built-in tools cover design data (nodes, screenshots, components) but not comments. Reading comments requires a separate API call: `GET api.figma.com/v1/files/{key}/comments` with your `FIGMA_TOKEN` header.

---

### Direction 3: Prompt → Everywhere

For quick changes that don't need a formal feedback cycle:

> *"Change the stats bar background from indigo to dark indigo"*
> *"Implement approved comments on the Bounteous frame"*

Claude implements the change in code, deploys to Vercel, and updates the affected Figma frames. Both stay in sync from a single instruction.

---

## The Full Loop

```
                    FIGMA
                  (the record)
                 ↗           ↖
    Direction 1               Direction 2
   Designer marks           Figma comments
   "Ready for dev"          or Vercel feedback
              ↙               ↘
           CLAUDE
         (the bridge)
              ↕
           VERCEL
         (the scratchpad)
```

**Vercel is the scratchpad** — fast changes, client feedback, and stakeholder review all happen here.

**Figma is the record** — every approved change is reflected back. Figma shows what was decided.

**Claude is the bridge** — reads from both, writes to both, keeps them in sync regardless of which direction a change originates.

---

## Revision History: Before/After Snapshots

For any client-facing project, maintain a **Revision History page** in the Figma file. Every time a round of feedback is implemented, Claude adds a new entry:

```
Revision History page
├── Rev 01 — Industries Section · Apr 24, 2026
│   ├── Comment (verbatim, from Figma API)
│   ├── BEFORE · v1  (reconstructed snapshot of original)
│   └── AFTER  · v2  (snapshot of implemented change)
├── Rev 02 — Hero Section · May 1, 2026
│   └── ...
```

This gives stakeholders a full audit trail of design decisions — what was requested, what changed, and when. It also protects against "why did we change this?" conversations down the line.

To trigger it, tell Claude:
> *"Implement approved comments and track the changes as a before/after revision"*

---

## What Each Role Actually Does

| Role | Tools | Git involvement |
|---|---|---|
| **Designer** | Figma + browser (Vercel URL) | None |
| **Stakeholder / client** | Browser only (Vercel feedback toolbar or Figma view access) | None |
| **Claude** | Figma MCP + Figma REST API + editor + Vercel CLI | Pushes to main or feature branch |
| **Developer** | Repo setup + optional PR review | One-time setup + review if needed |

---

## Team Considerations

### Multiple designers, multiple screens

Each screen maps to its own Figma node ID and its own file path in the repo. Multiple designers can iterate different screens simultaneously with no conflicts. Claude handles each independently.

### Code review before deploy

If the team wants oversight before anything goes live, Claude pushes to a feature branch instead of `main`. Vercel auto-generates a preview URL for every branch, so the designer can review before merge. The developer reviews the PR, merges, and the stable URL updates.

For pure client prototyping, pushing straight to `main` is usually the right call — speed matters more than process at that stage.

### When to involve the developer

- **Initial setup:** Always (repo, Vercel, CLAUDE.md, figma.config.js)
- **Day-to-day iteration:** Not needed
- **Production handoff:** Developer reviews component library, cleans up code, sets up proper deployment pipeline

---

## What Changes at Each Scale

| | Solo / client demo | Small team | Full team |
|---|---|---|---|
| **Design system source** | Client URL + brand palette image | Built in Figma, tokens in repo | Tokens Studio syncs Figma → repo automatically |
| **Trigger** | Tell Claude directly | Figma "Ready for dev" status | Figma Dev Mode + GitHub Action polls API |
| **Deployment** | Vercel CLI, instant | Push to main, auto-deploy | Push to main, auto-deploy, PR previews |
| **Feedback reading** | Claude reads Figma comments via REST API on demand | Same + Vercel feedback toolbar | Automatic polling via GitHub Action |
| **Figma sync** | Claude updates frames on request | Claude updates frames after every deploy | Claude updates frames after every deploy |
| **Revision tracking** | Before/after snapshots in Figma on request | Same, automated per feedback cycle | Same |
| **Code review** | Not needed | Optional | PR review before merge |

---

## Prerequisites Checklist

### Always required
- [ ] Claude Code with Figma MCP connected
- [ ] Figma file with a page designated for prototype frames
- [ ] `figma.config.js` committed with node IDs
- [ ] `CLAUDE.md` committed with stack + naming conventions

### For comment reading (recommended)
- [ ] Figma Personal Access Token generated (Settings → Security → Personal Access Tokens)
- [ ] Token stored as `FIGMA_TOKEN` in shell config (`~/.zshrc` or `~/.bash_profile`)

### For the Vercel deployment
- [ ] GitHub repo created
- [ ] Vercel GitHub App installed at github.com/apps/vercel
- [ ] Repo connected via Vercel dashboard → Add New → Project
- [ ] `vercel.json` committed with `outputDirectory: "."` and rewrite rule
- [ ] Output Directory set to `.` in Vercel project settings
- [ ] Stable URL shared with designers and stakeholders (bookmark it — it never changes)

### For Dev Mode handoff (team workflow)
- [ ] Figma paid seat (Dev Mode requires Professional plan or above)
- [ ] Frames marked "Ready for dev" by designer before Claude implements

---

## Advanced: Closing the Loop Further

**Tokens Studio + Style Dictionary**
A Figma plugin that stores design tokens as JSON and syncs them to the repo on every save. Design token changes in Figma flow to code automatically — no manual token file updates.

**GitHub Action to poll Figma Dev Mode**
A lightweight action that checks the Figma API every few minutes for frames with `devStatus: READY_FOR_DEV` and triggers Claude automatically — removing the need to tell Claude to "check Figma." The loop becomes fully autonomous.

**Prompt input embedded in the prototype**
A floating "request a change" input built into the Vercel prototype itself. Stakeholders type what they want while looking at the live screen. Claude reads the request, updates code and Figma simultaneously. No Figma access required for the stakeholder.

**Automated revision history**
Wire the before/after snapshot creation into the standard feedback cycle so it runs every time without being asked. Every client session ends with a Revision History page that's always current.
