# Canvas MCP for Poke

Custom MCP server that uses the Canvas LMS API to surface grades, deadlines, assignments, announcements, and calendar context inside [Poke](https://poke.com), the AI assistant I use day to day for managing my email, to‑do list, and academic schedule by text.

## Overview

- **Authentication:** Canvas Personal Access Token outbound, API key (Bearer or `x-api-key`) inbound from Poke
- **Data access:** read‑only on a Canvas student account
- **Transport:** Streamable HTTP, served by FastMCP at `/mcp`

## What I can ask Poke

- "What are my upcoming deadlines?"
- "What are my current grades?" *(new, see `get_grades_overview` below)*
- "What announcements were posted recently?"
- "What does my upcoming week look like?"
- "What does my academic day today look like?"
- "Did any of my assignments get graded recently?"

## Setup

### Canvas access token

1. Log in to Canvas
2. Account → Settings
3. Scroll to **Approved Integrations**
4. **+ New Access Token** and copy the token (it is only shown once)

### Local run

```bash
git clone https://github.com/temujinkz/poke-canvas-mcp
cd poke-canvas-mcp

python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

cp .env.example .env
# Fill in CANVAS_BASE_URL, CANVAS_ACCESS_TOKEN, POKE_API_KEY (use 32+ random chars)

python src/server.py
# In a second terminal:
npx @modelcontextprotocol/inspector
```

Server runs at `http://localhost:8000/mcp`. Connect using the **Streamable HTTP** transport.

### Environment variables

```bash
CANVAS_BASE_URL=https://your_school.instructure.com
CANVAS_ACCESS_TOKEN=your_canvas_token
POKE_API_KEY=your_strong_api_key
```

## Deploying

`render.yaml` is included for one‑click Render deploy:

1. Fork the repo
2. Create a new Web Service on Render and connect the fork
3. Set `CANVAS_BASE_URL`, `CANVAS_ACCESS_TOKEN`, `POKE_API_KEY` in the env
4. Deploy

## Wiring it into Poke

Add the deployed MCP URL at [poke.com/settings/connections](https://poke.com/settings/connections), with your `POKE_API_KEY` so Poke can authenticate. Header form: `x-api-key: <key>` or `Authorization: Bearer <key>`.

Quick test prompts:

- "What do I need to do today on canvas?"
- "What's my current grade in [course]?"
- "Use Canvas MCP connection's get_today_summary tool"

## MCP tools

| Tool | Purpose |
|---|---|
| `get_today_summary` | Daily check‑in: deadlines + announcements + new grades + overdue items in a 24–48 h window |
| `get_upcoming_assignments` | Aggregated upcoming/overdue assignments across active dashboard courses, urgent first |
| `get_recent_announcements` | Recent announcements, optionally with full bodies for summarization |
| `get_recently_graded` | Newly graded items via the Canvas planner feed |
| `get_grades_overview` | **(new)** Current and final grade/score per active dashboard course |
| `get_week_ahead` | Assignments, deadlines, announcements, and calendar events for the upcoming week |
| `list_courses_raw` | Raw Canvas course list (active enrollments) |
| `get_dashboard_cards` | Lightweight active dashboard course list, optionally filtered by term prefix (`26SS`, `26FS`) |
| `get_course_assignments` | Upcoming (and optionally overdue) assignments for a single course id |

### `get_grades_overview` (added in this fork)

Calls `GET /api/v1/courses?include[]=total_scores` and merges the response with the dashboard course list to return only courses the user actually has on their dashboard, in dashboard order:

```json
[
  {
    "course_id": 12345,
    "course_name": "CSCI 100 Algorithms",
    "current_grade": "A-",
    "current_score": 91.7,
    "final_grade": null,
    "final_score": null,
    "html_url": "https://your_school.instructure.com/courses/12345/grades"
  }
]
```

Canvas only exposes `computed_current_*` / `computed_final_*` once the instructor has graded enough work and the course settings allow students to view total grades. When that's not yet the case, those fields come back `null`.

## Why this exists

Keeping up with Canvas notifications, deadlines, and grade releases on top of personal email, school email, and a to‑do list is genuinely hard mid‑semester. Putting all of that behind a single Poke conversation that I can text from anywhere makes it manageable.

## License

MIT. See `LICENSE`.
