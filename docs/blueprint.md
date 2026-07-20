# CryptoPing — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

CryptoPing is a private Telegram bot that lets users track cryptocurrency prices with customizable alerts for price thresholds and percentage changes over time intervals. It includes a morning summary feature, quiet hours suppression, and provides the owner with anonymized usage statistics and alert frequency metrics.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Cryptocurrency traders
- Crypto asset holders
- Telegram bot users seeking price alerts

## Success criteria

- Users can add/remove coins from their watchlist
- Price threshold and percent-change alerts trigger accurately
- Owner receives daily aggregate metrics about bot usage

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Begin onboarding and set up timezone
  - inputs: Telegram ID, timezone
  - outputs: main menu with coin options
- **/price** (command, actor: user, command: /price) — Check current price of specified coin or entire watchlist
  - inputs: ticker symbol
  - outputs: price data with timestamp and 24h change
- **Add Coin** (button, actor: user, callback: watchlist:add) — Add a new cryptocurrency to watchlist
  - inputs: coin ticker
  - outputs: coin added confirmation
- **Manage Alerts** (button, actor: user, callback: alerts:manage) — Configure or modify existing alert rules
  - inputs: alert type, parameters
  - outputs: alert settings confirmation
- **Settings** (button, actor: user, callback: settings:open) — Configure quiet hours, morning summary, and cooldown defaults
  - inputs: quiet hours window, summary time
  - outputs: updated user preferences

## Flows

### onboarding_flow
_Trigger:_ /start

1. Detect user's timezone
2. Offer manual timezone adjustment
3. Display main menu with coin buttons

_Data touched:_ user_profile

### add_coin_flow
_Trigger:_ watchlist:add

1. Request coin ticker (via button or text)
2. Resolve ticker to canonical symbol
3. Confirm addition with price data

_Data touched:_ watchlist_item, price_snapshot

### price_threshold_alert_flow
_Trigger:_ alert:price_threshold

1. Select coin from watchlist
2. Choose above/below direction
3. Enter target price
4. Set cooldown period

_Data touched:_ alert_rule

### percent_change_alert_flow
_Trigger:_ alert:percent_change

1. Select coin(s)
2. Enter percent threshold
3. Choose lookback interval
4. Set cooldown period

_Data touched:_ alert_rule

### morning_summary_flow
_Trigger:_ scheduled:summary

1. Check user's quiet hours status
2. Aggregate watchlist prices
3. Include triggered alerts from previous night

_Data touched:_ alert_history, price_snapshot

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user_profile** _(retention: persistent)_ — User-specific settings and metadata
  - fields: Telegram ID, display name, timezone, quiet_hours_start, quiet_hours_end, summary_time, summary_opt_in
- **watchlist_item** _(retention: persistent)_ — Monitored cryptocurrency with user-defined name
  - fields: ticker, friendly name, active status
- **alert_rule** _(retention: persistent)_ — Price alert configuration with suppression rules
  - fields: type, direction, target_price, percent_threshold, lookback_interval, cooldown, enabled
- **price_snapshot** _(retention: session)_ — Latest known price data for tickers
  - fields: ticker, price, timestamp
- **alert_history** _(retention: persistent)_ — Record of triggered alerts for user reference
  - fields: timestamp, price_before, price_after, percent_change, delivered_status
- **owner_metrics** _(retention: persistent)_ — Aggregate usage statistics for the bot owner
  - fields: total_users, active_users_30d, alert_type_counts, top_alerted_tickers

## Integrations

- **Telegram** (required) — Bot API messaging and notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Receive daily aggregate metrics via admin chat
- Configure top N most-triggered alerts to display
- Set admin chat for owner notifications

## Notifications

- Price alert notifications to users
- Morning summary with price changes
- Owner daily metrics report

## Permissions & privacy

- All user data is private and not shared
- Owner metrics are anonymized and aggregated
- Alert history contains only non-personal price data

## Edge cases

- Unrecognized ticker symbols with suggestions
- Price feed failures with retry logic
- Overlapping quiet hours and scheduled alerts

## Required tests

- Add coin flow with typo resolution
- Price threshold alert triggering and cooldown
- Morning summary delivery during quiet hours
- Alert queue management during suppression

## Assumptions

- Default lookback window is 1 hour
- Default cooldown is 1 hour
- Morning summary requires explicit opt-in
- Quiet hours default to 23:00-07:00
- Price feed failures are retried 3 times
