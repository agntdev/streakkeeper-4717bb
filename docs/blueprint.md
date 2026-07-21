# StreakKeeper — Bot specification

**Archetype:** custom

**Voice:** encouraging and concise — write every user-facing message, button label, error, and empty state in this voice.

StreakKeeper is a private Telegram bot that helps users build habits through gentle local-time reminders, one-tap check-ins, and progress tracking. It supports daily, weekday, or X-times-per-week schedules, tracks current and best streaks, and celebrates milestones with non-shaming encouragement. All data is private to each user.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual users who want a lightweight habit tracker inside Telegram
- People who prefer discrete, encouraging reminders and simple one-tap interaction on mobile

## Success criteria

- Users can create and manage habits with one-tap check-ins
- Users receive timely local-time reminders for their habits
- Streak tracking and milestone notifications are delivered without shaming language
- All user data remains private and accessible only to the user's Telegram account

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu with habit overview
- **Add habit** (button, actor: user, callback: habit:add) — Start the habit creation wizard
  - inputs: habit name, schedule type, reminder time, optional description
  - outputs: New habit added to user's list
- **My habits** (button, actor: user, callback: habits:list) — Show all active and paused habits with status
  - outputs: Compact list of habits with current streaks and next reminders
- **Weekly summary** (button, actor: user, callback: summary:weekly) — Request a weekly progress summary
  - outputs: Weekly summary showing check-in history and completion percentage

## Flows

### Onboarding
_Trigger:_ /start

1. Greet user and ask for timezone
2. Show quick tutorial about habit creation and check-ins
3. Display initial habit list (empty)

_Data touched:_ User

### Habit creation
_Trigger:_ habit:add

1. Ask for habit name
2. Select schedule type (daily/weekdays/X times per week)
3. Choose specific days if needed
4. Set reminder time
5. Add optional description
6. Confirm and save habit

_Data touched:_ Habit

### Daily reminder
_Trigger:_ scheduled_reminder

1. Send reminder message with check-in buttons
2. Process user response (Done/Missed/Snooze/Pause)

_Data touched:_ Check-in record, Habit

### Manual edit
_Trigger:_ habit:edit

1. Select habit to edit
2. Choose edit action (mark day, change reminder, pause/resume, delete)
3. Confirm changes

_Data touched:_ Habit, Check-in record

### Weekly summary
_Trigger:_ summary:weekly

1. Generate summary of check-in history
2. Calculate current and best streaks
3. Show completion percentage
4. Add encouraging message

_Data touched:_ Check-in record, Habit

### Milestone notification
_Trigger:_ milestone:achieved

1. Detect milestone (7/14/21/30/90 days)
2. Send milestone message with encouraging text

_Data touched:_ Habit

### Missed day nudge
_Trigger:_ missed_day

1. Detect missed day
2. Send encouraging nudge message

_Data touched:_ Habit

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram account with timezone and preferences
  - fields: telegram_id, timezone, preferences
- **Habit** _(retention: persistent)_ — User-defined habit with schedule and status
  - fields: id, title, description, schedule_type, target_days, reminder_time, active, created_at
- **Check-in record** _(retention: persistent)_ — Daily check-in status for a habit
  - fields: habit_id, date, status, timestamp
- **Streak summary** _(retention: transient)_ — Calculated streak statistics for a habit
  - fields: habit_id, current_streak, best_streak, completion_percentage
- **Milestone** _(retention: persistent)_ — Achieved habit milestones (7/14/21/30/90 days)
  - fields: habit_id, milestone_days, achieved_at

## Integrations

- **Telegram** (required) — Bot API messaging and user interaction
- **Timezone detection** (required) — Determine user's local time for reminders
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure milestone thresholds
- Set default snooze duration
- Enable/disable automatic weekly summaries
- Set opt-out for milestone notifications

## Notifications

- Daily reminders with check-in buttons
- Milestone achievement messages
- Encouraging nudges after missed days
- Weekly summary notifications

## Permissions & privacy

- All user data is private and accessible only to the user's Telegram account
- No social sharing or public data
- Users can delete their data at any time

## Edge cases

- User changes timezone after habit creation
- Multiple check-in attempts for same day
- Missed day after snooze period
- Habit schedule changes mid-week

## Required tests

- Verify one-tap check-in prevents duplicate entries
- Test reminder scheduling across timezones
- Validate streak calculations after missed days
- Confirm milestone notifications are non-shaming

## Assumptions

- Users will provide accurate timezone information during onboarding
- Telegram's API will handle scheduled messages reliably
- Users will understand the difference between 'missed' and 'failed' statuses
