# MailerSend MCP — which tool for which job

Full tool definitions arrive via `tools/list`. This is a routing table for choosing among them,
covering the jobs that come up most and the ones where the obvious choice is wrong.

## Monitoring and diagnosis

| Job | Tool | Notes |
|---|---|---|
| Is my domain verified / paused? | `list_domains` | Covers all domains in one call; check before deeper digging |
| Why is one domain failing verification? | `get_domain_verification_status` | Per-domain detail |
| What DNS records do I still need? | `get_dns_records` | Returns the records to publish |
| Bounce or spam spike check | `list_activities` + `event` filter | Filter server-side; do not pull everything and count |
| Find emails to one recipient, subject, tag or template | `list_emails` | Server-side filters — one call, never a pagination crawl through activity |
| Everything about one email | `get_email` | Content, recipient and the event timeline, from an id returned by `list_emails` |
| What happened at one point in the event stream? | `get_activity` | Needs an activity id from `list_activities`; for a whole email prefer `get_email` |
| Rates for a campaign or template | `get_analytics_by_date` + `tags` | Returns counts; compute rates yourself |
| Where were opens, or on what client? | `get_analytics_by_country`, `get_analytics_by_user_agent_name`, `get_analytics_by_user_agent_type` | All accept `tags` |
| Status of a bulk send | `get_bulk_email_status` | Includes per-email validation errors |

`list_activities` is the event stream. Reach for it to count outcomes across a domain over a window;
reach for `list_emails` whenever the question is about particular emails.

## Recipients and suppressions

| Job | Tool | Notes |
|---|---|---|
| Was this email suppressed, and why? | `list_emails` | `suppression_reason` on the row answers it without a separate lookup |
| Browse a suppression list | `list_blocklist`, `list_hard_bounces`, `list_unsubscribes`, `list_spam_complaints` | For auditing the lists themselves |
| Stop emailing an address | `add_to_blocklist` | Destructive — confirm first |
| Recipients on a domain | `get_domain_recipients` | Scoped to one domain |
| Detail for one recipient | `get_recipient` | |

A suppressed address is never sent to, so it generates no delivery activity at all. Absence of
activity is evidence to check suppression, not evidence the send failed.

## Templates

| Job | Tool | Notes |
|---|---|---|
| What templates exist? | `list_templates` | Metadata only |
| Template detail | `get_template` | Metadata, variables and stats |
| Create or update a template | `create_template`, `update_template` | |

`get_template` does not return the rendered body of a drag-and-drop or rich-text template. If a user
asks to read or export the content of one, say so plainly rather than trying other tools.

## Sending

| Job | Tool | Notes |
|---|---|---|
| Send one email | `send_email` | Real delivery, consumes quota — confirm first |
| Send many | `send_bulk_email` | Then poll `get_bulk_email_status` |
| Scheduled sends | `list_scheduled_messages`, `get_scheduled_message`, `delete_scheduled_message` | |
| Send SMS | `send_sms` | Separate quota and numbers |

Always tag sends you intend to measure later — see the tags workflow in `SKILL.md`.

## Account and access

| Job | Tool | Notes |
|---|---|---|
| Am I authenticated, and as whom? | `get_auth_status` | |
| API tokens | `list_tokens`, `create_token`, `delete_token` | `create_token` returns the secret once — never echo it back |
| SMTP users | `list_smtp_users`, `create_smtp_user` | Credential-producing; do not print passwords |
| Team members | `list_users`, `invite_user` | |

## Known gaps

Requests that have no MCP tool today. Say so directly rather than substituting something close.

- Reading or exporting the body of a drag-and-drop or rich-text template.
- Emails sent today or this month, the monthly sending limit, or the account's current plan.
  `x-apiquota-remaining` covers daily API requests, not email volume.
- The account's own data retention window. The error message on a rejected date range states it;
  nothing reports it up front.
