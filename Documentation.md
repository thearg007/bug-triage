## 1. Overview

### Purpose

Bug Triage Agent automates GitHub issue prioritization using AI. It classifies bugs into P0-P3 priorities, calculates severity scores, detects duplicates, and can apply labels directly to GitHub.

### Tech Stack

- **Frontend:** Streamlit
- **AI Analysis:** Google Gemini
- **Compression:** ScaleDown
- **Issue Source:** GitHub API

---

## 2. Architecture

### Flow

1. Fetch issues from GitHub
2. Compress context via ScaleDown
3. Analyze with Gemini AI
4. Parse response and calculate scores
5. Display results
6. Optionally apply labels to GitHub

### File Structure

- [**app.py**](http://app.py/) - Main application
- **.env** - API keys
- **requirements.txt** - Dependencies
- [**README.md**](http://readme.md/) - User guide
- [**DOCUMENTATION.md**](http://documentation.md/) - This file

---

## 3. API Integrations

### GitHub API

**Fetch open issues**

- Endpoint: GET /repos/{owner}/{repo}/issues
- Parameters: state=open, per_page=25

**Fetch all issues (for duplicates)**

- Endpoint: GET /repos/{owner}/{repo}/issues
- Parameters: state=all, per_page=50

**Add labels**

- Endpoint: POST /repos/{owner}/{repo}/issues/{num}/labels
- Body: labels array with priority and team

**Add comment**

- Endpoint: POST /repos/{owner}/{repo}/issues/{num}/comments
- Body: markdown formatted triage summary

**Remove label**

- Endpoint: DELETE /repos/{owner}/{repo}/issues/{num}/labels/{name}

### ScaleDown API

- Endpoint: POST https://api.scaledown.xyz/compress/raw/
- Headers: x-api-key, Content-Type application/json
- Body: context, prompt, model, scaledown rate

### Gemini API

- Endpoint: POST [https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent](https://generativelanguage.googleapis.com/v1beta/models/%7Bmodel%7D:generateContent)
- Parameters: key (API key)
- Body: contents with text, generationConfig with temperature and maxOutputTokens

---

## 4. Function Reference

### Helper Functions

**clean_repo(url)**
Normalizes repo URL to owner/repo format. Returns string.

**check_write_access(repo)**
Checks if GitHub token has write permissions. Returns boolean.

### API Functions

**get_open_issues(repo, limit=25)**
Fetches open issues from GitHub. Returns tuple of issues list, context string, and error.

**get_all_issues(repo, limit=50)**
Fetches all issues for duplicate detection. Returns list.

**compress_context(context)**
Compresses context via ScaleDown API. Returns tuple of data dict and error.

**get_default_model()**
Detects available Gemini model. Returns model name string.

**analyze_all_bugs(compressed, model, issues)**
Classifies all bugs using Gemini AI. Returns tuple of result dict and error.

**get_fix_suggestion(bug, issue, model)**
Generates AI fix recommendation. Returns tuple of suggestion text and error.

**find_duplicates(issue, all_issues, model)**
Finds similar issues using AI. Returns list of duplicates.

**apply_triage(repo, num, priority, team, dupes)**
Applies labels and comments to GitHub. Returns tuple of success boolean and message.

### Analysis Functions

**classify_by_keywords(issue)**
Keyword-based fallback classification. Returns tuple of priority and team.

**parse_ai_response(text, issues)**
Parses AI JSON with fallback. Returns tuple of result dict and error.

**calculate_severity_score(priority, title, body)**
Calculates 0-100 severity score. Returns integer.

**estimate_resolution_time(priority, score)**
Estimates fix time. Returns string like "2-4 hrs".

### UI Functions

**apply_custom_style()**
Applies dark GitHub-inspired CSS theme.

**show_metrics(data)**
Displays compression statistics in 4 columns.

**get_severity_color(score)**
Returns emoji based on score. Red for 80+, orange for 60+, yellow for 35+, green for below 35.

**render_bug(bug, issue)**
Renders single bug with details and progress bar.

---

## 5. Data Structures

### Issue Object

- **number** - GitHub issue number (integer)
- **title** - Issue title (string)
- **body** - Issue description, truncated (string)
- **url** - GitHub issue URL (string)
- **state** - "open" or "closed", only in get_all_issues (string)

### Bug Object

- **number** - GitHub issue number (integer)
- **priority** - P0, P1, P2, or P3 (string)
- **team** - Backend, Frontend, or Documentation (string)
- **summary** - Brief description (string)
- **severity_score** - 0-100 score (integer)
- **est_time** - Time estimate like "2-4 hrs" (string)
- **duplicates** - List of similar issues with number and similarity

### Session State

- **result** - Analysis results with bugs list
- **issues** - Fetched GitHub issues
- **data** - Compression metrics
- **repo_url** - Current repository
- **has_write_access** - GitHub write permission boolean
- **model** - Active Gemini model name

---

## 6. Classification Logic

### Priority Definitions

**P0 - Critical (2-4 hrs)**
Keywords: crash, freeze, payment, security, data loss

**P1 - High (1-2 days)**
Keywords: memory leak, timeout, slow, broken, error 500

**P2 - Medium (3-5 days)**
Keywords: safari, firefox, chrome, css, layout, ui

**P3 - Low (1-2 weeks)**
Keywords: typo, spelling, readme, documentation, copyright

### Severity Scoring Formula

- Base score: P0=85, P1=65, P2=40, P3=15
- Add 10 for each critical keyword found
- Add 5 for each high keyword found
- Maximum score: 100

### Classification Flow

1. Send issues to Gemini AI with classification rules
2. Parse JSON response, extract priority and team
3. Apply keyword overrides (payment forces P0, typo forces P3)
4. If AI fails, fallback to keyword-only classification
5. Calculate severity score and time estimate

---

## 7. Session Management

### State Flow

1. User clicks Analyze button
2. App fetches issues from GitHub
3. Context gets compressed via ScaleDown
4. Gemini AI analyzes and classifies bugs
5. Results stored in st.session_state
6. UI renders from session state

### Caching

- **get_open_issues()** - cached 5 minutes
- **get_all_issues()** - cached 5 minutes

Caching reduces GitHub API calls on Streamlit re-renders.

---


## 8. Troubleshooting

### "Repository not found"

**Cause:** Invalid format or private repo without access.

**Fix:** Use owner/repo format. Add GITHUB_TOKEN for private repos.

### "API keys missing"

**Cause:** .env file not configured.

**Fix:** Create .env with SCALEDOWN_API_KEY and GEMINI_API_KEY.

### "Compression skipped"

**Cause:** ScaleDown API error or timeout.

**Fix:** Check API key. App still works without compression.

### "Read-only access"

**Cause:** GitHub token lacks write permissions.

**Fix:** Generate new token with repo scope.

### Wrong priorities assigned

**Cause:** AI interpretation varies.

**Fix:** Keyword overrides auto-correct common cases. Payment always becomes P0, typo always becomes P3.

### Environment Variables

- **SCALEDOWN_API_KEY** - Required for compression
- **GEMINI_API_KEY** - Required for AI analysis
- **GITHUB_TOKEN** - Optional, enables write access for labels and comments

### Rate Limits

- **GitHub without token:** 60 requests per hour
- **GitHub with token:** 5000 requests per hour
- **Gemini and ScaleDown:** Varies by plan

---

## 9. Label Schema

### Priority Labels

- **P0** (Red) - Critical, fix immediately
- **P1** (Orange) - High, fix this week
- **P2** (Yellow) - Medium, backlog
- **P3** (Green) - Low, when time permits

### Team Labels

- **Backend** - API, database, server issues
- **Frontend** - UI, CSS, browser compatibility
- **Documentation** - Typos, README, docs

---

## 10. API Keys Setup

### ScaleDown API Key

1. Visit scaledown.xyz
2. Sign up for an account
3. Navigate to API Keys section
4. Generate a new API key

### Gemini API Key

1. Visit Google AI Studio ([makersuite.google.com/app/apikey](http://makersuite.google.com/app/apikey))
2. Sign in with your Google account
3. Create a new API key

### GitHub Token (Optional)

1. Go to GitHub → Settings → Developer Settings
2. Click Personal access tokens → Tokens (classic)
3. Generate new token with repo scope for full access
4. Or use public_repo scope for public repos only