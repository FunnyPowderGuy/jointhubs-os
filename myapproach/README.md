# myapproach

This folder contains my personal AI agent setup for automating school schedule management.

## What's Here

| File | Purpose |
|------|---------|
| [`school-diary-sync.agent.md`](school-diary-sync.agent.md) | Agent that scrapes the school diary and syncs lesson changes to Google Calendar |
| [`skills/school-diary/SKILL.md`](skills/school-diary/SKILL.md) | Domain knowledge — how to read the diary, detect changes, and format calendar events |

## How It Works

1. The **School Diary Sync** agent opens the school's online diary (e-dziennik / e-diary)
2. It scrapes the current week's (or next week's) timetable
3. It compares the schedule against what's already in Google Calendar
4. It creates, updates, or deletes calendar events to reflect the latest lessons, substitutions, and cancellations

## Setup

Before using the agent:

1. Configure the Google Workspace MCP server — see [`.vscode/mcp.json.example`](../.vscode/mcp.json.example)
2. Set your school diary URL in the skill file: [`skills/school-diary/SKILL.md`](skills/school-diary/SKILL.md)
3. Select the **School Diary Sync** agent in Copilot Chat

## Dependencies

- Google Calendar access via `googleWorkspace` MCP server
- Web browsing via Playwright (`playwright/*` tools) for scraping the diary
