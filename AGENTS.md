# AI Agent Guidelines for Mergington High School Activities API

## Project Overview

A FastAPI web application for Mergington High School extracurricular activity management. Students can browse available activities and sign up via a simple frontend interface.

**Key Domain**: Activity management system with in-memory storage, no authentication/authorization.

## Quick Start

| Task | Command |
|------|---------|
| Install dependencies | `pip install -r requirements.txt` |
| Run server | `uvicorn src.app:app --reload` (with auto-reload) |
| Access API | http://localhost:8000/docs (Swagger documentation) |
| Run tests | `pytest` |
| Frontend | http://localhost:8000/ (redirects to `/static/index.html`) |

## Architecture

### Backend (FastAPI)
- **File**: [src/app.py](src/app.py)
- **Data**: In-memory Python dict (`activities`) — **persists only during server runtime**
- **Routes**: 
  - `GET /` → Redirect to static HTML
  - `GET /activities` → Returns all activities as JSON
  - `POST /activities/{activity_name}/signup?email={email}` → Sign up student
  - `GET /static/*` → Serve frontend files

### Frontend (Vanilla JS + HTML/CSS)
- **Files**: [src/static/](src/static/) (app.js, index.html, styles.css)
- **Logic**: Fetch activities on page load, populate dropdown and activity cards, handle form submission with fetch API
- **Integration Point**: Assumes activity names are URL-safe (frontend uses `encodeURIComponent()`)

## Key Conventions & Patterns

### Code Style
- **Python**: Simple linear app structure, single file (no blueprints/routers yet)
- **JavaScript**: Vanilla JS with event listeners, fetch API for HTTP calls
- **Naming**: kebab-case for HTML IDs (e.g., `activities-list`), camelCase for JS variables

### Data Model
- Activities indexed by **name** (string, assumed unique)
- Participants stored as flat list of email strings
- No Student/User objects yet
- Activity structure: `{description, schedule, max_participants, participants: [...]}`

### Frontend-Backend Contract
- Activity names may contain special characters — frontend must URL-encode them
- Email provided as query parameter (not request body): `/activities/{name}/signup?email={email}`
- Success/error messages displayed inline with 5-second auto-hide

## Critical Issues & Gotchas ⚠️

### Current Limitations
1. **No input validation** — Email parameter is not validated; activity existence check only via dict lookup
2. **No duplicate prevention** — Same email can sign up multiple times for same activity
3. **No capacity enforcement** — Signup succeeds even if activity is at max participants
4. **In-memory storage** — All data lost on server restart (no persistence)
5. **No CORS setup** — Works locally; may fail if frontend and backend on different domains
6. **No authentication** — Any email can sign up as any student

### When Adding Features
- Validate email format and activity names using Pydantic models
- Add capacity checks: `if len(activity["participants"]) >= activity["max_participants"]`
- Store duplicate signups correctly: check email not already in participants list before appending
- Use `HTTPException(status_code=409)` for conflict errors (e.g., already signed up)
- Test activity names with special characters: spaces, `/`, `&`, etc.

## File Locations Reference

| What | Where |
|------|-------|
| Main FastAPI app | [src/app.py](src/app.py) |
| Frontend HTML | [src/static/index.html](src/static/index.html) |
| Frontend JS | [src/static/app.js](src/static/app.js) |
| Frontend CSS | [src/static/styles.css](src/static/styles.css) |
| Dependencies | [requirements.txt](requirements.txt) |
| Test config | [pytest.ini](pytest.ini) |

## Testing Strategy

- **Current**: `pytest` command (Python path configured in pytest.ini to include `.`)
- **API testing**: Use httpx (included in requirements) to test endpoints
- **Frontend**: Manual browser testing recommended for UI/form validation
- **Test patterns**: Import app and test route handlers directly

## Before Making Changes

✅ **Always**:
- Run `uvicorn src.app:app --reload` to test locally
- Check FastAPI auto-generated docs at `/docs` after changes
- Verify frontend still works after backend changes
- URL-encode activity names in requests with special characters

❌ **Avoid**:
- Storing persistent data in memory (use file/database for real features)
- Adding new endpoints without validation (use Pydantic Query/Path)
- Assuming email uniqueness (multiple signups per email possible)
- Modifying activity data without checking capacity first
