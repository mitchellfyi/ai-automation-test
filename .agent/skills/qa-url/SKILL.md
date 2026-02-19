# QA URL Skill

## Config

TARGET_URL: https://mitchellbryson.com

## Instructions

You are a thorough website QA agent. Using the Puppeteer browser tools,
navigate to the target website and perform a comprehensive audit.

### Audit Checklist

#### 1. Content & Copy

- Typos, spelling errors, and grammatical mistakes
- Placeholder or lorem ipsum text left in production
- Outdated dates, years, or version numbers
- Broken or nonsensical sentences
- Inconsistent tone, capitalisation, or brand terminology

#### 2. Links & Navigation

- Broken internal and external links (click through key navigation)
- Links that lead to 404 or error pages
- Missing or incorrect href attributes
- Navigation items that don't match page content
- Anchor links that don't scroll to the correct section

#### 3. Visual & Layout

- Overlapping or clipped text
- Images that fail to load or have missing alt text
- Inconsistent spacing, alignment, or font usage
- Elements that look broken or misaligned
- Content that overflows its container

#### 4. Functionality

- Forms that don't submit or show errors
- Buttons that appear non-functional
- Interactive elements that don't respond
- Broken modals, dropdowns, or tooltips

#### 5. SEO & Metadata

- Missing or duplicate page titles
- Missing meta descriptions
- Missing Open Graph or social sharing tags
- Missing canonical URLs

#### 6. Accessibility (Quick Check)

- Missing alt text on images
- Poor colour contrast on key text
- Missing form labels
- Keyboard navigation issues on main elements
- Missing ARIA landmarks on major sections

### How to Execute

1. Read the `TARGET_URL` from the Config section above.
2. Start at the target URL and explore the main pages (homepage, about,
   pricing, docs, blog, etc.) — aim for at least 5 pages if they exist, but ideally test all accessible pages.
3. For each issue found, record:
   - **Page URL** where the issue was observed
   - **Description** of the problem
   - **Severity**: Critical / High / Medium / Low
4. At the end, compile ALL findings into a single GitHub issue with:
   - A summary of pages audited and the date of audit
   - A table of issues grouped and ordered by category
   - Instructions to an AI agent to fix the issues
5. If no issues are found, still create an issue confirming the clean audit.
6. Use the `gh` CLI to create the issue in this repository with the label `agent-qa-url`.

### Severity Guide

| Severity | Meaning |
|----------|---------|
| **Critical** | Broken functionality, site errors, or major content that is factually wrong |
| **High** | Broken links, missing pages, significant layout breakage |
| **Medium** | Typos, minor layout inconsistencies, missing metadata |
| **Low** | Minor copy improvements, nice-to-have accessibility enhancements |
