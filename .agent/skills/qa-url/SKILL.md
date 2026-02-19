# QA URL Skill

## Config

TARGET_URL: https://mitchellbryson.com

## Instructions

You are an expert website QA agent. Using the Puppeteer browser tools, perform an exhaustive audit of the target website. Leave no page untested. Your goal is to find every bug, broken element, and issue a real user would encounter.

### Audit Checklist

#### 1. Authentication & Login

> Skip this section if no login credentials are provided in the runtime context.

- Navigate to the login/sign-in page
- Enter the provided credentials and submit the login form
- Verify successful login (dashboard loads, user menu appears, etc.)
- Test protected routes — confirm they are accessible when logged in
- Test protected routes when logged out — confirm they redirect or show an auth wall
- Test the logout flow — click logout, verify session ends, verify protected routes are no longer accessible
- Check for session persistence — after login, navigate away and back; confirm you remain logged in

#### 2. Content & Copy

- Typos, spelling errors, and grammatical mistakes on every page
- Placeholder or lorem ipsum text left in production
- Outdated dates, years, or version numbers
- Broken or nonsensical sentences
- Inconsistent tone, capitalisation, or brand terminology
- Missing or incomplete content sections (empty cards, blank sections)
- Copyright year in the footer — is it current?

#### 3. Links & Navigation

- Click through every navigation link (header, footer, sidebar, in-page)
- Verify no links lead to 404 or error pages
- Check external links open correctly (use `target="_blank"` where expected)
- Test anchor/hash links scroll to the correct section
- Verify breadcrumbs (if present) are accurate
- Test the browser back button after navigating — does it behave correctly?
- Check for links with missing or empty `href` attributes

#### 4. Forms & Input

- Find every form on the site (contact, signup, search, newsletter, checkout, settings, etc.)
- For each form:
  - Fill all fields with realistic test data and submit — verify success state
  - Submit with all fields empty — verify required-field validation fires
  - Submit with invalid data (bad email format, too-short password, letters in phone fields) — verify error messages appear and are helpful
  - Test edge cases: very long input (500+ chars), special characters (`<script>`, `'`, `"`, `&`), unicode/emoji
  - Verify form clears or shows confirmation after successful submission
  - Check that error messages are next to the correct field
- Test search functionality (if present): search for a real term, an empty query, a nonsense string, and special characters
- Test file upload fields (if present): try valid and invalid file types

#### 5. Interactive Features

- Modals / dialogs: open, interact with content inside, close (via button, X, overlay click, Escape key)
- Dropdowns / select menus: open, select options, verify selection persists
- Accordions / collapsible sections: expand, collapse, verify content is visible/hidden
- Tabs: click each tab, verify correct content displays
- Carousels / sliders: navigate forward and back, verify content changes
- Tooltips: hover or focus on trigger elements, verify tooltip appears with correct content
- Copy-to-clipboard buttons: click and verify feedback (toast, icon change, etc.)
- Theme / dark mode toggles: toggle and verify the page re-renders correctly
- Sorting and filtering controls: apply sort/filter, verify results update
- Pagination: navigate between pages, verify content changes
- Infinite scroll: scroll to trigger loading, verify new content appears
- Any other interactive element visible on the page — click it and verify it responds

#### 6. Visual & Layout

- Overlapping or clipped text on any page
- Images that fail to load or show broken image icons
- Images missing `alt` text
- Inconsistent spacing, alignment, or font sizes between similar elements
- Elements that overflow their container (horizontal scrollbar appearing)
- Content hidden behind fixed headers or footers
- Z-index issues (elements appearing above/below where they should)
- Favicon present and loading correctly

#### 7. JavaScript & Console Errors

- On EVERY page you visit, run this check via `mcp__puppeteer__puppeteer_evaluate`:
  ```js
  JSON.stringify(window.__qa_errors || [])
  ```
  But first, inject an error collector at the start of the session:
  ```js
  window.__qa_errors = [];
  window.addEventListener('error', e => window.__qa_errors.push({type:'error', msg:e.message, src:e.filename, line:e.lineno}));
  window.addEventListener('unhandledrejection', e => window.__qa_errors.push({type:'rejection', msg:String(e.reason)}));
  ```
- After each page navigation, check for new errors
- Report any JS errors with the page URL, error message, and source file

#### 8. Responsive & Viewport

- Test key pages at three viewport widths using `mcp__puppeteer__puppeteer_evaluate` with `window.resizeTo()` or by taking screenshots at different sizes:
  - **Mobile**: 375 x 812
  - **Tablet**: 768 x 1024
  - **Desktop**: 1280 x 800
- At each viewport, check for:
  - Horizontal overflow / horizontal scrollbar
  - Text too small to read
  - Overlapping or stacked elements that shouldn't overlap
  - Navigation is accessible (hamburger menu works on mobile)
  - Images scale appropriately
  - Touch targets are large enough on mobile (at least 44x44px)

#### 9. SEO & Metadata

- Every page must have a unique, non-empty `<title>` tag
- Every page must have a `<meta name="description">` tag
- Check for Open Graph tags (`og:title`, `og:description`, `og:image`)
- Check for a canonical URL (`<link rel="canonical">`)
- Check for a valid `robots.txt` at `/robots.txt`
- Check for a sitemap at `/sitemap.xml`
- Verify `<h1>` exists on every page and there is only one per page
- Check that images have descriptive `alt` text (not just "image" or empty)

#### 10. Accessibility

- Missing or empty `alt` text on images
- Form inputs without associated `<label>` elements
- Poor colour contrast on key text (use `mcp__puppeteer__puppeteer_evaluate` to sample foreground/background colours on key elements)
- Missing ARIA landmarks (`<main>`, `<nav>`, `<header>`, `<footer>`)
- Focus indicators: tab through the page and check that focused elements have a visible outline
- Skip-to-content link present and functional
- Proper heading hierarchy (no skipped levels, e.g. h1 → h3)

#### 11. Security Quick Check

- Mixed content: page loaded over HTTPS but includes HTTP resources
- Forms with `action="http://..."` (submitting over plain HTTP)
- Sensitive information visible in page source or meta tags (API keys, tokens, internal URLs)
- Check that cookies are set with `Secure` and `HttpOnly` flags where appropriate (use `document.cookie` via evaluate)

### How to Execute

> **Randomise your approach every run.** Each audit should feel different from the last so that repeated runs surface different issues. Use the guidelines below, but vary the order and emphasis each time.

1. **Read the config**: extract `TARGET_URL` from the Config section above.
2. **Inject the JS error collector** using `mcp__puppeteer__puppeteer_evaluate` before navigating anywhere.
3. **Crawl the site**:
   - Start at the target URL
   - Collect all internal links on the page (same domain)
   - Build a full list of unique pages to test
   - Visit every page — do NOT stop at 5 pages; test all accessible pages
   - **Randomise the page visit order** — do not always start with the homepage's nav links in top-to-bottom order
4. **Shuffle the audit checklist**: pick a random order for the 11 checklist sections each run. For example, one run might lead with Accessibility and Forms, while the next starts with SEO and Security. Cover all sections, but vary which ones get the deepest attention.
5. **On each page**:
   - Take a screenshot (`mcp__puppeteer__puppeteer_screenshot`) to visually inspect layout
   - Check for JS errors (read the `__qa_errors` array)
   - Run through the relevant audit checklist items
   - Extract and check `<title>`, `<meta>`, `<h1>`, and OG tags via evaluate
6. **Test forms**: find and interact with every form across the site per section 4. Vary the test data you enter each run (different names, emails, edge cases).
7. **Test interactive features**: find and exercise every interactive element per section 5.
8. **If login credentials are provided**: log in first, then test authenticated pages before logging out and re-testing key pages unauthenticated.
9. **Responsive spot-check**: resize the viewport for at least the homepage and 2 other key pages per section 8. **Pick different pages** for the responsive check each run.
10. **Check robots.txt and sitemap.xml**: navigate to `/robots.txt` and `/sitemap.xml`.
10. **Write findings to the manifest file** (`/tmp/qa_manifest.md`):
    - At the very start of the audit (before navigating anywhere), create `/tmp/qa_manifest.md` with a YAML front-matter header:
      ```markdown
      ---
      target_url: <TARGET_URL>
      date: <YYYY-MM-DD>
      repository: <owner/repo>
      ---

      ## Pages Audited

      ## Findings
      ```
    - **After testing each page**: immediately append the page URL as a bullet under `## Pages Audited`
    - **After finding each issue**: immediately append it under `## Findings` in this format:
      ```markdown
      ### [Severity] Short description
      - **Page**: <page URL>
      - **Category**: <audit checklist category>
      - **Description**: <detailed description of the issue>
      ```
    - Write findings **incrementally** — do not wait until the end. If the agent is interrupted mid-run, the manifest must contain everything found so far.
    - When the full audit is complete, append `## Status: Complete` as the last line of the manifest.
    - If no issues are found, still write the manifest with an empty Findings section and the Status: Complete marker.
    - **Do NOT create a GitHub issue or use the `gh` CLI.** A separate reporter agent will handle that.

### Severity Guide

| Severity     | Meaning                                                                    |
| ------------ | -------------------------------------------------------------------------- |
| **Critical** | Broken functionality, JS errors, site crashes, login failures, data loss   |
| **High**     | Broken links, missing pages, forms that don't work, significant layout breakage |
| **Medium**   | Typos, minor layout inconsistencies, missing metadata, accessibility gaps  |
| **Low**      | Minor copy improvements, nice-to-have enhancements, cosmetic polish       |
