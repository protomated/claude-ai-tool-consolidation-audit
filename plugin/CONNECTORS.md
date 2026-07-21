# Connectors

This plugin uses one connector that ships with Claude Desktop. It is managed by Anthropic — you do not need to set up credentials or install anything beyond Claude Desktop.

## Connector for this plugin

| Connector | What it does | Setup |
|---|---|---|
| **Google Calendar** | Creates calendar events for computed deadlines. The plugin only creates events when you explicitly confirm after reviewing the full event details. | Connect once via Claude Desktop → Settings → Connectors → Google Calendar → "Connect," then sign in with your Google account |

> **Connector identifier note:** The connector identifier used in this plugin's `.mcp.json` is `google-calendar`. If Claude Desktop's Connectors panel uses a different name, verify the current identifier against the live connector surface and update `.mcp.json` to match.

## How to connect

1. Open Claude Desktop.
2. Go to **Settings → Connectors**.
3. Find **Google Calendar** — click **Connect**.
4. Sign in with the Google account that holds your firm's calendar.
5. Authorize calendar access when the permission prompt appears.
6. Restart Claude Desktop if prompted.

> **Firm Google Workspace:** If your firm uses Google Workspace, sign in with your Workspace account (`yourname@firmname.com`) rather than a personal Gmail. Deadline events will land in your firm calendar and stay under your organization's data retention and access policies.

## What the connector can access

| Connector | Can access | Cannot access |
|---|---|---|
| Google Calendar | Events in the signed-in Google account's calendar | Events in calendars you have not authorized; other Google services |

## Privacy note

The plugin computes deadlines entirely within your Claude Desktop conversation. No matter details, trigger dates, or rule text are transmitted to Protomated or any third party. The Google Calendar connector only creates events when you explicitly confirm, and only within the calendar account you authorized. Data is processed under your Claude plan's data handling terms and Google's terms for the connected account.

## Troubleshooting

**Google Calendar shows "Not connected":**
Go to Settings → Connectors → Google Calendar and click Connect. Complete the Google sign-in flow and grant calendar access when prompted.

**"Permission denied" when creating an event:**
The signed-in Google account may not have write access to the target calendar. Verify that you are signed in with the correct account and that the account has create permissions for the calendar in question.

**Plugin can't find the Google Calendar connector:**
Make sure you are on a qualifying Claude plan (Claude for Work, Team, or Enterprise). Confirm that Google Calendar appears as an available connector in Claude Desktop → Settings → Connectors. If it does not appear, check Anthropic's current connector documentation for availability on your plan.

**Event created with wrong details:**
The plugin shows the full event draft before creating anything. If an event was created with incorrect information, delete it from Google Calendar directly and run `/court-deadline` again, reviewing the draft carefully before confirming.
