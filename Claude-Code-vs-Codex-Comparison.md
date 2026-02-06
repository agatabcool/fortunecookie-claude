# Claude Code vs Codex: Fortune Cookie App Comparison

## The Experiment

Both tools were given the same initial prompt:

> "Build a single-page web app that's a fortune cookie. Clean, minimal design — a nice illustrated cookie on a white/cream background. You click the cookie and it cracks open with a satisfying animation to reveal an absurd fortune that's loosely inspired by real current news headlines..."

Both were then asked to deploy to Vercel and add news links to fortunes. From there, the two sessions diverged significantly.

---

## Conversation & Interaction

| Metric | Claude Code | Codex |
|--------|------------|-------|
| **User prompts** | 5 | 12 |
| **Total back-and-forth exchanges** | ~10 | ~24 |
| **Bug-fix cycles** | 0 | 7+ |
| **Deploys to Vercel** | 3 | 8+ |
| **"Are you working?" moments** | 1 (brief) | 2 (extended) |
| **User complaints about broken functionality** | 0 | 6 |
| **Debug overlays added** | 0 | 1 (had to add ?debug=1 mode) |

### Claude Code's conversation arc (5 user prompts):
1. Initial prompt → built & tested app locally via browser → done
2. "Deploy to Vercel" → deployed in one shot
3. "Add news links" → added hardcoded links with "inspired by" section
4. "Are you working?" → continued editing
5. "Links go to generic AP News" → searched for specific article URLs, updated all 30 fortunes, redeployed

### Codex's conversation arc (12 user prompts):
1. Initial prompt → built app
2. "Deploy to Vercel" → asked which host, then deployed
3. "Doesn't look like a cookie" → offered two options, then redesigned
4. "Fortunes repeat, link to news?" → asked which approach (3 options), then wired up RSS
5. "Getting 'Live headlines unavailable'" → added proxy fallbacks
6. "Cookie is no longer clickable" → attempted fix
7. "Still not clickable" → another fix attempt
8. "Not getting a fortune when I click" → fixed height/aspect-ratio issue
9. "Can you use an MCP to test?" → couldn't, offered debug overlay
10. "Sure" → added debug mode
11. "I don't see changes" → cache debugging
12. (Multiple more rounds fixing regex errors, cookie reset, fortune quality)

---

## Code Metrics

| Metric | Claude Code | Codex |
|--------|------------|-------|
| **Total lines** | 558 | 545 |
| **CSS lines** | 327 | 222 |
| **HTML markup lines** | 73 | 30 |
| **JavaScript lines** | 145 | 275 |
| **Total characters** | 24,456 | 19,156 |
| **JS functions** | 5 | 13 |
| **CSS rules** | ~45 | ~28 |
| **CSS animations** | 4 (shake, crack, crumbs, screen shake) | 0 |
| **Hardcoded fortunes** | 30 | 25 |
| **Chinese words** | 15 | 10 |

---

## Architecture & Design Approach

### Visual Design

| Aspect | Claude Code | Codex |
|--------|------------|-------|
| **Cookie rendering** | Hand-drawn SVG (3 SVGs: whole, left half, right half) | CSS-only (border-radius + radial gradients) |
| **External fonts** | Yes (Google Fonts: Crimson Text + Inter) | No (system fonts: Alegreya/Georgia fallback) |
| **Color palette** | Warm cream (#FFF9F0) with golden-brown accents | Warm cream (#f7f2e8) with golden-brown accents |
| **Crack animation** | Cookie shakes → halves split with spring easing → crumbs fall → screen shakes | Halves translate apart with ease timing |
| **Fortune presentation** | Slip slides in below cookie with scale+fade | Slip fades in below cookie |

### News Integration — The Big Divergence

This is where the two tools made fundamentally different architectural choices:

**Claude Code** went with **hardcoded fortunes paired to real articles**. Each of the 30 fortunes was hand-written to be funny ("A penny saved is a Penny who wins at Westminster") and linked to a specific news URL (NPR, CNN, NBC, Variety, etc.). No runtime fetching.

**Codex** went with **live RSS feeds** from BBC and NPR, fetched at runtime through CORS proxies (allorigins.win, freeboard.io, jina.ai). It extracts keywords from headlines and plugs them into fortune templates ("While {key} trends, your smartest move is a snack and a slow blink.").

| Trade-off | Claude Code | Codex |
|-----------|------------|-------|
| **Freshness** | Static (fortunes age) | Dynamic (live headlines) |
| **Reliability** | 100% (no external calls) | Fragile (proxy-dependent; broke during session) |
| **Humor quality** | High (hand-crafted jokes) | Generic (template-based) |
| **Link specificity** | Each fortune → specific article | Each fortune → general headline link |
| **Content safety** | Manually curated | Blacklist/whitelist filtering at runtime |
| **Offline capability** | Fully works offline | Falls back to generic fortunes |

---

## Quality & Polish

### Claude Code
- Self-tested the app in a real browser using Chrome automation
- Caught and fixed its own layout issues (fortune clipping, scroll behavior, hint visibility) before showing the user
- Added 4 CSS animations (cookie shake, crack spring, crumb fall, screen shake)
- Built detailed SVG cookie art with texture dots, highlights, shadows, and break edges
- Every fortune is genuinely witty and hand-linked to a real article
- Zero user-reported bugs

### Codex
- Could not test in a browser (no MCP access)
- Multiple rounds of the user reporting broken click handlers, missing fortunes, regex errors
- Needed a debug overlay (?debug=1) to troubleshoot
- Simpler CSS-only cookie (looks decent but less "illustrated")
- Fortune templates produce readable but not particularly funny output
- Had to add content safety filtering (blacklist for violence/crime) after a fortune made fun of a child murder headline
- Ended up with broken JavaScript in the final file — there's a syntax error visible in the chat log (duplicate `.flatMap` line, misplaced code)

---

## Time & Effort Efficiency

| Dimension | Claude Code | Codex |
|-----------|------------|-------|
| **Prompts to working v1** | 1 | 1 |
| **Prompts to deployed + polished** | 5 | 12+ |
| **User effort** | Low (mostly "yes" and feature requests) | High (debugging, retesting, providing screenshots, hard-refreshing) |
| **Self-correction** | Yes (caught layout bugs via browser testing) | No (relied on user to find all bugs) |
| **Total deploys** | 3 (build → add links → fix URLs) | 8+ (many were bug fixes) |

---

## Summary

Claude Code delivered a more polished product in fewer interactions. Its key advantages were the ability to self-test via browser automation (catching bugs before the user ever saw them), and the choice to hardcode curated content rather than introduce fragile runtime dependencies. The result was funnier fortunes, zero user-reported bugs, and a richer visual experience with SVG art and multiple animations.

Codex took a more ambitious technical approach with live RSS feeds, but that ambition introduced fragility — CORS proxies broke, click handlers stopped working, and the user spent most of the session debugging rather than enjoying the app. Without browser testing capability, every bug required a round trip through the user.

The fortune cookie experiment is a nice microcosm of a broader pattern: the tool that can verify its own work before shipping tends to produce better outcomes with less user friction.
