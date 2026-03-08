---
name: email-gazette
description: "Email Gazette: Transform unread inbox emails into a beautiful newspaper-style HTML front page. Use this skill whenever the user asks for an email briefing, inbox summary, email digest, morning briefing, 'what happened in my inbox', 'catch me up on emails', 'summarize my emails', or anything about turning emails into a readable overview. Also trigger when the user mentions 'gazette' or asks for a newspaper-style view of their emails, daily digest, or email newspaper."
---

# Email Gazette

Turn an unread inbox into a broadsheet newspaper front page that opens in the browser.

## Overview

This skill fetches unread emails via the Google Workspace CLI (`gws`), reads and categorizes them, enriches key stories with web research, then generates a single self-contained HTML file styled like a classic newspaper (think The New York Times). It opens directly in the browser and is archived for future reference.

## Step 0: Prerequisites Check & First-Run Configuration

Before anything else, verify that the Google Workspace CLI (`gws`) is installed and authenticated. Run:

```bash
gws gmail +triage --max 1 --format json
```

If this fails, stop and tell the user:
> "This skill requires the Google Workspace CLI (`gws`) to access your Gmail. You can install it and the related Gmail skills using `npx skills add nicholasgasior/gws` (or however the gws skills are distributed). Once installed, run `gws auth login` to authenticate, then try again."

Do not proceed until `gws` is working.

On first use, ask the user to configure their gazette. Store their answers so you remember them across sessions (e.g., in the project's CLAUDE.md or your memory). If they've already configured, skip ahead.

Ask these questions:

1. **Gazette name** — "What would you like to call your gazette?" (e.g., "The Morning Wire", "Alex's Daily Brief", "The Inbox Times"). This becomes the masthead title.

2. **Tagline** — "Want a tagline beneath the masthead?" Suggest a default like "All the Email That's Fit to Print" but let them customize (e.g., "Your inbox, editorialized", "News from the void").

3. **Output directory** — "Where should I save past editions?" Suggest `~/Desktop/email-gazette/` as default. Create the directory if it doesn't exist.

4. **Topics of interest** — "What topics matter most to you? These get headline treatment." Suggest common categories (technology, AI, business, finance, crypto, health, politics, science) but let them specify anything. Everything else still appears, just in smaller print.

5. **Email cap** — "How many unread emails should I process per edition?" Default: 50. More = richer gazette but slower.

6. **Date format for filenames** — "How should I name the files?" Default: `YYYY-MM-DD.html` (sorts well). Alternative: `DD-MM-YYYY.html`.

If the user seems impatient or says something like "just use defaults", use sensible defaults (gazette name: "The Email Gazette", tagline: "All the Email That's Fit to Print", output: `~/Desktop/email-gazette/`, topics: technology/AI/business, cap: 50, date: YYYY-MM-DD) and move on.

## Step 1: Fetch Unread Emails

Use the `gws` CLI to list unread inbox messages:

```bash
gws gmail +triage --max {EMAIL_CAP} --format json
```

This returns a JSON array with sender, subject, date, and message ID for each unread message.

**Requires:** The `gws` CLI must be installed and authenticated. If the command fails, let the user know they need to set up the Google Workspace CLI first.

## Step 2: Read Email Content

For each email from triage, fetch the full message to understand its content:

```bash
gws gmail users messages get --params '{"userId":"me","id":"MESSAGE_ID","format":"metadata","metadataHeaders":"Subject"}' --format json
```

This returns the email snippet (a preview of the body content). You don't need to fetch every single email in full — use judgment:

- **Full read** for emails matching the user's topics of interest, anything that looks like it contains important news, and personal emails that seem urgent
- **Subject-only** is fine for obvious marketing, automated notifications, and promotional emails

## Step 2b: Web Research for Key Topics

After identifying the major stories (typically 3-6), enrich them with external sources. This turns the gazette from a simple email summary into a genuinely informative briefing.

For each major headline story, use web search to find:
- **X/Twitter posts** from notable accounts discussing the topic
- **arXiv papers** if the story involves research or a model release
- **Official blog posts** from the companies involved
- **News articles** from reputable press

Run searches in parallel where possible. For each story, search for: `[topic] [key company/person] [current year]`. Aim for 2-5 external sources per major story.

**Important:** Subagents may not have web search permissions. If so, run the web searches directly yourself rather than delegating. Don't search for low-priority "In Other News" items — only enrich the stories that matter.

## Step 3: Categorize

Sort every email into one of these tiers, using the user's configured topics of interest to guide what counts as "headline" material:

### HEADLINE NEWS (top priority, most visual space)
- Newsletters and updates matching the user's topics of interest
- Breaking or time-sensitive information in those domains
- Major industry news

### CRUCIAL ALERTS (visually prominent, draws immediate attention)
- Emails requiring urgent action
- Important deadlines or reminders
- Interview requests, scheduling, or meeting invitations
- Anything from a real person (not a mailing list) that seems urgent or important

### IN OTHER NEWS (small print, side panel, minimal space)
- Routine automated notifications
- Low-value marketing newsletters
- Social media digests
- Promotional emails
- Anything that doesn't need attention

## Step 4: Generate the Newspaper HTML

Produce a single self-contained HTML file. The aesthetic direction is **classic broadsheet newspaper**:

### Design Direction

**Aesthetic:** Editorial / classic newspaper with a modern twist. Think The New York Times or The Financial Times meets a beautifully typeset broadsheet.

**Typography:**
- Use a distinctive serif font for headlines (e.g., from Google Fonts: Playfair Display, Cormorant Garamond, or similar editorial serif)
- A clean readable serif or sans-serif for body text (e.g., Source Serif Pro, Libre Baskerville, or similar)
- The masthead should use the user's configured gazette name, rendered grand and prominent like a real newspaper nameplate
- Include the current date beneath the masthead, formatted like a newspaper date line (e.g., "Saturday, March 8, 2026")
- Show the user's configured tagline in italics beneath the name

**Layout:**
- Multi-column layout using CSS grid, like a real newspaper
- HEADLINE NEWS gets the most space: a lead story spanning multiple columns at the top, with supporting stories in a grid
- CRUCIAL ALERTS get a visually distinct treatment: a bordered box with a red or bold accent, "URGENT" or "BREAKING" style banners
- IN OTHER NEWS goes at the bottom in noticeably smaller type, compressed like classified ads
- Add subtle column rules (thin vertical lines between columns) for authenticity

**Visual details:**
- Thin horizontal rules to separate sections
- A subtle paper/cream background color for warmth
- Drop caps on the lead story's first paragraph
- Section headers styled like newspaper section labels (derived from the user's topics, e.g., "TECHNOLOGY", "FINANCE", "AI & MACHINE LEARNING")
- For each article/email summary: a headline (derived from the subject), a brief summary (2-3 sentences), and the source/sender in small italics
- Stories with particularly important or newsworthy content get larger headlines and more column space

**Reference numbering and source links:**

Every story and claim should have small superscript reference numbers linking to sources. This is a two-part system:

1. **Email references** — Each email that contributed to a story gets a superscript number (e.g., `<sup>[1]</sup>`) linking to the email in Gmail: `https://mail.google.com/mail/u/0/#inbox/MESSAGE_ID`

2. **Web references** — External sources from web research (Step 2b) also get superscript numbers, continuing the sequence. These link directly to the external URL.

At the bottom of each story, render references as a small footnote block. Style in a very small, understated font (academic citation style) so they don't clutter the newspaper aesthetic but are available when needed.

Include a consolidated "Sources" section at the bottom of the gazette (before the footer) listing all references grouped by story — a bibliography for readers who want to dive deeper.

**Crucial alerts treatment:**
- Visually prominent: boxed section with distinct background color or border
- Labels like "URGENT", "ACTION REQUIRED", "REMINDER" as small badges
- Key details: who it's from, what's needed, any deadline

**In Other News treatment:**
- Very small font size, dense compact layout
- Just sender, subject, and one line of context
- Each item links to its source email
- Should feel like the fine print at the bottom of a newspaper page

### Technical Requirements
- Single self-contained HTML file (all CSS inline or in a `<style>` block)
- Load fonts from Google Fonts CDN (the only external dependency allowed)
- Responsive: optimized for desktop but shouldn't break on mobile
- No JavaScript required unless you want subtle hover effects
- Save to `{OUTPUT_DIR}/{DATE}.html` using the user's configured directory and date format

## Step 5: Open in Browser

```bash
open {OUTPUT_DIR}/{DATE}.html
```

On Linux, use `xdg-open` instead. On Windows/WSL, use `wslview` or `explorer.exe`.

## Important Notes

- **Read-only.** Never modify, delete, or mark emails as read.
- **Empty inbox?** Generate a charming gazette page that says something like "No News is Good News" with a tasteful empty-state design.
- **Group related emails.** Multiple emails from the same newsletter or about the same topic should be merged into a single story.
- **Editorial voice.** Write summaries in a newspaper editorial voice: concise, informative, slightly formal but engaging. Transform email subjects into proper newspaper-style headlines and ledes.
- **It's a newspaper, not a list.** The gazette should feel like a real publication. Editorialize the presentation — the goal is to make reading email feel like reading the morning paper.
