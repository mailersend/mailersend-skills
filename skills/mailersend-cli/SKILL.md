---
name: MailerSend
description: >-
  This skill should be used when the user asks to "send an email", "send email with mailersend",
  "check email analytics", "manage domains", "verify a domain", "list messages", "check delivery status",
  "manage webhooks", "verify an email address", "manage suppressions", "send SMS", "send bulk email",
  "check API quota", "manage SMTP users", "manage sender identities", "manage templates",
  "manage inbound routes", "manage API tokens", "manage recipients", "launch the dashboard",
  "configure mailersend", "set up mailersend profile", or any task involving the MailerSend email
  sending platform via CLI. Provides comprehensive guidance for using the `mailersend` command-line tool.
---

# MailerSend CLI

The MailerSend CLI (`mailersend`) is a command-line tool and interactive TUI dashboard for the MailerSend email API. It supports sending transactional emails and SMS, managing domains, viewing analytics, handling suppressions, email verification, webhooks, and full account management.

## Installation

Install via Homebrew, Nix, or download a binary from GitHub releases:

```bash
# Homebrew
brew install mailersend/tap/mailersend-cli

# Nix flake
nix profile install github:mailersend/mailersend-cli

# Or download from https://github.com/mailersend/mailersend-cli/releases
```

The binary is called `mailersend`.

## Authentication

Before using the CLI, authenticate with a MailerSend API token:

```bash
# Interactive login (prompts for method and token)
mailersend auth login

# Direct token login
mailersend auth login --method token --token "mlsn.your_token_here"

# Check authentication status
mailersend auth status
```

### Multi-Profile Support

Manage multiple accounts or tokens with named profiles:

```bash
mailersend profile add production --token "mlsn.prod_token"
mailersend profile add staging --token "mlsn.staging_token"
mailersend profile switch production
mailersend profile list
```

Use `--profile <name>` on any command to override the active profile.

**Token resolution order:** `MAILERSEND_API_TOKEN` env var > `--profile` flag > active profile > first profile.

Config is stored at `~/.config/mailersend/config.yaml`.

## Global Flags

These flags work on all commands:

| Flag | Description |
|------|-------------|
| `--profile <name>` | Select config profile |
| `--verbose` / `-v` | Show HTTP request/response details |
| `--json` | Output as machine-readable JSON |

The CLI respects the `NO_COLOR` environment variable.

## Critical: Always Use JSON Output

When executing `mailersend` commands, **always append `--json`** to every command. JSON output is structured and parseable, making it reliable for extracting IDs, statuses, and other data needed for follow-up commands. The default table output is for human reading only and difficult to parse programmatically.

```bash
# Correct - always use --json
mailersend domain list --json
mailersend message get <id> --json

# Incorrect - table output is hard to parse
mailersend domain list
```

The only exceptions are `dashboard` (interactive TUI), `completion` (shell scripts), and write operations like `email send` where the response confirmation is simple enough without JSON.

## Core Workflows

### Sending Email

```bash
# Send with flags
mailersend email send \
  --from "sender@yourdomain.com" \
  --from-name "Sender Name" \
  --to "recipient@example.com" \
  --subject "Hello" \
  --text "Plain text body" \
  --html "<h1>HTML body</h1>"

# Send with HTML file
mailersend email send \
  --from "sender@yourdomain.com" \
  --to "recipient@example.com" \
  --subject "Newsletter" \
  --html-file ./email.html

# Send using a template
mailersend email send \
  --from "sender@yourdomain.com" \
  --to "recipient@example.com" \
  --template-id "template_id_here"

# Schedule for later (unix timestamp)
mailersend email send \
  --from "sender@yourdomain.com" \
  --to "recipient@example.com" \
  --subject "Scheduled" \
  --text "This is scheduled" \
  --send-at 1700000000

# Enable tracking
mailersend email send \
  --from "sender@yourdomain.com" \
  --to "recipient@example.com" \
  --subject "Tracked" \
  --text "Body" \
  --track-clicks --track-opens
```

Omit required flags to enter interactive mode with prompts.

### Domain Management

```bash
mailersend domain list --json                   # List all domains
mailersend domain list --verified --json        # List verified domains only
mailersend domain add --name example.com --json # Add a domain
mailersend domain dns example.com --json        # Show DNS records to configure
mailersend domain verify example.com --json     # Verify domain DNS
mailersend domain get example.com --json        # Get domain details
mailersend domain update-settings example.com \
  --track-clicks --track-opens                  # Update tracking settings
mailersend domain delete example.com            # Delete a domain
```

Domain commands accept either the domain name or domain ID.

### Analytics

```bash
# Date-based analytics (required: --date-from, --date-to, --event)
mailersend analytics date \
  --date-from 2025-01-01 --date-to 2025-01-31 \
  --event sent --event delivered --event opened \
  --group-by days --json

# By country, user agent name, or user agent type
mailersend analytics country --date-from 2025-01-01 --date-to 2025-01-31 --json
mailersend analytics ua-name --date-from 2025-01-01 --date-to 2025-01-31 --json
mailersend analytics ua-type --date-from 2025-01-01 --date-to 2025-01-31 --json
```

Event types: `queued`, `sent`, `delivered`, `soft_bounced`, `hard_bounced`, `opened`, `clicked`, `unsubscribed`, `spam_complaints`.

### Messages & Activity

```bash
mailersend message list --limit 10 --json                     # Recent messages
mailersend message list --status delivered --domain example.com --json
mailersend message get <message_id> --json                    # Message details
mailersend message scheduled list --json                      # Scheduled messages
mailersend message scheduled delete <id>                      # Cancel scheduled

mailersend activity list --domain example.com \
  --date-from 2025-01-01 --date-to 2025-01-31 --json        # Activity log
```

### Email Verification

```bash
mailersend verification verify user@example.com --json       # Verify single email
mailersend verification list create --name "My List" \
  --emails-file ./emails.txt --json                          # Create verification list
mailersend verification list verify <id> --wait --json       # Verify list and wait
mailersend verification list results <id> --json             # Get results
```

### Sending SMS

```bash
mailersend sms send --from "+1234567890" --to "+0987654321" --text "Hello via SMS"
mailersend sms message list --json                           # List SMS messages
mailersend sms number list --json                            # List phone numbers
```

### Bulk Email

```bash
# Prepare a JSON file with an array of email objects, then:
mailersend bulk-email send --file ./emails.json
mailersend bulk-email status <bulk_email_id> --json
```

### TUI Dashboard

```bash
mailersend dashboard
```

Launch an interactive terminal dashboard with sidebar navigation, real-time data, and vim-style keybindings (`j`/`k` to navigate, `Enter` to select, `?` for help, `q` to quit).

## Key Patterns

- **Domain resolution:** Commands accept domain name or domain ID interchangeably.
- **Date inputs:** Accept `YYYY-MM-DD` format or unix timestamps.
- **Pagination:** Use `--limit <n>` to control results. `--limit 0` fetches all results.
- **Interactive mode:** Omit required flags and the CLI prompts interactively.
- **JSON output:** Append `--json` to any command for machine-readable output.
- **Repeatable flags:** Flags like `--event`, `--to`, `--tags` can be specified multiple times.

## Additional Resources

### Reference Files

For the complete command reference with all subcommands and flags, consult:
- **`references/command-reference.md`** - Full command tree for all 20+ command groups with every flag documented
