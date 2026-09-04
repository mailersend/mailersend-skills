---
name: mailersend-mcp
description: >-
  This skill should be used when the user asks to "check email delivery health", "are my emails
  getting through", "check bounce rate", "check spam complaints", "did my email arrive",
  "find emails sent to someone", "why didn't this email arrive", "check open rate",
  "check delivery rate", "how did this campaign perform", "check template performance",
  "is my domain verified", "is my domain paused", "check sending domain status",
  "audit email activity", "investigate a delivery failure", or any task involving monitoring,
  auditing or reporting on email sending through the MailerSend MCP server. Provides workflows
  for delivery-health monitoring, activity investigation, and per-email or per-template
  statistics via tags.
---

# MailerSend MCP

The MailerSend MCP server exposes the MailerSend email platform as MCP tools — sending, domains,
templates, activity, analytics, suppressions, webhooks, SMS and DMARC monitoring.

Connect at `https://mcp.mailersend.com/mcp` (OAuth). Tool definitions arrive via `tools/list`, so
this skill does not repeat them. It covers what the schemas cannot tell you: which tool to reach
for, correct sequencing, and the server-side limits that cause most failed calls.

## Operating rules

1. **Always pass `domain_id` to activity and analytics calls.** Without it the call fails or
   returns nothing useful. Resolve it once with `list_domains` and reuse it.

2. **Never brute-force pagination to find a recipient.** Activity is rate limited to **10
   requests per minute** by default. Paging through 6 pages to locate one address will trip the
   limiter and fail. Narrow by `event` type and the tightest date window instead.

3. **Set `date_to` in the past, not "now".** The server compares it to its own clock. A timestamp
   computed as "now" is already stale by the time it is validated and gets rejected. Use roughly
   one minute ago.

4. **Check the retention window before choosing a date range.** Activity retention is 1–30 days
   depending on plan; analytics is 6 months. These are different limits — see
   `references/limits-and-errors.md`.

5. **Analytics returns counts, never rates.** Compute rates yourself and state the denominator you
   used, so the number is not mistaken for an official metric.

6. **Confirm before sending.** `send_email`, `send_bulk_email` and `send_sms` deliver to real
   recipients and consume quota. Show the recipient list, subject and sending domain first.

7. **Treat suppression changes as destructive.** Adding to or removing from the blocklist,
   unsubscribes, hard bounces or spam complaints changes who can ever be emailed again. Confirm
   first, and never bulk-remove suppressions to "fix" deliverability.

## Delivery health check

The most common use. A routine pass over sending health.

1. `list_domains` — confirm each sending domain is verified and not paused.
2. `get_domain_verification_status` on any domain that looks wrong.
3. `list_activities` with `event: ["hard_bounced", "soft_bounced", "spam_complaints"]` over the
   watch window, scoped to `domain_id`.
4. Compare against thresholds: bounce rate above **2%** and spam complaints above **0.1%** warrant
   action. Sustained spam complaints above **0.3%** risk blocking at the mailbox provider.

Filter by `event` rather than pulling everything and counting client-side. It is one call instead
of many and keeps you inside the rate limit.

## Per-email and per-template statistics via tags

Open and delivery rates for a *specific* email or template are not available directly — there is no
"stats for template X" tool. Tags are the supported route.

**Step 1 — tag the send.** `send_email` accepts `tags` (max 5, each ≤191 characters). Templates
carry their own tags, and a tag set at send time **overrides** the template's.

```
send_email(..., tags: ["welcome-v2"])
```

Use one stable, specific tag per thing you want to measure. `welcome-v2` is measurable;
`transactional` spans everything and tells you nothing.

**Step 2 — query analytics filtered by that tag.**

```
get_analytics_by_date(
  domain_id: "<id>",
  tags: ["welcome-v2"],
  event: ["sent", "delivered", "opened_unique", "clicked_unique", "hard_bounced", "soft_bounced"],
  date_from: <unix>,
  date_to: <unix>,
  group_by: "days"
)
```

**Step 3 — compute the rates.** The response gives counts per event per period. Derive:

| Rate | Formula |
|---|---|
| Delivery rate | `delivered / sent` |
| Open rate | `opened_unique / delivered` |
| Click rate | `clicked_unique / delivered` |
| Click-to-open rate | `clicked_unique / opened_unique` |
| Bounce rate | `(hard_bounced + soft_bounced) / sent` |
| Spam rate | `spam_complaints / delivered` |

Use `opened_unique` and `clicked_unique` for rates — `opened` and `clicked` count repeat events and
will overstate engagement.

Two caveats to state alongside any open rate: it requires open tracking to be enabled on the
domain, and privacy features such as Apple Mail Privacy Protection inflate it. Delivery and bounce
rates are reliable; open rate is directional.

`get_analytics_by_country`, `get_analytics_by_user_agent_name` and `get_analytics_by_user_agent_type`
accept the same `tags` filter for breakdowns of opens.

## Investigating a specific recipient or failure

There is no server-side recipient filter on activity today, so avoid paging blindly.

1. Narrow the date window as far as the user's knowledge allows — hours, not the whole retention
   window.
2. Filter by `event` to the outcomes that matter (`hard_bounced`, `soft_bounced`, `delivered`).
3. Raise `limit` to 100 so one call covers what four would.
4. `get_activity` on a specific activity id for the full detail, including the reason.
5. If the address is absent entirely, check suppressions — `list_blocklist`, `list_hard_bounces`,
   `list_unsubscribes`, `list_spam_complaints`. A suppressed recipient is never sent to and so
   produces no delivery activity.

Step 5 is the answer more often than people expect for "the email never arrived".

## Key patterns

- **Dates:** Unix timestamps (UTC). `date_from` must be lower than `date_to`, and `date_to` must
  not be in the future.
- **Pagination:** `limit` accepts 10–100 and defaults to 25 server-side. Ask for 100.
- **Domain IDs:** hashed strings from `list_domains`, not domain names.
- **Tags:** max 5 per email, each ≤191 characters. Send-time tags override template tags.
- **Quota:** responses carry `x-apiquota-remaining` and `x-apiquota-reset` headers.

## Additional resources

- **`references/limits-and-errors.md`** — retention windows by plan, rate limits, event types, and
  an error-to-cause-to-fix table for the failures seen most often.
- **`references/tool-selection.md`** — which tool to reach for, per job.
