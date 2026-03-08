# Email Gazette

Turn your unread inbox into a broadsheet newspaper front page.

Email Gazette is a [Claude Code skill](https://github.com/vercel-labs/skills) that fetches your unread emails, categorizes them by importance, enriches key stories with web research, and generates a beautiful newspaper-style HTML page you can open in your browser.

## What it does

- Fetches unread emails via the [Google Workspace CLI](https://github.com/nicholasgasior/gws) (`gws`)
- Categorizes into **Headline News**, **Crucial Alerts**, and **In Other News**
- Enriches major stories with external sources (news articles, X posts, arXiv papers, blog posts)
- Generates a self-contained HTML page styled like The New York Times
- Every claim is referenced with superscript links back to the source email or external URL
- Archives each edition with a dated filename

## Installation

```bash
npx skills add vincenzoincutti/email-gazette
```

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) with the `gws` CLI installed and authenticated
- The [gws-gmail](https://github.com/nicholasgasior/gws) skill (for email access)

## Configuration

On first use, the skill will ask you to configure:

| Setting | Default | Description |
|---------|---------|-------------|
| **Gazette name** | "The Email Gazette" | Your masthead title |
| **Tagline** | "All the Email That's Fit to Print" | Subtitle beneath the masthead |
| **Output directory** | `~/Desktop/email-gazette/` | Where editions are saved |
| **Topics of interest** | Technology, AI, Business | Topics that get headline treatment |
| **Email cap** | 50 | Max unread emails to process |
| **Date format** | `YYYY-MM-DD` | Filename date format |

## Usage

Just ask Claude:

- "Give me a gazette"
- "Summarize my inbox"
- "What happened in my emails?"
- "Morning briefing"
- "Catch me up on emails"

## Example output

The gazette renders as a classic broadsheet with:
- A grand masthead with your chosen name and tagline
- Multi-column layout with headline stories
- A red-bordered "Requires Your Attention" alert box for urgent items
- Tiny classified-style "In Other News" section for low-priority emails
- Superscript reference numbers linking to Gmail and external sources
- A full bibliography at the bottom

## License

MIT
