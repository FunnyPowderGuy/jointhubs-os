# School Diary Skill

> Domain knowledge for scraping the school online diary and syncing lessons to Google Calendar.

## Configuration

Before using the School Diary Sync agent, fill in these values:

```
School diary URL:   https://your-school-diary.example.com   ← REPLACE THIS
Calendar name:      School Lessons                           ← REPLACE with your Google Calendar name
Timezone:           Europe/Warsaw                            ← adjust if needed
Default week start: Monday
```

Store credentials in a `.env` file in the project root (never in this file):

```
DIARY_USERNAME=your_login
DIARY_PASSWORD=your_password
```

## Supported Diary Systems

Common Polish school diary platforms (e-dziennik):

| System | Login URL pattern | Notes |
|--------|-------------------|-------|
| **Librus Synergia** | `https://synergia.librus.pl` | Most common in Poland; requires cookie-based session |
| **UONET+ / Vulcan** | `https://{school}.uonetplus.vulcan.net.pl` | Used in many gmina schools |
| **mobiDziennik** | `https://www.mobidziennik.pl` | Login via email/password |
| **iDziennik** | Varies by school | Often behind a school-specific domain |

Set your diary system type here: **`REPLACE_WITH_YOUR_SYSTEM`**

## Timetable Structure

### What to Scrape Per Lesson

For each lesson slot in the timetable extract:

| Field | Example | Notes |
|-------|---------|-------|
| `date` | `2025-06-16` | ISO format |
| `day_of_week` | `Monday` | |
| `slot_number` | `3` | Lesson number in the school day (1st, 2nd, ...) |
| `start_time` | `09:50` | 24h format |
| `end_time` | `10:35` | 24h format |
| `subject` | `Mathematics` | Full subject name |
| `teacher` | `Nowak Jan` | Last name first in Polish diaries |
| `room` | `204` | Room/classroom |
| `status` | `normal` | `normal` / `substitution` / `cancelled` / `moved` |
| `notes` | `Topic: Quadratic equations` | Optional lesson notes or topic |

### Status Values

| Status | Meaning | Calendar action |
|--------|---------|-----------------|
| `normal` | Regular lesson | Create/keep event |
| `substitution` | Different teacher or subject | Update event, prefix `🔄` |
| `cancelled` | Lesson not happening | Prefix title with `❌`, keep event so student sees it |
| `moved` | Lesson at different time | Delete old slot, create new one |

## Scraping Approach

### Librus Synergia

1. POST to `https://synergia.librus.pl/loguj` with `login` and `haslo` fields
2. Navigate to `https://synergia.librus.pl/przegladaj_plan_lekcji`
3. Parse the HTML timetable table — rows = time slots, columns = days of the week
4. Check for coloured cells (substitutions = orange/yellow, cancellations = red/grey)

### UONET+ / Vulcan

1. Use the REST API if available: `GET /api/mobile/register/timetable?dateFrom=YYYY-MM-DD`
2. Or scrape the web view under `Dziennik → Plan lekcji`
3. JSON response includes `Lesson.Subject`, `Lesson.Teacher`, `Lesson.Room`, `Lesson.Change`

### Generic / Unknown System

1. Open the diary URL with Playwright
2. Locate the timetable section (look for a `<table>` with time slots)
3. Screenshot the page and ask the user to identify the relevant table
4. Extract text from cells, parse subject/teacher/room using regex patterns

## Google Calendar Integration

### Setup

The agent uses the `googleWorkspace` MCP server configured in `.vscode/mcp.json`.

See `.vscode/mcp.json.example` for the required OAuth credentials format.

### Calendar Naming

Create a dedicated Google Calendar for school lessons — e.g. **"School Lessons"** — to avoid mixing with personal events.

Set the name here: **`REPLACE_WITH_YOUR_CALENDAR_NAME`**

### Event Fields

```json
{
  "summary": "Mathematics — Nowak Jan",
  "location": "Room 204",
  "start": { "dateTime": "2025-06-16T09:50:00", "timeZone": "Europe/Warsaw" },
  "end":   { "dateTime": "2025-06-16T10:35:00", "timeZone": "Europe/Warsaw" },
  "description": "Subject: Mathematics\nTeacher: Nowak Jan\nRoom: 204\nStatus: normal\nSynced: 2025-06-14T10:00:00Z",
  "colorId": "5"
}
```

### Color Coding

| Status | Google Calendar colorId | Color |
|--------|------------------------|-------|
| `normal` | `5` | Banana (yellow) |
| `substitution` | `6` | Sage (green) |
| `cancelled` | `11` | Tomato (red) |

## Typical School Day (Poland)

Adjust if your school has different hours:

| Slot | Start | End |
|------|-------|-----|
| 1 | 08:00 | 08:45 |
| 2 | 08:50 | 09:35 |
| 3 | 09:50 | 10:35 |
| 4 | 10:45 | 11:30 |
| 5 | 11:35 | 12:20 |
| 6 | 12:25 | 13:10 |
| 7 | 13:15 | 14:00 |
| 8 | 14:05 | 14:50 |

## Deduplication Key

To match calendar events with scraped lessons, use this key:

```
{ISO_date}_{start_time}_{normalized_subject}
Example: 2025-06-16_09:50_mathematics
```

Normalize subject names: lowercase, strip diacritics, trim whitespace.

## Known Quirks

- **Librus date format**: Returns dates as `DD.MM.YYYY` — convert to ISO before processing
- **Teacher name order**: Polish diaries often use `Lastname Firstname` — store as-is, don't reorder
- **Empty cells**: An empty cell in the timetable grid means no lesson in that slot (not cancelled)
- **Multiple classes**: If the student is in a split group (e.g. language groups), the diary may show overlapping lessons — keep all of them
- **Holiday weeks**: The diary may show an empty or unavailable timetable for holidays — detect and skip gracefully

## Related Skills

- [agentic-engineering](../../.github/skills/agentic-engineering/SKILL.md) — System architecture reference
- [project-context](../../.github/skills/project-context/SKILL.md) — If you want to track this as a project
