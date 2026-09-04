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
| What happened to one specific email? | `get_activity` | Needs an activity id from `list_activities` |
| Rates for a campaign or template | `get_analytics_by_date` + `tags` | Returns counts; compute rates yourself |
| Where were opens, or on what client? | `get_analytics_by_country`, `get_analytics_by_user_agent_name`, `get_analytics_by_user_agent_type` | All accept `tags` |
| Status of a bulk send | `get_bulk_email_status` | Includes per-email validation errors |

## Recipients and suppressions

| Job | Tool | Notes |
|---|---|---|
| Has this address been suppressed? | `list_blocklist`, `list_hard_bounces`, `list_unsubscribes`, `list_spam_complaints` | Check these when an email "never arrived" but no activity exists |
| Stop emailing an address | `add_to_blocklist` | Destructive — confirm first |
| Recipients on a domain | `get_domain_recipients` | Scoped to one domain |
| Detail for one recipient | `get_recipient` | |

A suppressed address is never sent to, so it generates no delivery activity at all. Absence of
activity is evidence to check suppressions, not evidence the send failed.

## Templates

| Job | Tool | Notes |
|---|---|---|
| What templates exist? | `list_templates` | Metadata only |
| Template detail | `get_template` | Returns metadata, variables and stats — **not** the HTML body |
| Create or update a template | `create_template`, `update_template` | `update_template` requires `html` **and** `text` on every call, even when changing only the name |

Reading or exporting the rendered HTML of a drag-and-drop template is not available through MCP.
`get_template` returns the variables and metadata. If a user asks to read or edit template content,
say so plainly rather than trying other tools.

## Sending

| Job | Tool | Notes |
|---|---|---|
| Send one email | `send_email` | Real delivery, consumes quota — confirm first |
| Send many | `send_bulk_email` | Then poll `get_bulk_email_status` |
| Scheduled sends | `list_scheduled_messages`, `get_scheduled_message`, `delete_scheduled_message` | |
| Send SMS | `send_sms` | Separate quota and numbers |

Always tag sends you intend to measure later — see the tags workflow in `SKILL.md`. Tags cannot be
added retrospectively, so an untagged send can never be reported on by tag.

## Account and access

| Job | Tool | Notes |
|---|---|---|
| Am I authenticated, and as whom? | `get_auth_status` | |
| API tokens | `list_tokens`, `create_token`, `delete_token` | `create_token` returns the secret once — never echo it back |
| SMTP users | `list_smtp_users`, `create_smtp_user` | Credential-producing; do not print passwords |
| Team members | `list_users`, `invite_user` | |

## Known gaps

Requests that have no MCP tool today. Say so directly rather than substituting something close.

- Reading or exporting a template's HTML body.
- Emails sent today or this month, the monthly sending limit, or the account's current plan.
  `x-apiquota-remaining` covers API requests, not email volume.
- Filtering activity server-side by recipient address or message id.
