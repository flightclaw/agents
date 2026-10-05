---
name: manage-booking
description: Manages flight bookings made with the FlightClaw MCP server. Lists orders, shows booking references and status, quotes flight change options and cancels with a refund quote first. Use when the user asks about an existing flight booking, wants to change a date, cancel a flight, check a refund or find a booking reference from Claude, ChatGPT, Cursor or another MCP client after they book flights with AI.
---

# Manage Flight Bookings with AI (FlightClaw)

Manage flight bookings made with the FlightClaw AI flight booking agent. Find orders, read booking references, get change quotes and cancel with a refund quote, all from your AI assistant.

## When to use

- The user asks about a flight they booked with FlightClaw.
- The user needs a booking reference or the order status.
- The user wants to change a date or cancel a flight.

## Prerequisites

Connect the FlightClaw hosted MCP server: `https://mcp.flightclaw.com/mcp` (streamable HTTP, OAuth sign-in). Nothing to install.

- **Claude Code**: `claude mcp add --transport http flightclaw https://mcp.flightclaw.com/mcp`
- **Cursor**: [One-click install](https://cursor.com/en/install-mcp?name=flightclaw&config=eyJ1cmwiOiJodHRwczovL21jcC5mbGlnaHRjbGF3LmNvbS9tY3AifQ%3D%3D), or add `{"mcpServers":{"flightclaw":{"url":"https://mcp.flightclaw.com/mcp"}}}` to `~/.cursor/mcp.json`
- **VS Code**: [One-click install](https://vscode.dev/redirect/mcp/install?name=flightclaw&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.flightclaw.com%2Fmcp%22%7D), or `code --add-mcp '{"name":"flightclaw","type":"http","url":"https://mcp.flightclaw.com/mcp"}'`
- **Claude (web / desktop)**: Settings > Connectors > Add custom connector > paste `https://mcp.flightclaw.com/mcp`
- **ChatGPT**: Settings > Apps & Connectors > Create (developer mode) > paste `https://mcp.flightclaw.com/mcp`

If the FlightClaw tools are not available, ask the user to connect the server first.

## Tool flow

1. **Find the order.** Call `list_orders` to list orders booked from this account.
2. **Order details.** Call `get_order` with `order_id`. It returns status, booking reference, passengers and slices.
3. **Change quote.** Call `request_change` with `order_id`, `new_date` (YYYY-MM-DD) and optional `slice_index` (0 = outbound, 1 = return), `new_origin`, `new_destination`, `cabin`. It returns change options with prices.
   - This is a quote only. FlightClaw cannot confirm changes yet. To change, the user can cancel for a refund and book again, or contact the airline with the booking reference.
4. **Cancel.** This is a two-step flow:
   1. Call `cancel_order` with `order_id` only. It returns `cancellation_id`, `refund_amount` and `refund_currency`.
   2. Show the refund to the user and get an explicit yes. Then call `cancel_order` again with `order_id`, `cancellation_id` and `confirm: true`.
   - FlightClaw refunds the traveler manually after the cancellation.

## Rules

- Always get explicit user confirmation before `request_change` and before each `cancel_order` call. Cancellation cannot be undone.
- Always show the refund amount before the confirming `cancel_order` call.
- Never call `cancel_order` with `confirm: true` without a fresh `cancellation_id` from the first call.
- Show change prices exactly as returned. Explain that changes cannot be confirmed through FlightClaw.
- Change quotes are limited per user per day. Avoid repeated calls.

## Example prompts

- "What is the booking reference for my Paris flight?"
- "Show my upcoming flight bookings."
- "How much would it cost to move my return flight to 20 June?"
- "Cancel my flight to Dublin and tell me the refund first."

Learn more at [flightclaw.com](https://flightclaw.com).
