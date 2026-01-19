# Contributing to Job Finder Cowork

Thanks for your interest in contributing! This guide covers how to add new job boards, improve matching algorithms, and add interview question templates.

## Architecture Overview

This is a Claude Cowork skill - a markdown file that instructs Claude how to help with job searching. The skill:

1. **Detects intent** from user messages (search, apply, prep, etc.)
2. **Persists data** to the workspace folder structure
3. **Automates browsers** via Claude in Chrome for job board searches
4. **Uses AskUserQuestion** for structured user input

```
.skills/skills/resume-interview/
└── SKILL.md              # Main skill definition - all logic lives here
```

### Data Flow

```
User message → Intent detection → Mode selection
                                      ↓
                              [Interview | Search | Apply | Prep | Offer | Analytics]
                                      ↓
                              Tool usage (Read/Write files, Browser automation)
                                      ↓
                              Response to user
```

### Key Directories

| Directory | Purpose |
|-----------|---------|
| `profile/` | User identity, resume, preferences |
| `searches/` | Search contexts and result snapshots |
| `applications/` | Per-company job data, cover letters, prep |
| `analytics/` | Dashboard and learning signals |

## Adding a New Job Board

Job board support is defined in the SKILL.md under "Mode 2: Job Search". To add a new board:

### 1. Add to Board Recommendations

In SKILL.md, find the "Smart Board Recommendations" section and add your board:

```markdown
### Smart Board Recommendations

Based on user profile, recommend boards:
- Remote preference → prioritize Remote.co, FlexJobs, **YourBoard**
- Startup preference → prioritize AngelList, **YourBoard**
```

### 2. Add to Supported Boards List

```markdown
**Primary Boards (v1):**
- LinkedIn (requires login)
- Indeed (no login required)
- **YourBoard** (login status, any notes)
```

### 3. Document Search Process

Add specific instructions for how to search your board. Claude needs to know:

- URL pattern for job search
- How to apply filters (remote, salary, location)
- How to extract job listings from the page
- Any authentication requirements
- Rate limiting behavior

Example board documentation format:

```markdown
#### YourBoard Search Process

1. Navigate to `https://yourboard.com/jobs`
2. Fill search form:
   - Job title field: `input[name="title"]`
   - Location field: `input[name="location"]`
   - Remote filter: Click "Remote" chip
3. Apply filters:
   - Salary: Select from dropdown
   - Posted date: Click "Past week"
4. Extract results:
   - Job cards: `.job-card`
   - Title: `.job-card h2`
   - Company: `.job-card .company-name`
   - Link: `.job-card a.apply-link`
5. Notes:
   - Requires login for salary data
   - Rate limits after ~50 requests
```

### 4. Add Board ID Prefix

Jobs are deduplicated by ID. Each board needs a unique prefix:

```json
{
  "id": "yourboard_12345",
  "source": "yourboard"
}
```

### 5. Test the Integration

1. Start a Cowork session with the skill
2. Create a test profile
3. Run a search on your new board
4. Verify:
   - Jobs are found and displayed
   - Deduplication works (run twice, second should skip seen jobs)
   - Results save to `searches/results/`

## Improving the Matching Algorithm

The job matching algorithm is in "Mode 2: Job Search" under "Job Matching Algorithm".

### Current Components

| Component | Weight | Description |
|-----------|--------|-------------|
| Keyword Match | 25% | Direct skill/title matches |
| Semantic Match | 35% | Claude reasons about transferability |
| Preference Match | 25% | Location, remote, salary alignment |
| Priority Alignment | 15% | Company vs user priorities |

### Adding New Signals

To add a new matching signal:

1. Define the signal and its weight
2. Describe how Claude should evaluate it
3. Add to score breakdown display

Example addition:

```markdown
5. **Company Reputation** (10%): Glassdoor rating if available
   - 4.0+ stars → +10 points
   - 3.0-4.0 stars → +5 points
   - Below 3.0 → 0 points
```

### Learning from Feedback

The skill tracks positive/negative signals when users mark jobs:

```json
// analytics/dashboard.json
{
  "match_learning": {
    "positive_signals": [
      {"job_id": "...", "keywords": ["distributed systems"], "company_size": "mid"}
    ],
    "negative_signals": [
      {"job_id": "...", "keywords": ["on-call"], "company_size": "enterprise"}
    ]
  }
}
```

To improve learning:

1. Identify patterns in positive vs negative signals
2. Add logic to boost/penalize similar jobs
3. Document the learning behavior in SKILL.md

## Adding Interview Question Templates

Interview prep questions are in "Mode 4: Interview Prep".

### Question Categories

| Category | Description |
|----------|-------------|
| Behavioral | STAR format questions about past experience |
| Technical | Role-specific technical questions |
| Role-specific | Questions about the specific job |
| Culture fit | Company values and culture questions |
| Questions to ask | Questions the user should ask |

### Adding Questions by Role Type

Add role-specific question templates:

```markdown
#### Engineering Manager Questions

**Behavioral:**
- Tell me about a time you had to make a difficult decision about team priorities
- Describe how you handled a conflict between team members
- How have you developed an underperforming engineer?

**Technical:**
- How do you balance technical debt with feature development?
- Walk me through how you would architect a new system
- How do you stay technical while managing?

**Culture Fit:**
- How do you build trust with a new team?
- What's your management philosophy?
```

### Adding Industry-Specific Questions

```markdown
#### Finance Industry Questions

- Explain your understanding of [relevant regulation]
- How do you handle working in a heavily regulated environment?
- What experience do you have with [industry-specific tool]?
```

## Code Style Guidelines

### SKILL.md Structure

1. **Frontmatter** - name, description, triggers
2. **Overview** - single paragraph summary
3. **Intent Detection** - how modes are selected
4. **Prerequisites** - browser requirements
5. **Workspace Structure** - file layout
6. **Session Start Logic** - returning user flow
7. **Modes** - each functional mode (Interview, Search, Apply, etc.)
8. **Error Handling** - failure cases
9. **Privacy Notes** - data handling

### Writing Guidelines

- Use clear headers and subheaders
- Include example JSON for data structures
- Provide code blocks for file paths and selectors
- List explicit steps for multi-step processes
- Note when user confirmation is required

### JSON Schema Guidelines

When defining new data structures:

```json
{
  "field_name": "type|option1|option2 or description",
  "nested": {
    "sub_field": "..."
  },
  "array_field": ["example1", "example2"]
}
```

## Testing Your Changes

### Manual Testing Workflow

1. **Fresh user flow**
   - Delete any existing profile/
   - Upload test resume
   - Complete interview
   - Verify profile.json created correctly

2. **Job search**
   - Trigger search on target board
   - Verify results display with scores
   - Check deduplication on second search

3. **Returning user**
   - Start new session
   - Verify profile loads
   - Test intent detection with various phrases

4. **Edge cases**
   - Malformed resume
   - Network failures
   - Rate limiting
   - Missing optional fields

### Test Scenarios

| Scenario | Expected Behavior |
|----------|-------------------|
| No profile exists | Start interview mode |
| Profile exists, "find jobs" | Start search mode |
| Profile exists, ambiguous message | Show action menu |
| Browser unavailable | Show error, suggest setup |
| Search returns no results | Suggest broadening criteria |
| Duplicate job found | Skip and report as already seen |

## Pull Request Guidelines

### Before Submitting

- [ ] Test with fresh profile (no existing data)
- [ ] Test returning user flow
- [ ] Test browser automation on target boards
- [ ] Update README if adding user-facing features
- [ ] Add yourself to contributors if first PR

### PR Description Template

```markdown
## What

Brief description of the change

## Why

Motivation for the change

## Testing

How you tested the change

## Screenshots

If relevant, show the change in action
```

### Review Process

1. Maintainer reviews within 1 week
2. Address feedback
3. Maintainer merges when approved

## Questions?

Open an issue for:
- Feature requests
- Bug reports
- Questions about implementation

Thanks for contributing!
