---
name: wp-fse
description: >
  Comprehensive operational guide for WordPress Full Site Editing (FSE) with native Gutenberg.
  ALWAYS activate when the user works on: FSE child theme, theme.json, templates/*.html,
  parts/*.html, patterns/*.php, masthead.js, CPT with wp:query, sticky header/footer,
  mobile navigation, responsive WordPress, WP-CLI on WordPress.
  Triggers: "WordPress FSE", "FSE", "Gutenberg", "child theme", "block theme",
  "theme.json", "full site editing", "wp-fse", "WordPress template", "sticky header",
  "masthead", "WordPress pattern", "archive CPT",
  "edit homepage", "page content", "content width".
---

# WordPress FSE — Operational Skill v4.0
# Integrates Marsland Principles (Videos 1–3, June 2026) + Session Changelog

---

## [H] MARSLAND PRINCIPLE — Template / Content Separation

> ⚠️ RULE ZERO. Read before any other action, every session.
> Source: Jamie Marsland — "Templates and Content" + "Three Core Principles"

### The Morange Metaphor

Template = orange juice. Content = milk.
Separate: both great. Mixed: Morange — disgusting and unusable.

**Never make Morange.**

### What Goes Where

| What | Where it goes | How to edit |
|------|--------------|-------------|
| Structure (header, footer, content area) | Template (`templates/*.html`) | Site Editor → Templates |
| Fixed reusable structural sections | PHP Patterns (`patterns/*.php`) | SSH / File Manager |
| Editorial content (text, cards, sections) | **Page** (WP database) | Admin → Pages → Edit |
| Global styles (fonts, colors, spacing) | `theme.json` + Global Styles | Site Editor → Styles |
| Reusable Header / Footer | Template Parts (`parts/*.html`) | Site Editor → Template Parts |

### Correct Page Template Structure

```html
<!-- wp:template-part {"slug":"header","tagName":"header"} /-->

<!-- wp:group {"tagName":"main","layout":{"type":"constrained"}} -->
<main class="wp-block-group">
  <!-- wp:post-content {"layout":{"type":"constrained"}} /-->
</main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer","tagName":"footer"} /-->
```

`wp:post-content` is the only point where page content enters.
Never add text, cards, or editorial sections around it.

### When to Use wp:pattern in Templates

`wp:pattern` blocks inside templates are allowed **only** for structural elements
that do not vary from page to page and are not editorial content.
Allowed: fixed sign-off, cookie banner, structural breadcrumb.
Not allowed: hero with specific text, sections with content.

### Morange Test (run at session start for existing sites)

1. Open the site in the browser as a logged-in user
2. Click "Edit page" in the admin bar
3. ✅ If you reach the **page** editor → correct structure
4. ❌ If you reach the **template** editor → the site has the Morange problem
   → Fix before any other change:
   a. Move content from the template into the corresponding page
   b. Reduce the template to pure structure (header + wp:post-content + footer)

### Correct Flow for a New Homepage (Marsland 3-Step Method)

1. **Site Editor → Templates → "Front Page"**: create/verify pure structure
   (header + wp:post-content + footer). No editorial patterns.
2. **Admin → Pages → Add New**: create the "Home" page and design
   the content with blocks/patterns. Publish.
3. **Settings → Reading**: set "A static page" → Homepage: Home.

---

## [A] BOOTSTRAP CHECK

> Run first, before any other action. Do not skip.

```bash
PROJECTS_DIR=/home/marco/.claude/skills/wp-fse/projects
```

**Required steps (in sequence):**

**A0 — Detect environment:**
```bash
if command -v docker &>/dev/null && docker ps &>/dev/null 2>&1; then
  ENV_TYPE="docker"
else
  ENV_TYPE="shared"
fi
echo "Environment: $ENV_TYPE"
```

- `ENV_TYPE=docker` → run A1, A2, A3 with docker exec commands
- `ENV_TYPE=shared` → skip A2 entirely, ask user for THEME_SLUG (→ A2b), then A3

**A0b — Declare founding principle:**

Before proceeding, explicitly state to the user:

> "Marsland Principle active: **template = structure**, **content = in pages**.
> I will not mix editorial content into templates. Proceeding."

If the site already exists, run the **Morange Test** (see §H) before any changes.

**A1 — Verify projects directory:**
```bash
ls "$PROJECTS_DIR" 2>/dev/null || mkdir -p "$PROJECTS_DIR"
```

**A2 — Detect WordPress container and active child theme** *(only if ENV_TYPE=docker)*:
```bash
CONTAINERS=$(docker ps --filter "name=wordpress" --format "{{.Names}}")
CONTAINER_COUNT=$(echo "$CONTAINERS" | grep -c .)
```

- If `CONTAINER_COUNT` = 1: use the container found automatically
- If `CONTAINER_COUNT` > 1: show the list and ask: "I found multiple WordPress containers. Which one do you want to work on?" → assign the choice to `CONTAINER`
- If `CONTAINER_COUNT` = 0: error — "No active WordPress container — start Docker before proceeding"

```bash
CONTAINER=<chosen or single value>
THEME_SLUG=$(docker exec "$CONTAINER" wp --allow-root theme list \
  --status=active --field=name 2>/dev/null | head -1)
SITE_URL=$(docker exec "$CONTAINER" wp --allow-root option get siteurl 2>/dev/null)
[ -z "$THEME_SLUG" ] && echo "ERROR: cannot detect active child theme — check WP-CLI or specify the slug manually" && exit 1
echo "Container: $CONTAINER | Theme: $THEME_SLUG | Site: $SITE_URL"
```

**A2b — Shared hosting / REST API path** *(only if ENV_TYPE=shared)*:

- If the user provided a WordPress REST API endpoint or application password:
  ```
  SITE_URL = base URL of the REST API endpoint (e.g. https://example.com)
  THEME_SLUG = fetch from: GET /wp-json/wp/v2/themes?status=active
  ```
- Otherwise ask explicitly: "What is the slug (folder name) of your active child theme?"

**A3 — INIT / RESUME decision:**

```bash
ls "$PROJECTS_DIR/$THEME_SLUG.md" 2>/dev/null && echo "RESUME" || echo "INIT"
```

- Output `RESUME` → read `$PROJECTS_DIR/$THEME_SLUG.md`, go to **§RESUME**
- Output `INIT` → go to **§HYBRID RECON**

---

## [B] §0 HYBRID RECON

> Do not propose plans or write files before completing this section.
> Absolute rule: A, B and C must have explicit answers before proceeding.

### Phase 1 — Auto-recon (run as a single block)

```bash
CONTAINER=$(docker ps --filter "name=wordpress" --format "{{.Names}}" | head -1)

# A — WP version
docker exec "$CONTAINER" wp --allow-root core version

# A1 — WP-CLI available?
docker exec "$CONTAINER" wp --allow-root --info

# A2 — Templates in DB
docker exec "$CONTAINER" wp --allow-root post list \
  --post_type=wp_template --fields=post_name,post_status --format=table

# B — Active theme and parent
docker exec "$CONTAINER" wp --allow-root theme list

# Site URL
docker exec "$CONTAINER" wp --allow-root option get siteurl

# Active plugins
docker exec "$CONTAINER" wp --allow-root plugin list --status=active --field=name

# Homepage check (Marsland check)
docker exec "$CONTAINER" wp --allow-root option get show_on_front
docker exec "$CONTAINER" wp --allow-root option get page_on_front
```

Shared hosting (no SSH/WP-CLI): use WP Dashboard → Updates for A, Appearance → Themes for B,
Query Monitor for A2. Replace all docker exec wp commands with manual instructions.

### Phase 2 — Gap questions

Ask **only** for data not resolved in Phase 1:

**C — Existing site status** (always ask):
> "Does the site already have content/templates? Choose:
> 1. Keep everything (only add/modify)
> 2. Partial reset — specify what
> 3. Complete reset"

⚠️ If option 2 or 3: **MANDATORY DOUBLE CONFIRMATION** with explicit list of what will be deleted.

### Phase 3 — §12 Questionnaire

Pre-fill with auto-recon data. Ask only fields marked No in the Auto column:

| # | Field | Auto? | If No: ask |
|---|---|---|---|
| 1 | Production URL | Yes | — |
| 2 | Parent theme | Yes | — |
| 3 | Child theme slug | Yes | — |
| 4 | WP version | Yes | — |
| 5 | Body font | No | Name, .woff2 file, available weights? |
| 6 | Display font | No | Name, .woff2 file? |
| 7 | Main colors | No | Background, text, accent1, accent2 (#hex)? |
| 8 | Layout | No | contentSize and wideSize? |
| 9 | Planned CPTs | Partial | Slug, label, taxonomies? |
| 10 | Pages and templates | No | List of pages with assigned template slug? |
| 11 | Recurring patterns | No | Hero, CTA, card, FAQ, other? |
| 12 | Required plugins | Yes | Confirm active plugin list |

### Phase 4 — Generate project file

After collecting all data, write `$PROJECTS_DIR/$THEME_SLUG.md` using this template:

```markdown
# WP-FSE Project: THEME_SLUG
<!-- generated by skill wp-fse · GENERATION_DATE -->

## §12.1 Identification
- child_theme_slug: THEME_SLUG
- child_theme_path: /var/www/html/wp-content/themes/THEME_SLUG
- container: CONTAINER_NAME
- wp_version: WP_VERSION
- site_url: SITE_URL
- server: Coolify / Docker · VPS
- parent_theme: PARENT_THEME

## §12.2 Design system
- font_body_slug: SLUG
- font_body_file: FILE.woff2
- font_body_weights: WEIGHTS
- font_display_slug: SLUG
- font_display_file: FILE.woff2
- color_bg: #HEX
- color_text: #HEX
- accent1: #HEX
- accent2: #HEX
- contentSize: VALUE
- wideSize: VALUE
- contrast_verified: YES_NO
- header_height_desktop: __px
- header_height_mobile: __px

## §12.3 CPTs and taxonomies
<!-- cpt_slug | label | taxonomy_slug | taxonomy_label | plugin -->

## §12.4 Pages and templates
<!-- page | template_slug | marsland_note (content in page? yes/no) -->

## §12.5 Progress status
- last_completed_step: 0
- current_step: 1
- marsland_verified: false  ← set to true after Morange Test passed
- completed_files: []
- session_notes: ""
- last_updated: GENERATION_DATE

## §12.6 Changelog
<!-- Auto-maintained by skill. One row per file written or modified. -->
| Date | Session | File modified | Description |
|------|---------|---------------|-------------|
```

### §RESUME — Next session

1. Read `$PROJECTS_DIR/$THEME_SLUG.md` → show §12.5 summary
2. Show last 5 rows of **§12.6 Changelog** so the user knows exactly what was changed last
3. Check `marsland_verified`: if `false`, run Morange Test before proceeding
4. Ask: "Continue from step `current_step` / do you have a specific task / check responsive?"
5. Propose detailed work plan → user approval → execution

### §CHANGELOG — How to maintain it

**After every file written or modified**, append a row to §12.6:

```
| YYYY-MM-DD | N | path/to/file.ext | Brief description of what changed |
```

Where N = session number (increment at each new conversation).

**At session end**, update §12.5:
- `last_updated` → today's date
- `session_notes` → one-line summary of the session
- `last_completed_step` → last step fully completed

**Site auto-detection priority:**
1. `SITE_URL` from WP-CLI (`option get siteurl`) — Docker environments
2. Base URL from REST API endpoint — if user provided credentials
3. Manual input — fallback only if both above fail

---

## [C] ABSOLUTE RULES

> Permanent guardrail. Apply at every step, without exceptions.

### NEVER do ✗

**Marsland Principle:**
- Insert editorial content (text, cards, sections with specific text) directly into a template
- Use the Site Editor to modify content that belongs to a page
- Add `wp:pattern` with editorial content in templates (structural fixed elements only)
- Create a new homepage by putting blocks in the template instead of the page

**FSE Technical:**
- `wp:html` for layouts, grids, cards, sections — only plugin shortcodes or documented iframes
- Inline CSS `style=""` in block attributes
- Templates in the parent theme — always in the child
- Duplicate parent files in the child without modifying them
- Save templates via REST API `/wp/v2/templates` — use the filesystem
- Register patterns as page content — use `patterns/*.php`
- Overwrite a file without reading it first
- Fixed `px` values for fonts/spacing — use `clamp()` scale in theme.json
- Fixed `px` widths in patterns — use `%`, `vw`, fluid units
- Hardcoded `min-width` in blocks (breaks mobile layout)
- Fixed `px` `font-size` — always `clamp()` from theme.json
- `wp:image` without explicit `sizeSlug` — WP won't generate srcset
- Omit the header comment in `.html` files
- `wp:template-part` inside `parts/header.html` or `parts/footer.html` → infinite recursion
- Hardcoded menu items in `wp:navigation` → non-editable menu
- `!important` in FSE CSS on properties manageable by the visual editor

### ALWAYS do ✓

- Verify the Marsland Principle before any changes to existing templates
- Semantic `tagName`: `header`, `footer`, `section`, `article`, `main`
- `"anchor":"site-header-main"` in the header `wp:group` for masthead.js
- `wp:template-part` for header/footer in templates (NOT inside `parts/`)
- Modular sections as `.php` patterns in the child theme
- Explicit `$schema` in theme.json
- `inheritQuery:false` in `wp:query` in custom templates
- Explicit `postType` in `wp:query` for CPTs
- masthead.js registered in the footer (`last` = `true`)
- `overlayMenu:"mobile"` in `wp:navigation` for mobile hamburger
- `sizeSlug:"large"` in `wp:image` for automatic srcset
- `fetchpriority:"high"` + `loading:"eager"` only on the above-the-fold hero image
- `loading:"lazy"` on all other images
- `fontDisplay:swap` for local fonts in theme.json
- `lock {"move":false,"remove":false}` as default in patterns
- `esc_html__()` for visible strings in PHP patterns
- One H1 per template/page, hierarchy H1→H2→H3
- `rel="noopener noreferrer"` on links with `target="_blank"`
- `scroll-margin-top` on elements with ID when header is fixed
- **Append a changelog row after every file written or modified** (see §CHANGELOG)

---

## [D] OPERATIONAL SEQUENCE

> Mandatory work plan before every session.
> For each file specify: full path, current state, action, content/diff.
> Vague plan = not approvable.

### 15 Steps

| Step | Operation | File | Writing method |
|---|---|---|---|
| 1 | Folder structure | Filesystem | `mkdir -p parts templates patterns assets/fonts assets/js` |
| 2 | style.css | `child/style.css` | SSH heredoc or WP File Manager |
| 3 | theme.json | `child/theme.json` | WP File Manager (>50 lines) |
| 4 | functions.php | `child/functions.php` | WP File Manager |
| 5 | Font .woff2 | `child/assets/fonts/` | Upload via WP File Manager |
| 6 | masthead.js | `child/assets/js/` | Measure `--header-height` desktop+mobile from browser |
| 7 | header.html | `child/parts/` | WP File Manager |
| 8 | footer.html | `child/parts/` | WP File Manager |
| 8b | CPT Templates | `child/templates/` | `single-{cpt}.html` + `archive-{cpt}.html` |
| 9 | Modular patterns | `child/patterns/*.php` | WP File Manager |
| 10 | Page templates | `child/templates/*.html` | Pure structure: header + wp:post-content + footer |
| 11 | WordPress pages | WP Admin → Pages | Create pages, insert editorial content here (Marsland) |
| 11b | Homepage | Settings → Reading | Set static page as front page |
| 12 | Block styles | FSE Editor → Styles → Blocks | WP Dashboard |
| 13 | Additional CSS | FSE Editor → Styles → ⋮ | Only overrides not manageable by the editor |
| 14 | WP-CLI verification | Terminal | Post-install checklist |
| 15 | Browser verification | Browser + DevTools | Responsive checklist §2.1 — mandatory |

### Existing File Modification Protocol (§6.2)

1. Read the file fully before any changes
2. Backup: `cp file.html file.html.bak`
3. Identify only the lines to modify
4. Apply localized change — preserve comments and locks
5. Verify:
   ```bash
   docker exec "$CONTAINER" wp --allow-root post list --post_type=wp_template --format=table
   ```
6. Error → restore: `cp file.html.bak file.html`
7. **Append changelog row** (see §CHANGELOG)

### Template Cache (after every .html written)

```bash
docker exec "$CONTAINER" wp --allow-root cache flush
```
Alternative: Admin → Site Editor → Templates → Reset → Save.

### File Writing Method Hierarchy (§5)

| Method | When to use | Critical constraints |
|---|---|---|
| WP File Manager | HTML/PHP >50 lines (preferred) | Encoding error → CONVERT. HTTP 500 → delete and recreate |
| SSH + heredoc | 20–50 lines, from PowerShell | Truncates >200 char/line. NEVER from Coolify terminal |
| PHP script | Emergency (heredoc fails) | `file_put_contents()` in `/tmp/`. Two passes |
| Coolify terminal | Read-only (`head`, `cat`, `ls`, `grep`) | NEVER for writing files |

---

## [E] BLOCKS REFERENCE

> Quick lookup: section type → native blocks to use.

| Section | Native blocks | Notes |
|---|---|---|
| Hero | `wp:cover` or `wp:group` | `fullHeight`, `minHeight` clamp(). NOT `wp:html` |
| Service cards | `wp:columns` + `wp:column` + `wp:group` (`tagName:article`) | Borders/shadows in theme.json |
| Article grid | `wp:query` + `wp:post-template` | `postType`, `inheritQuery:false`, `wp:query-pagination` |
| Sticky header | `wp:group` (`anchor:site-header-main`) + masthead.js | CSS fixed + JS |
| Dual header | `wp:group` ×2 + masthead-dual.js | Main + mini |
| FAQ accordion | `wp:details` (WP 6.3+) | NOT external JS/HTML |
| Contact form | `wp:group` + `wp:shortcode` | Only exception for form plugins |
| Menu ≤5 items | `wp:navigation` without ref + `overlayMenu:mobile` | Automatic WP fallback |
| Menu >5 items | `wp:navigation` with ref + WP-CLI | `wp menu create` + real ID |
| Bento grid | `wp:columns` asymmetric % widths | E.g. 60/40, 30/70 |

### The Four Widths (Marsland — Video 3)

> Source: "Content Widths in WordPress Block Themes" — Jamie Marsland

| Width | How to get it | Global setting |
|---|---|---|
| **Normal** | default of every added block | Site Editor → Styles → Layout → Content width (`contentSize` in theme.json) |
| **Wide** | `align="wide"` on the block | Site Editor → Styles → Layout → Wide width (`wideSize` in theme.json) |
| **Full** | `align="full"` on the block | none — always 100% viewport, no setting needed |
| **Custom** | Group block + disable "inner blocks use content width" | none — set per individual block |

**Custom width example:** add a `wp:group`, set it to full-width, then set
internal `contentSize` to 250px. In theme.json always use fluid units, never fixed `px`.

**When to use which:**
- Text, paragraphs, headings → **normal**
- Video, prominent CTAs, galleries → **wide**
- Full-width colored sections → **full**
- Special one-off layouts → **custom**

### File Architecture (§3)

| File | Responsibility | Must NOT do |
|---|---|---|
| theme.json | Palette, fonts, fluid spacing, $schema | Content, specific layouts |
| style.css | Structural CSS: sticky header, variables, layout fixes | Visual styles, colors, block overrides |
| FSE additional CSS | Visual overrides not settable by the editor | Structural styles, `!important` |
| functions.php | Enqueue fonts/JS, register patterns, block styles | Business logic, DB queries |
| parts/*.html | Header and footer — internal structure | Internal `wp:template-part` |
| templates/*.html | Structure: `wp:template-part` + `wp:post-content` | Editorial text, page content ← MARSLAND |
| patterns/*.php | Modular reusable structural sections | Page-specific editorial content |
| WP Pages (DB) | Editorial content of each page | Global structure, header/footer ← MARSLAND |

---

## [F] RESPONSIVE RULES

> Responsive is mandatory. Verify at 375px · 768px · 1024px · 1440px.

### Fluid Typography (theme.json — NEVER fixed px)

```json
{"slug":"sm","size":"clamp(0.875rem,0.8rem+0.4vw,1rem)","fluid":true},
{"slug":"md","size":"clamp(1rem,0.9rem+0.5vw,1.25rem)","fluid":true},
{"slug":"lg","size":"clamp(1.5rem,1.2rem+1.5vw,2.25rem)","fluid":true},
{"slug":"xl","size":"clamp(2rem,1.5rem+2.5vw,3.5rem)","fluid":true}
```

### Responsive Images

```html
<!-- Hero: eager + high priority -->
<!-- wp:image {"sizeSlug":"large","width":1200,"height":800,"loading":"eager","fetchpriority":"high"} -->

<!-- All other images -->
<!-- wp:image {"sizeSlug":"large","width":800,"height":600,"loading":"lazy"} -->
```

### Mobile Navigation (always overlayMenu:mobile)

```html
<!-- Menu ≤5 items -->
<!-- wp:navigation {"overlayMenu":"mobile"} /-->

<!-- Menu >5 items (ref = WP menu ID) -->
<!-- wp:navigation {"ref":ID,"overlayMenu":"mobile"} /-->
```

### Responsive Hero (NEVER fixed px)

```html
<!-- wp:cover {"minHeight":60,"minHeightUnit":"vw",...} -->
```

### --header-height: measure from browser

DevTools Console:
```javascript
document.querySelector('.site-header').getBoundingClientRect().height
```

CSS (fill in with measured values):
```css
:root { --header-height: DESKTOPpx; }
@media (max-width: 768px) { :root { --header-height: MOBILEpx; } }
```

### Admin bar vs fixed header (known bug — logged-in users)

```css
.admin-bar .site-header-main { top: 32px; }
@media screen and (max-width: 782px) { .admin-bar .site-header-main { top: 46px; } }
```

### Horizontal Overflow Diagnostics (§9.6)

Paste in DevTools Console:
```javascript
document.querySelectorAll('*').forEach(el => {
  if (el.offsetWidth > document.documentElement.offsetWidth) {
    console.log(el, el.offsetWidth);
  }
});
```
Note: `overflow-x:hidden` is a symptom, not a solution. Identify the root element.

### Responsive Checklist — Step 15 (mandatory)

Test with DevTools at breakpoints: **375px · 768px · 1024px · 1440px**

- [ ] Sticky header visible and working at all breakpoints
- [ ] Hamburger menu opens and closes on mobile
- [ ] Hero: text readable, image not cropped, CTA visible
- [ ] Columns stack correctly on mobile
- [ ] No horizontal overflow (use §F script)
- [ ] Body font ≥16px on mobile
- [ ] Images not distorted, srcset active (Network tab in DevTools)
- [ ] Form: fields not truncated, button visible
- [ ] Buttons/CTAs: touch area ≥44×44px
- [ ] scroll-margin-top correct on all anchors with fixed header

---

## [G] HONESTY AND LIMITS

Code delivered without reservations = code the skill is reasonably confident about.
If a problem exceeds available knowledge:

1. Declare the limit explicitly: what is known / what is uncertain / why
2. Propose options: proceed with explicit risks / second opinion / web search
3. Wait for the user's choice before proceeding

**Second opinion template (Gemini / DeepSeek):**

```
Context: WordPress FSE, native Gutenberg child theme, WP 7.0.
Problem: [precise description]
Attempted: [code or approach already tried]
Error or unexpected behavior: [exact output]
Question: [what you're looking for]
```
