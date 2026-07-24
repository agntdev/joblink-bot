# JobLink — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A global job marketplace bot connecting employers to seekers via moderated job posts, search filters, and application tracking. Employers post vacancies with auto-moderation; seekers discover opportunities through search, saved alerts, and curated DMs.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Job seekers seeking global opportunities
- Employers/recruiters posting vacancies

## Success criteria

- 1000+ active job posts published monthly
- 5000+ seeker applications tracked
- 95% auto-moderation accuracy rate

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with job search/posting options
  - inputs: user role (seeker/employer)
  - outputs: main dashboard
- **/post_job** (command, actor: user, command: /post_job) — Start job posting wizard for employers
  - inputs: job title, salary range, employment type
  - outputs: guided form
- **/search** (command, actor: user, command: /search) — Open search interface with filters
  - inputs: keywords, location, employment type
  - outputs: search results
- **Browse Recent Jobs** (button, actor: user, callback: browse:recent) — Show paginated list of newest job posts
  - inputs: page number
  - outputs: job previews
- **Manage Alerts** (button, actor: user, callback: alerts:manage) — Create/edit saved search subscriptions
  - inputs: keywords, filters
  - outputs: alert settings

## Flows

### Job Posting Workflow
_Trigger:_ /post_job

1. Collect required fields via guided form
2. Run auto-moderation checks
3. Publish or reject with feedback
4. Send confirmation to employer

_Data touched:_ Job Post

### Job Search Flow
_Trigger:_ /search

1. Show search interface with quick filters
2. Display matching job previews
3. Show full details on selection
4. Enable application submission

_Data touched:_ Job Post, Saved Search

### Application Submission
_Trigger:_ View job details

1. Display application instructions
2. Collect application link/file
3. Forward to employer contact
4. Log application event

_Data touched:_ Application Event

### Auto-Moderation
_Trigger:_ Job post submission

1. Check for banned phrases
2. Validate required fields
3. Set post status (published/rejected)
4. Notify employer

_Data touched:_ Job Post

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Job Post** _(retention: persistent)_ — Vacancy listing with moderation status
  - fields: title, salary_range, employment_type, description, application_link, tags, location, status
- **Employer Account** _(retention: persistent)_ — Telegram account linked to job posts
  - fields: telegram_id, display_name, logo_url
- **Saved Search** _(retention: persistent)_ — Seeker alert preferences
  - fields: keywords, location_filter, employment_type, alert_frequency
- **Application Event** _(retention: persistent)_ — Record of seeker applications
  - fields: job_id, seeker_chat_id, application_method, timestamp

## Integrations

- **Telegram** (required) — Bot API messaging and public channel posting
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Moderation rule configuration
- Public channel posting toggle
- Alert frequency defaults

## Notifications

- DM to seekers when matching jobs post
- Admin alerts for rejected posts
- Application confirmation to seekers

## Permissions & privacy

- No persistent seeker profile data stored
- Application files not cached
- Public channel posts visible to all

## Edge cases

- Moderation rule false positives
- Application delivery failures
- Seeker unsubscribes from alerts

## Required tests

- End-to-end job posting workflow
- Search filter accuracy
- DM alert delivery reliability

## Assumptions

- Auto-moderation rules block spam phrases
- Public channel is optional but recommended
- Application forwarding works without storage
