# MailerSend CLI Complete Command Reference

## Global Flags

| Flag | Description |
|------|-------------|
| `--profile <name>` | Select config profile |
| `--verbose` / `-v` | Show HTTP request/response details |
| `--json` | Machine-readable JSON output |

---

## auth - Authentication

### `auth login`

Authenticate via API token or OAuth browser flow.

| Flag | Description |
|------|-------------|
| `--method <token\|oauth>` | Authentication method |
| `--token <token>` | API token (for token method) |
| `--profile <name>` | Profile name to save credentials to |

### `auth logout`

Log out and remove stored credentials.

### `auth status`

Show current authentication status.

---

## profile - Profile Management

### `profile add <name>`

Add a new profile.

| Flag | Description |
|------|-------------|
| `--token <token>` | API token for this profile |

### `profile list`

List all profiles.

### `profile switch <name>`

Switch active profile.

### `profile remove <name>`

Remove a profile.

---

## email - Email

### `email send`

Send an email via the MailerSend API.

| Flag | Required | Description |
|------|----------|-------------|
| `--from <email>` | | Sender email address |
| `--from-name <name>` | | Sender name |
| `--to <email>` | Yes | Recipient email (repeatable) |
| `--to-name <name>` | | Recipient name |
| `--cc <email>` | | CC email address |
| `--bcc <email>` | | BCC email address |
| `--reply-to <email>` | | Reply-to email |
| `--subject <text>` | | Email subject |
| `--text <text>` | | Plain text body |
| `--html <text>` | | HTML body |
| `--html-file <path>` | | Path to HTML file |
| `--text-file <path>` | | Path to plain text file |
| `--template-id <id>` | | Template ID to use |
| `--tags <tag>` | | Email tags (repeatable) |
| `--send-at <timestamp>` | | Unix timestamp for scheduled sending |
| `--track-clicks` | | Enable click tracking |
| `--track-opens` | | Enable open tracking |
| `--track-content` | | Enable content tracking |

---

## domain - Domain Management

### `domain list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max domains to return (0 = all) |
| `--verified` | Filter by verified status |

### `domain get <domain_id_or_name>`

Get domain details.

### `domain add`

| Flag | Required | Description |
|------|----------|-------------|
| `--name <name>` | Yes | Domain name |
| `--return-path-subdomain <name>` | | Custom return path subdomain |
| `--custom-tracking-subdomain <name>` | | Custom tracking subdomain |

### `domain delete <domain_id_or_name>`

Delete a domain.

### `domain update-settings <domain_id_or_name>`

| Flag | Description |
|------|-------------|
| `--send-paused` | Pause sending |
| `--track-clicks` | Enable click tracking |
| `--track-opens` | Enable open tracking |
| `--track-unsubscribe` | Enable unsubscribe tracking |
| `--track-content` | Enable content tracking |
| `--custom-tracking-enabled` | Enable custom tracking |
| `--custom-tracking-subdomain <name>` | Custom tracking subdomain |
| `--precedence-bulk` | Set precedence bulk header |
| `--ignore-duplicated-recipients` | Ignore duplicated recipients |

### `domain dns <domain_id_or_name>`

Show DNS records for a domain.

### `domain verify <domain_id_or_name>`

Verify a domain's DNS configuration.

---

## message - Messages

### `message list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max results (default: 25) |
| `--status <status>` | Filter: queued, sent, delivered, failed |
| `--domain <domain>` | Filter by domain name or ID |
| `--date-from <date>` | Start date (YYYY-MM-DD or unix) |
| `--date-to <date>` | End date (YYYY-MM-DD or unix) |

### `message get <message_id>`

Get message details.

### `message scheduled list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max results (default: 25) |
| `--status <status>` | Filter: scheduled, sending, sent, error |
| `--domain <domain>` | Filter by domain name or ID |

### `message scheduled get <message_id>`

Get scheduled message details.

### `message scheduled delete <message_id>`

Delete a scheduled message.

---

## template - Templates

### `template list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max templates to return |
| `--domain <domain>` | Filter by domain name or ID |

### `template get <template_id>`

Get template details including stats.

### `template delete <template_id>`

Delete a template.

---

## analytics - Email Analytics

### `analytics date`

Analytics grouped by date.

| Flag | Required | Description |
|------|----------|-------------|
| `--date-from <date>` | Yes | Start date (YYYY-MM-DD or unix) |
| `--date-to <date>` | Yes | End date (YYYY-MM-DD or unix) |
| `--event <type>` | Yes (min 1) | Event type (repeatable) |
| `--domain <domain>` | | Filter by domain |
| `--group-by <period>` | | Group by: days, weeks, months, years |
| `--tags <tag>` | | Filter by tags (repeatable) |

Event types: `queued`, `sent`, `delivered`, `soft_bounced`, `hard_bounced`, `opened`, `clicked`, `unsubscribed`, `spam_complaints`

### `analytics country`

Analytics grouped by country.

| Flag | Required | Description |
|------|----------|-------------|
| `--date-from <date>` | Yes | Start date |
| `--date-to <date>` | Yes | End date |
| `--domain <domain>` | | Filter by domain |
| `--tags <tag>` | | Filter by tags (repeatable) |

### `analytics ua-name`

Analytics grouped by user agent name. Same flags as `analytics country`.

### `analytics ua-type`

Analytics grouped by user agent type. Same flags as `analytics country`.

---

## activity - Activity Log

### `activity list`

| Flag | Required | Description |
|------|----------|-------------|
| `--domain <domain>` | Yes | Domain name or ID |
| `--limit <n>` | | Max results |
| `--date-from <date>` | Yes | Start date |
| `--date-to <date>` | Yes | End date |
| `--event <type>` | | Filter by event type (repeatable) |

### `activity get <activity_id>`

Get activity details.

---

## webhook - Webhooks

### `webhook list`

| Flag | Required | Description |
|------|----------|-------------|
| `--domain <domain>` | Yes | Domain name or ID |
| `--limit <n>` | | Max webhooks |

### `webhook get <webhook_id>`

Get webhook details.

### `webhook create`

| Flag | Required | Description |
|------|----------|-------------|
| `--name <name>` | Yes | Webhook name |
| `--url <url>` | Yes | Webhook URL |
| `--domain <domain>` | Yes | Domain name or ID |
| `--events <event>` | Yes | Events (repeatable) |
| `--enabled` | | Enabled (default: true) |

Valid events: `activity.sent`, `activity.delivered`, `activity.soft_bounced`, `activity.hard_bounced`, `activity.opened`, `activity.opened_unique`, `activity.clicked`, `activity.clicked_unique`, `activity.unsubscribed`, `activity.spam_complaint`, `activity.survey_opened`, `activity.survey_submitted`, `maintenance.start`, `maintenance.end`, `email_single.verified`, `email_list.verified`, `bulk_email.completed`

### `webhook update <webhook_id>`

| Flag | Description |
|------|-------------|
| `--name <name>` | Webhook name |
| `--url <url>` | Webhook URL |
| `--events <event>` | Events (repeatable) |
| `--enabled` | Enabled |

### `webhook delete <webhook_id>`

Delete a webhook.

---

## verification - Email Verification

### `verification verify <email>`

Verify a single email address synchronously.

### `verification verify-async <email>`

Verify a single email address asynchronously.

### `verification status <id>`

Get async verification status.

### `verification list list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max lists |

### `verification list get <id>`

Get verification list details.

### `verification list create`

| Flag | Required | Description |
|------|----------|-------------|
| `--name <name>` | Yes | List name |
| `--emails <email>` | | Comma-separated emails (repeatable) |
| `--emails-file <path>` | | File with one email per line |

### `verification list verify <id>`

| Flag | Description |
|------|-------------|
| `--wait` | Poll until verification completes |

### `verification list results <id>`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max results |
| `--status <status>` | Filter: valid, invalid, catch_all, mailbox_full, role, unknown |

---

## suppression - Suppressions

All suppression subgroups (`blocklist`, `hard-bounces`, `spam-complaints`, `unsubscribes`, `on-hold`) follow the same pattern:

### `suppression <type> list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max results |
| `--domain <domain>` | Filter by domain |

### `suppression <type> add`

| Flag | Required | Description |
|------|----------|-------------|
| `--domain <domain>` | Yes | Domain name or ID |
| `--recipients <email>` | | Email addresses (repeatable) |
| `--patterns <pattern>` | | Patterns (repeatable, blocklist only) |

### `suppression <type> delete`

| Flag | Description |
|------|-------------|
| `--ids <id>` | IDs to delete (repeatable) |
| `--all` | Delete all |
| `--domain <domain>` | Filter by domain |

Types: `blocklist`, `hard-bounces`, `spam-complaints`, `unsubscribes`

### `suppression on-hold list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max results |
| `--domain <domain>` | Filter by domain |

### `suppression on-hold delete`

| Flag | Description |
|------|-------------|
| `--ids <id>` | IDs to delete (repeatable) |
| `--all` | Delete all |

---

## recipient - Recipients

### `recipient list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max recipients (0 = all) |
| `--domain <domain>` | Filter by domain |

### `recipient get <recipient_id>`

Get recipient details.

### `recipient delete <recipient_id>`

Delete a recipient.

---

## inbound - Inbound Routes

### `inbound list`

| Flag | Required | Description |
|------|----------|-------------|
| `--domain <domain>` | Yes | Domain name or ID |
| `--limit <n>` | | Max routes (0 = all) |

### `inbound get <id>`

Get inbound route details.

### `inbound create`

| Flag | Required | Description |
|------|----------|-------------|
| `--domain <domain>` | Yes | Domain name or ID |
| `--name <name>` | Yes | Route name |
| `--domain-enabled` | | Domain enabled (default: true) |
| `--inbound-domain <domain>` | | Inbound domain |
| `--inbound-priority <n>` | | Priority (default: 100) |
| `--catch-type <type>` | | catch_recipient, catch_all |
| `--catch-filter-type <type>` | | Catch filter type |
| `--match-filter-type <type>` | Yes | match_all, match_recipient |
| `--forwards <value>` | Yes | Forward URLs as "type:value" (repeatable) |

### `inbound update <id>`

Same flags as create, all optional.

### `inbound delete <id>`

Delete an inbound route.

---

## identity - Sender Identities

### `identity list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max identities (0 = all) |
| `--domain <domain>` | Filter by domain |

### `identity get <id_or_email>`

Get sender identity details. Accepts ID or email.

### `identity create`

| Flag | Required | Description |
|------|----------|-------------|
| `--domain <domain>` | Yes | Domain name or ID |
| `--name <name>` | Yes | Sender name |
| `--email <email>` | Yes | Sender email |
| `--reply-to-email <email>` | | Reply-to email |
| `--reply-to-name <name>` | | Reply-to name |
| `--add-note` | | Add personal note |
| `--personal-note <text>` | | Personal note text |

### `identity update <id_or_email>`

| Flag | Description |
|------|-------------|
| `--name <name>` | Sender name |
| `--reply-to-email <email>` | Reply-to email |
| `--reply-to-name <name>` | Reply-to name |
| `--add-note` | Add personal note |
| `--personal-note <text>` | Personal note text |

### `identity delete <id_or_email>`

Delete a sender identity.

---

## token - API Tokens

### `token list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max tokens (0 = all) |

### `token get <id>`

Get token details.

### `token create`

| Flag | Required | Description |
|------|----------|-------------|
| `--name <name>` | Yes | Token name |
| `--domain <domain>` | Yes | Domain name or ID |
| `--scopes <scope>` | Yes | Scopes (repeatable) |

### `token update <id>`

| Flag | Description |
|------|-------------|
| `--name <name>` | Token name |

### `token update-status <id>`

| Flag | Required | Description |
|------|----------|-------------|
| `--status <pause\|unpause>` | Yes | Token status |

### `token delete <id>`

Delete an API token.

---

## user - Account Users

### `user list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max users (0 = all) |

### `user get <id>`

Get user details.

### `user invite create`

| Flag | Required | Description |
|------|----------|-------------|
| `--email <email>` | Yes | Email address |
| `--role <role>` | Yes | User role |
| `--permissions <perm>` | | Permissions (repeatable) |
| `--templates <id>` | | Template IDs (repeatable) |
| `--domains <id>` | | Domain IDs (repeatable) |

### `user invite list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max invites (0 = all) |

### `user invite get <id>`

Get invite details.

### `user invite resend <id>`

Resend an invite.

### `user invite cancel <id>`

Cancel an invite.

### `user update <id>`

| Flag | Description |
|------|-------------|
| `--role <role>` | User role |
| `--permissions <perm>` | Permissions (repeatable) |
| `--templates <id>` | Template IDs (repeatable) |
| `--domains <id>` | Domain IDs (repeatable) |

### `user delete <id>`

Delete a user.

---

## smtp - SMTP Users

### `smtp list`

| Flag | Required | Description |
|------|----------|-------------|
| `--domain <domain>` | Yes | Domain name or ID |
| `--limit <n>` | | Max SMTP users (0 = all) |

### `smtp get <id>`

| Flag | Required | Description |
|------|----------|-------------|
| `--domain <domain>` | Yes | Domain name or ID |

### `smtp create`

| Flag | Required | Description |
|------|----------|-------------|
| `--domain <domain>` | Yes | Domain name or ID |
| `--name <name>` | Yes | SMTP user name |
| `--enabled` | | Enabled (default: true) |

### `smtp update <id>`

| Flag | Required | Description |
|------|----------|-------------|
| `--domain <domain>` | Yes | Domain name or ID |
| `--name <name>` | | SMTP user name |
| `--enabled` | | Enabled |

### `smtp delete <id>`

| Flag | Required | Description |
|------|----------|-------------|
| `--domain <domain>` | Yes | Domain name or ID |

---

## sms - SMS

### `sms send`

| Flag | Required | Description |
|------|----------|-------------|
| `--from <number>` | Yes | Sender phone number |
| `--to <number>` | Yes | Recipient phone number (repeatable) |
| `--text <text>` | Yes | Message text |

### `sms message list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max messages (0 = all) |

### `sms message get <id>`

Get SMS message details.

### `sms number list`

| Flag | Description |
|------|-------------|
| `--limit <n>` | Max numbers (0 = all) |
| `--paused` | Filter by paused status |

### `sms number get <id>`

Get SMS number details.

### `sms number update <id>`

| Flag | Description |
|------|-------------|
| `--paused` | Paused status |

### `sms number delete <id>`

Delete an SMS number.

### `sms activity list`

List SMS activity.

### `sms recipient list`

List SMS recipients.

---

## bulk-email - Bulk Email

### `bulk-email send`

| Flag | Required | Description |
|------|----------|-------------|
| `--file <path>` | Yes | Path to JSON file with email array |

### `bulk-email status <bulk_email_id>`

Get bulk email status.

---

## quota - API Quota

### `quota`

View current API quota usage. No flags.

---

## dashboard - TUI Dashboard

### `dashboard`

Launch interactive terminal dashboard.

**Keybindings:**
- `j`/`k` - Navigate up/down
- `Enter` - Select/view details
- `?` - Show help overlay
- `q` - Quit

**Views:** Domains, Activity, Analytics, Messages, Suppressions

---

## version

Print CLI version, commit hash, and build date.

---

## completion <shell>

Generate shell completion scripts.

Valid shells: `bash`, `zsh`, `fish`, `powershell`

```bash
# Example: add to shell config
mailersend completion fish > ~/.config/fish/completions/mailersend.fish
mailersend completion zsh > ~/.zsh/completions/_mailersend
```
