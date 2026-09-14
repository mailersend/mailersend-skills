---
name: mailersend-mcp
description: >-
  Use for any task run through the MailerSend MCP server — checking whether email is getting
  delivered, tracing what happened to one email or recipient, measuring how a campaign or template
  performed, sending, or changing suppression lists. Covers what the tool definitions cannot: which
  tool fits which job, how far back data goes on each plan, the rate limits that make activity calls
  fail, how to derive rates from the counts analytics returns, and which actions to confirm first.
  Also applies when a MailerSend call fails on a data retention limit, a rate limit or a rejected
  date range.
---

# MailerSend MCP

The MailerSend MCP server exposes the MailerSend email platform as MCP tools — sending, domains,
templates, activity, analytics, suppressions, webhooks, SMS and DMARC monitoring.

Connect at `https://mcp.mailersend.com/mcp` (OAuth). Tool definitions arrive via `tools/list`, so
this skill does not repeat them. It covers what the schemas cannot tell you: which tool to reach
for, correct sequencing, and the server-side limits that cause most failed calls.

## Operating rules

1. **Find a specific email with `list_emails`, never by paging activity.** It filters server-side by
   recipient address, subject, tag, template and message id. Activity is rate limited to **10
   requests per minute**, so crawling it page by page to locate one address trips the limiter and
   fails.

2. **Check the retention window before choosing a date range.** Activity and `list_emails` are both
   bounded by the account's data retention, 1–30 days depending on plan; analytics keeps 6 months.
   These are different limits — see `references/limits-and-errors.md`.

3. **Analytics returns counts, never rates.** Compute rates yourself and state the denominator you
   used, so the number is not mistaken for an official metric.

4. **Confirm before sending.** `send_email`, `send_bulk_email` and `send_sms` deliver to real
   recipients and consume quota. Show the recipient list, subject and sending domain first.

5. **Treat suppression changes as destructive.** Adding to or removing from the blocklist,
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

**Step 1 — tag the send.**

```
send_email(..., tags: ["welcome-v2"])
```

Use one stable, specific tag per thing you want to measure. `welcome-v2` is measurable;
`transactional` spans everything and tells you nothing. Tags cannot be added after the fact, so an
untagged send can never be reported on by tag.

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

For the individual emails behind a tag rather than the aggregate counts, `list_emails` takes a `tag`
filter over the same period.

## Investigating a specific recipient or failure

`list_emails` filters server-side, so this is one call rather than a pagination crawl.

1. `list_emails` scoped to the domain and a date window inside retention, filtered by
   `recipient_email` (exact, case-insensitive). `subject`, `tag`, `template_id` and `message_id`
   narrow the same way, as do `status` and `interaction`.
2. Read `status` and `suppression_reason` on the row. A non-null `suppression_reason` —
   `on_hold`, `hard_bounced`, `unsubscribed`, `spam_complained` or `blocklisted` — is usually the
   whole answer: a suppressed address is never sent to, which is why no delivery activity exists.
   This is the explanation more often than people expect for "the email never arrived".
3. `get_email` on that row's id for the full event timeline (newest first, capped at 200 events)
   and the message content. Content is null when content tracking is disabled for the domain.

If the filter returns nothing at all, the address was not sent to in that window. Widen the window,
within retention, before concluding the send failed.

## Additional resources

- **`references/limits-and-errors.md`** — retention windows by plan, rate limits, event-type
  gotchas, and an error-to-cause-to-fix table for the failures seen most often.
- **`references/tool-selection.md`** — which tool to reach for, per job.
