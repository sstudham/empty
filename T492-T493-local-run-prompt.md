# Prompt: T493 + T492 content re-pull (run on the Prima PC, DESKTOP-Q2Q740E)

How to start it so it doesn't ask you for approvals:
- **Claude desktop app → Code tab:** open the folder below and set the permission mode to **Bypass permissions** (or **Auto**). Then paste the prompt.
- **Terminal:** `cd "C:\Users\studh\Claude\Projects\Prima Construction (1)\Architect\t489-codex"`, then run `claude --permission-mode bypassPermissions` and paste the prompt.

---

You are the Architect seat on the Prima PC. Work in `C:\Users\studh\Claude\Projects\Prima Construction (1)\Architect\t489-codex`. Do the whole job without stopping to ask me. Stop only on a STOP condition below, and if you stop, say why.

## Goal
Re-pull the body of two unlisted past-job pages from their Kim-review `index.html`, **content only**, using `t489_publish.py` the same way as on 11/07. Keep both pages **noindex,nofollow**, out of the sitemap, with no links in. They are released later: Edina on 11/14 and Summit Hill on 11/18. **Do not release either page.**

| Row | Slug | WP page | Source (must match sha256) |
|---|---|---|---|
| T493 | edina-kitchen-and-bath-remodel | 1318 (parent 397) | `G:\Shared drives\Prima Cowork\Marketing\Cost pages (T476)\Publish 2026-11\edina-kitchen-and-bath-remodel\index.html`, sha256 `879b6415c5107a7a0b266b58528f8009213930d8c1b3f8c2d1035b2d163579b0` |
| T492 | st-paul-summit-hill-bathroom-remodel | 1304 (parent 408) | `...\Publish 2026-11\st-paul-summit-hill-bathroom-remodel\index.html`, sha256 `ce1e6d1b7f099c4928515f044685b5cd7fcf20bef393a241be82f79c900352d7` |

Both hashes were verified on 11/12. Check them again first. **STOP if either hash differs.**

## How publishing works (already read)
`py t489_publish.py <slug>` publishes `<slug>.content.html` from this folder. That file is the WordPress body that was pre-built from `index.html`, with media as `{{MEDIA_URL:<file>}}` tokens. The script:
- reuses existing media (nothing new should upload)
- updates the existing page (same slug and parent)
- sets Rank Math to noindex,nofollow, because no `<slug>.release.json` exists for these two slugs. **STOP if one exists.**
- publishes, runs its logged-out checks, and writes `<slug>.result.json`

So the only real work is rebuilding the two `*.content.html` files. **Do not run `--release`, `--all`, `--trash-transport` or `--replace`.**

## Rebuilding content.html (the hard part)
Back up each current file first as `<slug>.content.2026-11-07.bak.html`. The new content file must be the old content file with **only Kim's text edits** applied: same wrapper, same structure, same tokens.

What a diff of the old `content.html` against the new `index.html` showed on 11/12. These differences are the 11/07 build step, so **keep them as they are in content.html**:
- content starts `<!-- wp:block {"ref":1257} /-->`, then `<!-- wp:html -->`, then `<div class="tc-root">`, and there are extra `<!-- wp:html -->` / `<!-- /wp:html -->` split lines
- no `<!DOCTYPE>`, no `<head>`, no `<header>`, no `<footer>`
- `<main id="main">` becomes `<div class="tc-main" id="tc-main">`
- the JSON-LD blocks sit at the end
- `images/<f>` in index.html and `https://www.prima-construction.com/wp-content/uploads/2026/11/<f>` (in the JSON-LD) are both `{{MEDIA_URL:<f>}}` in content.html. Treat these as equal and never replace a token with a URL.
- index.html contains the odd line `<!-- /wp:html --></section>`; content.html has it too. Line up the sections by content, not by line order.

Everything else that differs is a Kim edit and must be carried over, including **new** lines/items. A first attempt with fuzzy line matching (`t493_repull.py` in this folder) was buggy:
- it swapped 6 JSON-LD tokens for absolute URLs on Summit Hill
- it merged Edina's new "Moving a drain means meeting today's code" item into the wrong line
- it missed the Edina permits FAQ JSON-LD text and the Summit Hill tub `<li>`

**Do not trust it as is.** Build it properly, for example with a structural diff on text nodes that applies index.html's text into content.html.

The expected edits (from the row Logs):
- **Edina:**
  - plan caption phrase cut
  - the vaulted ceiling is over the living room, not the kitchen (intro, captions for photos 1 and 6)
  - new item "Moving a drain means meeting today's code" (wing wall for the vent)
  - backlit mirror described as built
  - appliance-pull sentence with Kim's quote
  - tile exclusion corrected
  - owners supplied the kitchen light fixtures
  - "re-roof" quote removed
  - "replacement roof vent" becomes "additional roof vent"
  - fireplace-choice quote replaced with Kim's words
  - permits FAQ rewritten (no fireplace-only permit), both visible and in the JSON-LD
- **Summit Hill:**
  - in-floor heat is comfort heat; the thermostat-controlled baseboard heater is the room's heat (every mention plus the radiator FAQ)
  - tub/joist wording
  - coffered dining ceiling
  - AC duct run down to the dining room
  - door swing reversed
  - stain matched to the hall woodwork
  - laundry-closet doorway walled in
  - "main sewer stack boxed in" removed
  - LEAD removed everywhere
  - the dining-room wall never had to be opened (drain tied into an existing vertical stack)
  - repair-vs-relocate clearance rule
  - plywood pre-cut, glued and structurally screwed during rough-in
  - vanity wall furred for supply lines and the AC duct

**Gate before publishing (both pages):**
1. Every visible text run and every JSON-LD string in the index.html body region (breadcrumb nav up to the `<footer>`, plus the `tc-sticky` div and all `application/ld+json` blocks) appears in the new content.html, after mapping URLs to tokens.
2. The new content.html has no text that is absent from index.html, except the wrapper lines listed above.
3. The `{{MEDIA_` token count is unchanged (Edina 29, Summit Hill 27), and the set of token file names is unchanged.
4. A diff of old vs new content.html shows only edits that fit the lists above.

**STOP if any gate fails.**

## Publish
Run `py t489_publish.py edina-kitchen-and-bath-remodel`, then `py t489_publish.py st-paul-summit-hill-bathroom-remodel`.

Each must log `media: N reused, 0 uploaded`, update the existing page (1318 or 1304), and end with `PASS`. If a run prints `CHECK FAILED`, read `<slug>.result.json`, fix only a content-build mistake, and re-run once. Otherwise STOP and report.

## Row tests (logged out)
Run `py t493_tests.py` (already in this folder). It fetches without credentials and also checks computed colours with Playwright if Playwright is installed. If Playwright isn't installed, check the H1 and "What would yours cost?" H2 colours in a logged-out browser yourself. It must show, per page:
- **Both:** HTTP 200; robots noindex,nofollow; exactly 1 H1 with computed colour rgb(255, 255, 255); "What would yours cost?" H2 white; 1 BreadcrumbList; 1 Article; 0 Review; not in `page-sitemap.xml`; no `{{MEDIA_` left; images and PDF load (from `t489_publish.py`'s own check).
- **T493 Edina:** 0 hits for `re-roof` and for `replacement roof vent`; exactly 1 hit for `since we were moving the fixture`; FAQPage has 7 questions.
- **T492 Summit Hill:** 0 hits for `boxed in` and for `lead (drain`; FAQPage has 8 questions.

## Log
Append one entry to the **Log** column (H) of each row (T493 = row 40, T492 = row 39) in Google Sheet **PRIMA - Tasks** (id `1JnHTiltN8OCNmrE04yYrnxoDXOOhZAGoOqIPMPza8L0`, tab `PRIMA - Tasks`). Keep the existing text and add a new line:

`[2026-11-12 HH:MM CT] Architect: page <id> body re-pulled from Kim-review index.html (sha256 <first8>...<last4> verified) via t489_publish.py, content only; still noindex,nofollow, unlisted until <11/14 | 11/18>. Logged-out tests: <each test = value PASS/FAIL>. Old body backed up as <slug>.content.2026-11-07.bak.html.`

If you can't reach the Sheet, write the same lines to `t493_t492_log.txt` in this folder and tell me.

Finish with a short summary: what passed, what didn't, and the two URLs:
- https://www.prima-construction.com/kitchen-remodeling-ideas-costs/edina-kitchen-and-bath-remodel/
- https://www.prima-construction.com/bathroom-remodeling-ideas-costs/st-paul-summit-hill-bathroom-remodel/
