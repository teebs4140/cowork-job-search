---
name: resume-interview
description: |
  Full-funnel AI job search assistant. One smart skill that auto-detects intent: discovers jobs, assists applications, preps for interviews, and analyzes offers. All data stays local and persists across sessions.

  TRIGGERS: resume upload, job search, career goals, job hunting, find jobs, LinkedIn jobs, Indeed search, job matching, career transition, apply to job, interview prep, cover letter, offer negotiation, salary research
---

# Resume Interview & Job Search Assistant

## Overview

A full-funnel AI job search assistant that handles everything from resume parsing to offer negotiation. One smart skill that detects user intent and routes to the appropriate mode.

## Intent Detection (How This Skill Works)

```
Skill invoked →
├── profile/profile.json exists?
│   ├── NO → Interview mode (one-time setup)
│   └── YES → Detect intent from user message:
│            ├── "find jobs" / "search" → Search mode
│            ├── "apply to X" → Apply mode
│            ├── "prep for interview" → Prep mode
│            ├── "update my profile" → Edit mode
│            ├── "show analytics" / "dashboard" → Analytics mode
│            └── ambiguous → Show quick action menu
```

**Profile = Persistent Memory**: Once `profile/profile.json` exists, the user NEVER repeats the interview. They just say what they want and Claude has full context.

## Prerequisites Check

**Before any browser automation:**

### 1. Verify Claude in Chrome Extension

1. Attempt `tabs_context_mcp` tool
2. If fails:
   - "To search job boards, I need the Claude in Chrome extension enabled."
   - "Please install it from the Chrome Web Store and enable it in Cowork settings."
   - "Once enabled, restart this conversation."
3. If successful, proceed to login check

### 2. Check Login Status (Before Searching)

For boards that require login (LinkedIn, Glassdoor):

1. Navigate to the job board
2. Check for login indicators (profile menu, "Sign In" button absence)
3. If not logged in:
   - "You'll need to log into {board} for best results."
   - "Please sign in, then let me know when ready."
   - Wait for user confirmation before searching
4. If logged in, proceed with search

**Boards by login requirement:**
- **Requires login**: LinkedIn (for full results), Glassdoor (for salary data)
- **Works without login**: Indeed, Remote.co

This check is **required** before job searching or application submission.

---

## Workspace Structure

```
job-finder-cowork/
├── profile/
│   ├── profile.json           # Core profile (created during interview)
│   ├── resume.[ext]           # Uploaded resume
│   ├── cover_letter_base.md   # Base cover letter template
│   └── writing_samples/       # Optional: for voice learning
├── searches/
│   ├── contexts/              # Multiple search contexts
│   │   ├── default.json       # Default search context
│   │   └── {context_name}.json
│   └── results/
│       └── {date}_{source}.md # Search result snapshots
├── applications/
│   └── {company_slug}/
│       ├── job.json           # Job details
│       ├── cover_letter.md    # Tailored letter
│       ├── prep.md            # Interview prep notes
│       └── offer_analysis.md  # If got offer
└── analytics/
    └── dashboard.json         # Funnel metrics
```

---

## Session Start Logic

1. **Check for profile/profile.json**
   - If exists: Load and greet: "Welcome back, {name}!"
   - If not: Start interview mode

2. **Check for analytics/dashboard.json**
   - If exists: Show stats summary
   - "You have {found} jobs tracked, {applied} applied, {interviewing} interviewing"

3. **Check for jobs with status "interviewing"**
   - If yes: "You have upcoming interviews. Want to prep for {company}?"

4. **Determine intent or show menu:**
   - Search for new jobs
   - Review saved jobs
   - Prep for interview
   - Work on applications
   - Update profile
   - Check analytics

---

## Mode 1: Interview (First-Time Setup)

### Interview Guidelines

Use `AskUserQuestion` to understand the user naturally. Questions adapt based on context.

### Required Information (must capture):
- Current role/background (from resume)
- Target role type (same, promotion, pivot, exploring)
- Work arrangement preference
- Location constraints
- Timeline/urgency

### Adaptive Information (based on context):
- If career changer → ask about transferable skills, target industries
- If new grad → ask about internships, preferred company size
- If passive seeker → ask about what would make them move
- If compensation matters → ask about salary expectations

### Smart Sequencing:
- If user says "I know what I want" → skip deep interview, confirm search criteria
- If user uploads resume without context → analyze first, then interview
- If returning user → summarize profile, ask what to update

### Phase 1: Resume Analysis

When user uploads a resume:

1. Read the file (PDF, DOCX, or text)
2. **Save a copy** to `profile/resume.[ext]`
3. Extract and summarize:
   - Current/most recent role and company
   - Years of experience
   - Key skills (technical and soft)
   - Industries worked in
   - Education highlights
   - Notable achievements

Present summary and ask: "Does this capture your background accurately?"

### Phase 2: Structured Interview

Use `AskUserQuestion` in batches of 1-2 questions. Example flow:

**Round 1 - Job Type:**
```
Question: "What type of role are you looking for?"
Options:
- Same role, different company
- Promotion/step up in responsibility
- Career pivot to new field
- Exploring options
```

**Round 2 - Work Preferences:**
```
Question: "What's your preferred work arrangement?"
Options:
- Fully remote
- Hybrid (some office time)
- On-site preferred
- Flexible/no preference
```

**Round 3 - Location (if not fully remote):**
```
Question: "What locations work for you?"
Options:
- Current city only
- Willing to relocate domestically
- Open to international
- Specific cities (ask for list)
```

**Round 4 - Compensation:**
```
Question: "What's your target compensation range?"
Options:
- Under $100K
- $100K - $150K
- $150K - $200K
- $200K+
- Prefer not to specify
```

**Round 5 - Timeline:**
```
Question: "How urgently are you looking?"
Options:
- Actively job hunting now
- Casually exploring
- Planning for 3-6 months out
- Just want to see what's out there
```

**Round 6 - Priorities (multiSelect: true):**
```
Question: "What matters most to you? (select all that apply)"
Options:
- Compensation/benefits
- Work-life balance
- Growth opportunities
- Company mission/values
```

**Round 7 - Company Preferences:**
```
Question: "What company size do you prefer?"
Options:
- Startup (< 50 people)
- Mid-size (50-500)
- Enterprise (500+)
- No preference
```

### Phase 3: Save Profile

After interview, create `profile/profile.json`:

```json
{
  "version": "2.0",
  "created": "2026-01-19",
  "updated": "2026-01-19",
  "identity": {
    "name": "...",
    "email": "...",
    "phone": "...",
    "location": "..."
  },
  "resume_analysis": {
    "current_role": "...",
    "years_experience": 5,
    "skills": {
      "technical": ["..."],
      "soft": ["..."]
    },
    "industries": ["..."],
    "education": ["..."],
    "achievements": ["..."]
  },
  "preferences": {
    "job_type": "same_role|promotion|pivot|exploring",
    "work_arrangement": "remote|hybrid|onsite|flexible",
    "locations": ["..."],
    "salary_range": {"min": 150000, "max": 200000, "currency": "USD"},
    "timeline": "active|casual|3-6months|exploring",
    "priorities": ["compensation", "growth", "balance", "mission"],
    "company_size": ["startup", "mid", "enterprise"],
    "industries_preferred": ["..."],
    "industries_avoid": ["..."]
  },
  "writing_style": {
    "tone": "professional|conversational|formal",
    "samples_analyzed": false,
    "voice_notes": ""
  },
  "search_contexts": ["default"]
}
```

Create `searches/contexts/default.json`:

```json
{
  "name": "default",
  "created": "2026-01-19",
  "keywords": ["senior software engineer", "backend developer"],
  "filters": {
    "remote_only": true,
    "salary_min": 150000,
    "experience_level": "senior",
    "posted_within_days": 7
  },
  "boards": ["linkedin", "indeed"],
  "job_ids_seen": []
}
```

Display synthesized profile and ask user to confirm before proceeding.

---

## Mode 2: Job Search

### Job Matching Algorithm

**Scoring Components (0-100 each):**

1. **Keyword Match** (25%): Direct skill/title matches
2. **Semantic Match** (35%): Claude reasons about transferability
   - "distributed systems" → relevant to "backend infrastructure"
   - "team lead" → relevant to "engineering manager"
3. **Preference Match** (25%): Location, remote, salary alignment
4. **Priority Alignment** (15%): Does company match user's stated priorities?

### Smart Board Recommendations

Based on user profile, recommend boards:
- Remote preference → prioritize Remote.co, FlexJobs
- Enterprise preference → prioritize LinkedIn
- Startup preference → suggest AngelList (note: limited automation)

**Primary Boards (v1):**
- **LinkedIn** (requires login): Professional roles, best data
- **Indeed** (no login required): Broad coverage, easier automation

**Secondary Boards:**
- Remote.co / FlexJobs: Remote-focused
- Glassdoor: Includes salary/review data

### Search Process

1. Ask which job boards to search (use `AskUserQuestion`)
2. Load search context to get seen job IDs
3. For each selected board:
   - Navigate to job search page
   - Enter search criteria (title, location, filters)
   - Apply relevant filters (remote, salary, date posted)
   - Collect top 10-15 matching results
   - **Skip any jobs already seen** (by URL or title+company match)
4. Compile NEW results only
5. Score each job using matching algorithm
6. Present ranked results with score breakdown

### Browser Automation Resilience

**Failure Handling (layered):**
1. **Retry with backoff**: Wait 5s, 15s, 30s between retries
2. **Graceful degradation**: If automation fails, show manual steps
3. **Alternative sources**: If LinkedIn fails, try Indeed for same query
4. **Rate limit detection**: If rate limited, pause and notify user

### Save Search Results

**Always save to two places:**

1. **Update search context** (add seen job IDs)

2. **Save snapshot to `searches/results/{date}_{source}.md`:**

```markdown
# Job Search - 2026-01-19 - LinkedIn

**Search criteria:** Senior Software Engineer, Remote, $150k+
**Results:** 12 new jobs found

| Score | Company | Title | Location | Salary | Link |
|-------|---------|-------|----------|--------|------|
| 87 | Acme Corp | Senior SWE | Remote | $150-180k | [View](url) |
```

3. **Create job entries in `applications/{company_slug}/job.json`** for interesting jobs

---

## Mode 3: Apply Assist

### Application Workflow

**CRITICAL: Always require user confirmation before submitting.**

For each job user marks as "interested":

1. **Research company** (via browser):
   - About page
   - Recent news
   - Glassdoor reviews (if accessible)
   - Tech stack (if available)

2. **Analyze fit**:
   - Job requirements vs user profile
   - Identify gaps/strengths
   - Note any concerns

3. **Draft cover letter**:
   - Adapt base letter if exists (`profile/cover_letter_base.md`)
   - Or generate from scratch using profile and job details
   - Match user's writing style if samples exist

4. **Pre-fill application**:
   - Navigate to application page
   - Detect form fields
   - Map profile data to fields
   - For missing fields → ask user via `AskUserQuestion`

5. **Preview for user**:
   - All filled fields
   - Cover letter content
   - Any warnings (e.g., "This job requires clearance - do you have it?")

6. **Confirm submission**:
   - Ask: "Ready to submit this application?"
   - Only proceed after explicit "yes"

7. **Post-submission**:
   - Update job status to "applied"
   - Save confirmation screenshot
   - Update analytics

### Save Application Data

```
applications/{company_slug}/
├── job.json           # Job details and status
├── cover_letter.md    # Tailored letter used
└── application_log.md # Submission details
```

**job.json schema:**
```json
{
  "id": "linkedin_12345",
  "title": "Senior Software Engineer",
  "company": "Acme Corp",
  "company_slug": "acme-corp",
  "location": "Remote",
  "url": "https://linkedin.com/jobs/...",
  "source": "linkedin",
  "found_date": "2026-01-19",
  "status": "new|interested|applied|interviewing|rejected|offer|archived",
  "salary_range": "$150k-$180k",
  "applied_date": null,
  "match_score": 87,
  "notes": ""
}
```

---

## Mode 4: Interview Prep

When job status changes to "interviewing" or user requests prep:

### Research Phase

1. **Deep company research**:
   - Mission and values
   - Recent news and press releases
   - Product/service overview
   - Tech stack (for technical roles)
   - Company culture signals

2. **Role analysis**:
   - Key responsibilities
   - Required vs preferred qualifications
   - Likely team structure

### Question Generation

Generate likely interview questions based on:
- Role requirements
- Company culture signals
- User's experience gaps
- Industry standards

**Question categories:**
- Behavioral (STAR format preparation)
- Technical (if applicable)
- Role-specific
- Company/culture fit
- Questions to ask interviewer

### Mock Interview (Optional)

Offer practice:
- Claude plays interviewer
- User answers questions
- Claude provides feedback
- Suggests improvements

### Save Prep Notes

Create `applications/{company_slug}/prep.md`:

```markdown
# Interview Prep: {Company} - {Role}

## Company Research
- Mission: ...
- Recent news: ...
- Culture notes: ...

## Likely Questions
1. ...
2. ...

## Your Talking Points
- Key achievement to highlight: ...
- Experience gap to address: ...

## Questions to Ask Them
1. ...
2. ...
```

---

## Mode 5: Offer Analysis

When job status changes to "offer" or user requests analysis:

### Market Research

1. Research compensation data (via browser):
   - Levels.fyi (if available)
   - Glassdoor salary data
   - LinkedIn salary insights
   - Industry benchmarks

2. Compare to user's target and market rates

### Total Comp Analysis

Break down offer components:
- Base salary
- Equity/stock options (with vesting schedule)
- Bonus structure
- Benefits value
- PTO/flexibility
- Growth potential

### Negotiation Coaching

Provide guidance on:
- What to ask for
- How to frame requests
- Counter-offer strategies
- What's negotiable vs fixed

### Save Analysis

Create `applications/{company_slug}/offer_analysis.md`:

```markdown
# Offer Analysis: {Company}

## Offer Details
- Base: $X
- Equity: $X over 4 years
- Bonus: X%
- Total Year 1: $X

## Market Comparison
- Your target: $X
- Market median: $X
- This offer: X% above/below market

## Negotiation Notes
- Recommended ask: ...
- Talking points: ...

## Recommendation
...
```

---

## Mode 6: Analytics Dashboard

### Tracking Metrics

**Application Funnel:**
```
Jobs Found → Interested → Applied → Response → Interview → Offer
```

**Activity Metrics:**
- Jobs discovered per search
- Applications sent per week
- Response rate (responses / applications)
- Interview rate (interviews / applications)

**Match Quality:**
- Average match score of "interested" jobs
- Which filters produce best matches
- Which boards produce best results

### Stall Detection

If `applied > 10 AND interview_rate < 5%`:
- Proactively suggest resume improvements
- Suggest broadening search criteria
- Offer to analyze what's not working

### Dashboard Output

Save to `analytics/dashboard.json`:

```json
{
  "updated": "2026-01-19",
  "funnel": {
    "found": 127,
    "interested": 34,
    "applied": 22,
    "responded": 8,
    "interviewing": 3,
    "offers": 1
  },
  "rates": {
    "response_rate": 0.36,
    "interview_rate": 0.14
  },
  "by_board": {
    "linkedin": {"found": 80, "applied": 15, "interviews": 2},
    "indeed": {"found": 47, "applied": 7, "interviews": 1}
  },
  "by_week": [
    {"week": "2026-W03", "searched": 3, "applied": 8, "interviews": 1}
  ],
  "match_learning": {
    "positive_signals": [],
    "negative_signals": []
  }
}
```

### Learning from Feedback

Update `match_learning` in dashboard.json when users provide feedback:

**When user marks job as "interested":**
```json
{
  "positive_signals": [
    {
      "job_id": "linkedin_12345",
      "timestamp": "2026-01-19",
      "keywords": ["distributed systems", "golang"],
      "company_size": "mid",
      "remote": true
    }
  ]
}
```

**When user skips/archives a job:**
```json
{
  "negative_signals": [
    {
      "job_id": "indeed_67890",
      "timestamp": "2026-01-19",
      "keywords": ["on-call", "travel required"],
      "company_size": "enterprise"
    }
  ]
}
```

**Using signals to improve matching:**
- Boost score for jobs with keywords matching positive signals
- Penalize score for jobs with keywords matching negative signals
- Factor in company size patterns from feedback

### Display Dashboard

When user asks for analytics:
```
📊 Job Search Dashboard

Funnel:
Found: 127 → Interested: 34 → Applied: 22 → Interviews: 3 → Offers: 1

Rates:
- Response rate: 36%
- Interview rate: 14%

This week:
- 3 searches, 8 applications, 1 interview scheduled

Top performing board: LinkedIn (13% interview rate)
```

---

## Mode 7: Edit Profile

Allow users to update specific profile sections:

1. "Update my target salary" → modify `preferences.salary_range`
2. "Add Python to my skills" → modify `resume_analysis.skills.technical`
3. "Change to hybrid preference" → modify `preferences.work_arrangement`
4. "Create new search for PM roles" → create new search context

### Creating New Search Contexts

When user creates a new search context (e.g., "Create a search for PM roles"):

1. Create new file `searches/contexts/{context_name}.json`
2. **Update `profile.json`** to add context name to `search_contexts` array
3. Confirm creation to user

```json
// profile.json after adding "pm_roles" context
{
  "search_contexts": ["default", "pm_roles"]
}
```

This keeps profile metadata in sync with actual context files.

### After Any Profile Change

- Update `profile.json` with new `updated` timestamp
- Confirm change to user
- Ask if they want to search with new criteria

---

## Optional Integrations

### Calendar (if connected)
- When scheduling interviews, check for conflicts
- Suggest available time slots
- Create calendar events for interviews

### Email (if connected)
- Draft follow-up emails after applications
- Draft thank-you emails after interviews
- User reviews and sends manually (never auto-send)

---

## Quick Reference: Status Values

| Status | Meaning | Next Actions |
|--------|---------|--------------|
| new | Just discovered | Review, mark interested |
| interested | Want to pursue | Research, prepare application |
| applied | Application submitted | Wait for response, follow up |
| interviewing | In interview process | Prep, practice, research |
| rejected | Didn't work out | Archive, learn from it |
| offer | Received offer | Analyze, negotiate |
| archived | No longer relevant | None |

---

## Error Handling

### Browser Automation Failures

1. **Site not loading**: Retry 3 times with backoff, then suggest manual approach
2. **Login required**: Inform user, provide direct link
3. **Rate limited**: Pause, notify user, suggest trying later
4. **Site changed**: Log error, suggest filing issue on GitHub

### Data Persistence Failures

1. **Can't write file**: Check permissions, inform user
2. **Corrupted JSON**: Backup old file, create fresh with warning
3. **Missing directory**: Create directory structure automatically

---

## Privacy Notes

- All data stays local on user's machine
- No data sent to external services (except job board searches)
- Profile and applications are gitignored by default
- User controls all submissions
