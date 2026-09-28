   # Pomodoro – Design

Based on [requirements.md](requirements.md). Terms follow the glossary there.

## 1. Architecture

```
React frontend  ──HTTP/JSON──►  Pomodoro.Api  ──►  Pomodoro.Core
(runs the timer)                (endpoints, DTOs)   (domain model + rules)
```

- The **timer runs in the frontend**. The backend never ticks; it only stores
  completed sessions and settings, and computes statistics.
- **Pomodoro.Core** has no dependency on ASP.NET, the database or the system clock.
  All business rules live here and are unit-tested.
- **Pomodoro.Api** only translates HTTP to Core calls and back.

## 2. Domain model (Pomodoro.Core)

### SessionType (enum)

`Focus` | `ShortBreak` | `LongBreak`

### Session (entity)

| Property | Type | Notes |
|---|---|---|
| `Id` | `Guid` | identity, generated on creation |
| `Type` | `SessionType` | |
| `StartedAt` | `DateTimeOffset` | keeps the UTC offset, needed for "today" |
| `Duration` | `TimeSpan` | |

Invariants (enforced in the constructor / factory, so an invalid session cannot exist):
- `Duration > 0`
- `Type` is a defined `SessionType` value

### Settings

| Property | Type | Notes |
|---|---|---|
| `DailyGoal` | `int` | default 8 |

Invariant: `1 ≤ DailyGoal ≤ 20`

### Rules (pure functions)

| Rule | Input | Output | Source |
|---|---|---|---|
| `NextBreakType` | completed focus sessions today | `ShortBreak` or `LongBreak` | FR-2 |
| `DailyProgress` | sessions of today, daily goal | completed, goal, goal reached | US-3 |

Pure = same input gives same output, no database, no clock. This makes them
testable in milliseconds.

## 3. API (Pomodoro.Api)

All routes use the `/api` prefix. JSON uses camelCase; enums are sent as strings.

| Method | Route | Purpose | Success | Error |
|---|---|---|---|---|
| `POST` | `/api/sessions` | save a completed session (US-2) | `201 Created` | `400` invalid data |
| `GET` | `/api/sessions?date=YYYY-MM-DD` | list sessions of a day | `200 OK` | `400` invalid date |
| `GET` | `/api/stats/today` | daily progress and next break type | `200 OK` | |
| `GET` | `/api/settings` | read settings | `200 OK` | |
| `PUT` | `/api/settings` | update daily goal (US-3) | `200 OK` | `400` goal outside 1–20 |

### Example: `POST /api/sessions`

Request:

```json
{
  "type": "focus",
  "startedAt": "2026-09-28T14:00:00+02:00",
  "durationSeconds": 1500
}
```

Response `201 Created` (with `Location: /api/sessions/{id}`):

```json
{
  "id": "3f2b8c1e-5a7d-4e2b-9c1a-2d4e6f8a0b1c",
  "type": "focus",
  "startedAt": "2026-09-28T14:00:00+02:00",
  "durationSeconds": 1500
}
```

### Example: `GET /api/stats/today`

```json
{
  "completedFocusSessions": 3,
  "dailyGoal": 8,
  "goalReached": false,
  "nextBreakType": "shortBreak"
}
```

### Status code rules

- `201` when something new is stored, `200` for reads and updates.
- `400` for invalid client input, with a validation problem body (RFC 9457).
- `500` only for real bugs, never for invalid input.

## 4. Design decisions

### D-1: The client sends `startedAt` and `durationSeconds`

The session is recorded **after** it finished, so the server cannot know when it
started. Taking the server time on request arrival would be wrong because:
- the request arrives at the **end** of the session, not the start;
- a paused session has a real duration shorter than end − start;
- if the network is down, the frontend can retry later and the times stay correct;
- only the client knows the user's local time zone, which decides what "today" is.

The server still validates the data (duration > 0, start not in the future).

### D-2: DTOs are separate from domain types

API request/response records live in Pomodoro.Api, domain types in Pomodoro.Core.
The API contract and the domain can change independently.

### D-3: `DateTimeOffset` instead of `DateTime`

`DateTime` loses the offset and causes time-zone bugs around midnight.
