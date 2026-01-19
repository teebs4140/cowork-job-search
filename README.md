# Job Finder Cowork

A full-funnel AI job search assistant that handles everything from resume parsing to offer negotiation. All data stays local, persists across sessions, and you control every submission.

> **Note**: This workspace is built for **Claude Cowork** (Claude Desktop's workspace mode), not Claude Code (the CLI). Skills live in `.skills/` which Cowork detects automatically.

## What It Does

| Phase | Capability |
|-------|------------|
| **Discover** | Parse resume, interview preferences, search job boards |
| **Apply** | Research companies, draft cover letters, pre-fill applications |
| **Prep** | Generate interview questions, mock interviews, company research |
| **Negotiate** | Salary research, total comp analysis, negotiation coaching |

One smart skill that detects your intent - no need to remember commands.

## Requirements

- [Claude Desktop](https://claude.ai/download) with Cowork mode
- [Claude in Chrome extension](https://chromewebstore.google.com/detail/claude-in-chrome/afogcklkgicdbjhgnfnlomfgegflegoh) (for job board automation)
- Accounts on job boards you want to search (LinkedIn, Indeed, etc.)

## Quick Start

1. Clone this repo
2. Open Claude Desktop and start a new Cowork session
3. Select `job-finder-cowork` as your workspace
4. Upload your resume and say "help me find jobs"

The skill triggers automatically and guides you through:
- Resume analysis
- Quick interview about your preferences
- Job search on your chosen boards
- Everything saves for next time

## Returning Users

Once you've completed the initial interview, just say what you want:
- "Find more jobs" - searches with your saved criteria
- "Apply to the Acme Corp job" - starts application assist
- "Prep for my Google interview" - generates interview prep
- "Analyze this offer" - runs comp analysis
- "Show my dashboard" - displays analytics

Your profile persists - you never repeat the interview.

## Folder Structure

```
job-finder-cowork/
├── profile/
│   ├── profile.json           # Your preferences (auto-created)
│   ├── resume.[ext]           # Your uploaded resume
│   ├── cover_letter_base.md   # Base cover letter template
│   └── writing_samples/       # For voice learning (optional)
├── searches/
│   ├── contexts/              # Search configurations
│   │   └── default.json       # Default search criteria
│   └── results/               # Search snapshots by date
├── applications/
│   └── {company_slug}/
│       ├── job.json           # Job details and status
│       ├── cover_letter.md    # Tailored cover letter
│       ├── prep.md            # Interview prep notes
│       └── offer_analysis.md  # Offer breakdown
├── analytics/
│   └── dashboard.json         # Your job search metrics
├── .skills/                   # Skill definition (don't edit)
├── .gitignore
├── README.md
└── CONTRIBUTING.md
```

## Features

### Full-Funnel Support

**Discovery**
- Resume parsing and analysis
- Adaptive interview (not rigid questions)
- Multi-board search with deduplication

**Application Assist**
- Company research automation
- Cover letter generation
- Form pre-filling
- Always requires your confirmation before submitting

**Interview Prep**
- Deep company research
- Likely question generation
- Mock interview practice
- STAR format preparation

**Offer Analysis**
- Market salary research
- Total comp breakdown
- Negotiation strategies

### Job Tracking

All jobs tracked with status:
- `new` - Just discovered
- `interested` - Marked for follow-up
- `applied` - Application submitted
- `interviewing` - In interview process
- `rejected` - Didn't work out
- `offer` - Received an offer
- `archived` - No longer relevant

Update status naturally: "mark the Acme job as applied"

### Analytics Dashboard

Track your funnel:
```
Found: 127 → Interested: 34 → Applied: 22 → Interviews: 3 → Offers: 1
```

Stall detection: If you're applying but not getting interviews, the skill proactively suggests improvements.

### Multiple Search Contexts

Run different searches in parallel:
- Default search for your main role
- "PM roles" context for product management
- "Startup only" context for early-stage companies

Create with: "Create a new search for PM roles"

## Supported Job Boards

**Primary (best automation)**
- LinkedIn Jobs - requires login
- Indeed - works without login

**Secondary**
- Remote.co - remote-focused
- FlexJobs - requires subscription
- Glassdoor - includes salary data

## Privacy

Your data stays on your computer:
- All personal files in `profile/`, `applications/`, `analytics/` are gitignored
- Nothing sent to external services except job board searches
- You control every application submission

## Example Session

**First time:**
```
You: Help me find a job
Claude: I'll analyze your resume first... [analyzes]
Claude: What type of role are you looking for? [interview questions]
Claude: Great, I've saved your profile. Want me to search LinkedIn and Indeed?
You: Yes
Claude: Found 15 new jobs. Here are the top matches... [shows results with scores]
```

**Returning:**
```
You: Find more jobs
Claude: Welcome back! Searching with your saved criteria...
Claude: Found 8 new jobs since your last search.

You: Apply to the first one
Claude: I'll research the company and draft a cover letter...
Claude: Here's everything ready for submission. Confirm to apply?
```

## Troubleshooting

**Browser automation not working**
- Ensure Claude in Chrome extension is installed and enabled
- Check you're logged into the job boards
- Try Indeed first (less authentication friction)

**Profile not loading**
- Check `profile/profile.json` exists
- If corrupted, delete and re-run interview

**Rate limited on LinkedIn**
- Space out searches
- Try Indeed as alternative
- Wait and retry later

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to:
- Add new job boards
- Improve matching algorithms
- Add interview question templates

## License

MIT
