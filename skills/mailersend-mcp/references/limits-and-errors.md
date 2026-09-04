# MailerSend MCP — limits, retention and errors

Server-side constraints that are not visible in the tool schemas. Most failed calls are caused by
one of these.

## Data retention

Activity and analytics have **different** retention windows. Confusing the two is a common cause of
rejected date ranges.

### Activity (`list_activities`, `get_activity`)

Retention is a per-plan entitlement and resolves to one of four tiers: **1, 7, 14 or 30 days**.
Free and trial accounts sit at the 1-day tier. A data retention add-on can raise an account's
value, so the tier does not follow from the plan name alone.

There is no MCP tool that reports the account's current retention window. Do not assume a value —
if a date range is rejected, the error message states the actual limit, which is the reliable
source.

Requesting a `date_from` older than the limit fails with:

```
Date selection out of range. Your account has a N-day data retention limit.
Upgrade your plan or acquire a data retention addon.
```

On a 1-day account this means only the last 24 hours are queryable. Ranges like "since the account
was created" will always fail.

### Analytics (`get_analytics_by_*`)

**6 months**, regardless of plan. Exceeding it fails with:

```
Date selection out of range. Analytics has a 6-months data retention limit.
```

This is why per-tag statistics work over a much longer window than raw activity.

## Rate limits

Per minute, per account. Defaults — plans may raise them.

| Endpoint group | Default limit |
|---|---|
| Activity | **10 / minute** |
| General API | 60 / minute |
| Bulk email | 10 / minute |
| Email verification | 60 / minute |

Activity's 10/minute is the one that bites. Paginating through activity to find a single recipient
exhausts it in well under a minute. Narrow with `event` and a tight date window instead of paging.

Every response carries `x-apiquota-remaining` and `x-apiquota-reset`. On a 429, back off until the
reset rather than retrying immediately.

## Date handling

- Unix timestamps, UTC. Datetime strings such as `2015-10-01 00:00:00` are also accepted and
  assumed UTC.
- `date_from` must be strictly lower than `date_to`.
- `date_to` must not be in the future **as evaluated on the server**. A timestamp generated as
  "now" is often already in the past-tense check by milliseconds and gets rejected. Subtract about
  a minute.

## Event types

### Activity events

`queued`, `sent`, `delivered`, `soft_bounced`, `hard_bounced`, `deferred`, `opened`,
`opened_unique`, `clicked`, `clicked_unique`, `unsubscribed`, `spam_complaints`, `survey_opened`,
`survey_submitted`

Notes:

- `deferred` is feature-gated per account. Filtering by it on an account without the feature
  returns a validation error rather than an empty result.
- `junk` exists server-side but is reported as `soft_bounced`.
- `suppressed` exists server-side but is not currently selectable as a filter through MCP.

### Analytics events

`queued`, `sent`, `delivered`, `soft_bounced`, `hard_bounced`, `opened`, `opened_unique`,
`clicked`, `clicked_unique`, `unsubscribed`, `spam_complaints`, `survey_opened`, `survey_submitted`

`deferred` is not available in analytics. `event` is required and must contain at least one value.

## Error to cause to fix

| Error | Cause | Fix |
|---|---|---|
| `The date_to must be before or equal to <timestamp>` | `date_to` is in the future relative to the server clock — usually a timestamp computed as "now" | Set `date_to` about a minute in the past |
| `Date selection out of range. Your account has a N-day data retention limit.` | `date_from` predates the plan's activity retention | Shorten the window to within N days; the error states the actual N |
| `Date selection out of range. Analytics has a 6-months data retention limit.` | Analytics range exceeds 6 months | Shorten to 6 months or less |
| HTTP 429 | Too many calls, usually activity pagination | Back off to `x-apiquota-reset`; narrow by `event` and date instead of paging |
| Validation error naming `domain_id` | `domain_id` missing or a domain name was passed instead of the hashed id | Resolve the id via `list_domains` |
| Validation error on an `event` value | Filtering by a feature-gated event (`deferred`) the account lacks | Drop that event from the filter |
| `date_from` must be lower than `date_to` | Range inverted | Swap the values |

## Quota

`x-apiquota-remaining` and `x-apiquota-reset` describe the **API request** quota — how many calls
remain this period. They do not describe the email sending allowance. There is currently no MCP
tool reporting emails sent this month, the monthly sending limit, or the account's plan.
