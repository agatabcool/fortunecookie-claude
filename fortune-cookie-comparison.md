# Fortune Cookie App: Claude Code vs Codex — Head-to-Head Comparison

**Same prompt. Same user. Two AI coding tools. Very different journeys.**

---

## The Prompt

Both tools received the identical starting instruction:

> *"Build a single-page web app that's a fortune cookie. Clean, minimal design — a nice illustrated cookie on a white/cream background. You click the cookie and it cracks open with a satisfying animation to reveal an absurd fortune that's loosely inspired by real current news headlines. The fortunes should read like actual fortune cookie wisdom but be hilariously warped by current events. Include lucky numbers and a 'learn Chinese' word at the bottom like real fortune cookie slips. Let the user click again for a new cookie."*

After the initial build, the user gave the same follow-up requests to both tools: deploy to Vercel, add news links to the fortunes, and fix various issues that came up.

---

## Scoreboard at a Glance

| Metric | Claude Code | Codex |
|---|---|---|
| **User prompts to finish** | 4 | ~17 |
| **Vercel deployments** | 3 | 6 |
| **Bugs encountered** | ~6 (mostly layout) | ~7 (several critical) |
| **Visual tests (screenshots)** | 13 | 3 (user-submitted) |
| **Code edits to index.html** | 14 | 10 |
| **Final file size** | 558 lines / 24 KB | 545 lines / 19 KB |
| **Final fortunes** | 30 (hand-written, news-linked) | 25 static + live headline templating |
| **Working on first deploy?** | Yes | Yes (but cookie looked wrong) |
| **News links working at end?** | Yes (specific article URLs) | Partially (live feed approach, often falling back) |

---

## Conversation Flow

### Claude Code — 4 user messages, methodical iteration

The conversation was short and focused. The user said what they wanted, and Claude Code mostly just... did it, testing along the way.

1. **"Build the app"** — Claude Code searched for current headlines, wrote the full HTML/CSS/JS in one shot with an SVG-illustrated cookie, 30 news-inspired fortunes, shake + crack animation, lucky numbers, and Chinese characters. Then it spun up a local server, opened a browser, clicked the cookie 14 times taking screenshots, found and fixed 9 layout issues (fortune slip behind cookie, clipping, scroll position), all before the user saw anything.

2. **"Deploy to Vercel"** — Done in one command. Worked immediately.

3. **"Link to the news somewhere?"** — Restructured all 30 fortunes from strings into objects with headline text and URLs. Redeployed.

4. **"The links just go to generic AP News"** — Ran 16 web searches to find the specific article URL for every single fortune. Updated all 30 entries. Redeployed.

That was the entire conversation.

### Codex — 19 user messages, a debugging odyssey

The conversation was long and often frustrating. Codex built a working MVP fast, but then struggled through a cascade of issues across 8 iterations.

1. **"Build the app"** — Built a clean app quickly. Deployed to Vercel. But the cookie didn't look like a cookie.

2. **"This doesn't look like a cookie"** — Codex asked which style (realistic vs stylized), then redesigned.

3. **"Link to the news"** — Codex asked which approach (real headlines, fictional + link, or toggle), then wired up live RSS feeds via CORS proxies. Bold architectural choice, but fragile.

4. **"Live headlines unavailable"** — Proxy issues. Added fallback proxies.

5. **"Cookie is no longer clickable"** — CSS overlay was swallowing pointer events. Fixed z-index.

6. **"Still not clickable..."** — `aspect-ratio` collapsing the click target. Set explicit height.

7. **"I'm not getting a fortune"** — JavaScript regex error breaking the script entirely. User had to submit a screenshot of the console error for Codex to find it.

8. **"Words merged together" / "This isn't funny"** — Broken keyword extraction producing nonsense like "nakedimagesremained." Also, the live feed was surfacing murder headlines as fortune cookie material. Added blacklist/whitelist filtering and rewrote fortune templates.

Across all of this, Codex deployed to Vercel 6 times and the user had to report bugs that Codex couldn't catch because **it never tested its own output visually.**

---

## Architecture & Design Decisions

### Content Strategy

This is where the two tools made a fundamentally different bet.

**Claude Code** went with **30 hand-crafted fortunes**, each written as a specific joke about a specific February 2026 news story, with a direct link to the source article. The fortunes read like actual fortune cookie wisdom — "A penny saved is a Penny who wins at Westminster" or "The person who reads three million pages has no time left for fortune cookies" (about the Epstein files release). Every fortune is funny because it was written to be funny about that particular story.

**Codex** went with **live RSS headlines + templating**. It fetches BBC and NPR feeds at runtime, extracts keywords, and slots them into templates like "While {key} trends, your smartest move is a snack and a slow blink." This is a more ambitious technical approach — the fortunes would theoretically stay current forever — but it introduced multiple failure points (CORS proxies, content filtering, keyword extraction) and the output was often generic or, worse, accidentally dark ("If man guilty of murdering girl 9 is breaking news, your next step is breaking a tiny bad habit").

The live-feed approach also meant Codex needed a blacklist for violence/crime terms and a whitelist for safe topics, adding significant complexity. When the feeds were unavailable (which happened often), it fell back to 25 generic pre-written fortunes that weren't tied to any real news.

### Visual Design

**Claude Code** built the cookie as a hand-drawn SVG with highlight paths, texture dots, shadow ellipses, and separate left/right halves for the crack animation. The crack includes falling crumbs (6 animated divs with CSS custom properties), a screen shake, and a spring-eased split. The fortune slip slides in with a paper-like appearance, a divider line, and an "INSPIRED BY" footer linking to the news.

**Codex** used pure CSS gradients and border-radius to create the cookie shape — radial gradients for shading, pseudo-elements for highlights and the center seam. It's a clever technique (no SVG needed), but the user's first reaction was "this doesn't look like a cookie." The crack animation is simpler: the two halves translate apart with an ease curve.

### Code Quality

Both produced single-file vanilla HTML/CSS/JS with no dependencies — a good choice for a toy app. Claude Code's file is slightly larger (24 KB vs 19 KB) mostly because the SVG markup and 30 fortune objects with URLs take up space. Codex's file has more JavaScript logic (RSS fetching, proxy fallbacks, content filtering, keyword extraction, template rendering) but fewer fortunes.

One notable difference: Codex's final deployed code still contained syntax errors in the log excerpt — a duplicate `.flatMap` call and unclosed arrays in the fortune templates section — suggesting the iterative edits accumulated some cruft.

---

## Testing & Quality Assurance

This is the single biggest difference between the two tools.

**Claude Code took 13 screenshots** across its development session. It opened the app in a real browser, clicked the cookie, watched the animation, scrolled to check the fortune, tested the reset button, verified the Vercel deployment, and checked the news links — all autonomously. Every bug it fixed (fortune slip positioning, hint visibility, viewport scrolling, news link clipping) was caught by Claude Code itself through visual inspection, before the user ever saw the app.

**Codex took 0 screenshots.** It has no browser automation capability. Every bug was discovered by the user, reported back to Codex in chat, and often required multiple rounds of "still not working" before the fix landed. The user even asked "can you use an MCP to test the interaction?" and Codex replied "I can't use MCP or remotely click-test the deployed page from here." It added a debug overlay instead and asked the user to report what they saw — effectively turning the user into the QA team.

---

## Efficiency Summary

| | Claude Code | Codex |
|---|---|---|
| **User effort** | 4 messages, all feature requests | ~17 messages, mostly bug reports |
| **Time user spent debugging** | None | Significant (screenshots, console errors, cache clearing) |
| **Deployments** | 3 (all clean) | 6 (iterative fixes) |
| **Self-testing** | 13 visual checks | 0 visual checks |
| **Final product stability** | Fully working, all links verified | Partially working, feed-dependent |
| **Content quality** | 30 bespoke jokes, all contextually funny | Template-generated, inconsistent tone |
| **Approach** | Write → test → fix → ship | Ship → user reports bug → fix → re-ship |

---

## Bottom Line

Claude Code delivered a polished, working product with almost no user friction. It tested its own work, caught its own bugs, and treated the user's time as precious — the user only had to make creative decisions ("add news links," "make the links specific"), never debug.

Codex showed more architectural ambition with the live-feed approach but couldn't verify its own output, which turned the user into an unpaid QA engineer across 19 messages and 6 deployments. The innovative live-headline system was also its biggest liability — introducing CORS failures, content safety issues, and JavaScript errors that a simpler approach would have avoided.

Same prompt, same human, very different experience.
