# Pomodoro – Requirements

## Glossary

| Term | Meaning |
|---|---|
| Focus session | A timed work period (default 25 min) |
| Short break | A short rest after a focus session (default 5 min) |
| Long break | A longer rest after every 4th completed focus session (default 15 min) |
| Completed session | A session whose timer reached 0 (not reset early) |
| Daily goal | Number of focus sessions the user wants to complete per day |

## Architecture note

The timer runs in the frontend (React). The backend (ASP.NET Core) only records
completed sessions and stores settings.

---

## User stories (MVP)

### US-1: Run a focus session

> As a user, I want to start a 25-minute focus timer, so that I can work without distraction.

Acceptance criteria:
- The timer shows the remaining time as `MM:SS`.
- I can pause and resume the timer, and it continues from where it stopped.
- When it reaches 0, I get a sound or notification.

### US-2: Record completed sessions

> As a user, I want completed focus sessions to be saved, so that I can see my progress.

Acceptance criteria:
- A session is saved only when it finishes, not when it's reset early.
- A saved session has a start time, a duration and a type (focus, short break or long break).
- A session's duration must be greater than 0.

### US-3: Daily focus goal

> As a user, I want to set a daily goal of focus sessions, so that I know how many I still need to complete today.

Acceptance criteria:
- I can set the goal to a whole number from 1 to 20.
- The default goal is 8.
- The UI shows my progress as `completed / goal` (for example `3 / 8`).
- Only completed focus sessions count; breaks don't.
- The progress resets at midnight (local time).
- When I reach the goal, I see a message.

### US-4: Long break

> As a user, I want a longer break after several focus sessions, so that I can recover properly.

Acceptance criteria:
- After every 4th completed focus session of the day, the suggested break is a long break (15 min).
- After all other focus sessions, the suggested break is a short break (5 min).
- Focus sessions that were reset early don't count toward the 4.
- Breaks never count toward the 4.
- The count restarts at midnight (local time).
- I can skip a break and start the next focus session directly.

---

## Functional requirements

### FR-1: Session types and durations

The system supports three session types with these default durations:
- Focus: 25 minutes
- Short break: 5 minutes
- Long break: 15 minutes

### FR-2: Long-break rule

After every 4th completed focus session, the next break is a long break.
All other breaks are short breaks.

---

## Non-functional requirements

### NFR-1: Timer accuracy

The displayed time deviates by at most 1 second over a 25-minute session,
including when the browser tab is in the background.

### NFR-2: Code quality

The backend builds with 0 warnings. All business rules in `Pomodoro.Core` are
covered by unit tests, and CI rejects pull requests that fail the build, tests
or `dotnet format --verify-no-changes`.

---

## Later / out of scope

- Custom session durations
- Task list linked to sessions
- Weekly and monthly statistics with charts
- User accounts and login
- Dark mode
- Keyboard shortcuts