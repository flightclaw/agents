---
name: track-flight-price
description: Tracks flight prices with the FlightClaw MCP server and emails an alert when a fare drops. Works as an AI flight price tracker for cheap flights on any route and date, checked once a day. Use when the user wants to watch a fare, set a target price, get a price drop alert, list tracked flights or stop tracking a route from Claude, ChatGPT, Cursor or another MCP client.
---

# Flight Price Tracker with Email Alerts (FlightClaw)

Use FlightClaw as a flight price tracker from your AI assistant. FlightClaw checks each tracked route once a day and emails the account email when the price drops, so the user can catch cheap flights and then book with AI.

## When to use

- The user wants to watch a route and date for a lower fare.
- The user sets a target price and wants an alert.
- The user asks which flights they track, or wants to stop tracking one.

## Prerequisites

Connect the FlightClaw hosted MCP server: `https://mcp.flightclaw.com/mcp` (streamable HTTP, OAuth sign-in). Nothing to install.

- **Claude Code**: `claude mcp add --transport http flightclaw https://mcp.flightclaw.com/mcp`
- **Cursor**: [One-click install](https://cursor.com/en/install-mcp?name=flightclaw&config=eyJ1cmwiOiJodHRwczovL21jcC5mbGlnaHRjbGF3LmNvbS9tY3AifQ%3D%3D), or add `{"mcpServers":{"flightclaw":{"url":"https://mcp.flightclaw.com/mcp"}}}` to `~/.cursor/mcp.json`
- **VS Code**: [One-click install](https://vscode.dev/redirect/mcp/install?name=flightclaw&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.flightclaw.com%2Fmcp%22%7D), or `code --add-mcp '{"name":"flightclaw","type":"http","url":"https://mcp.flightclaw.com/mcp"}'`
- **Claude (web / desktop)**: Settings > Connectors > Add custom connector > paste `https://mcp.flightclaw.com/mcp`
- **ChatGPT**: Settings > Apps & Connectors > Create (developer mode) > paste `https://mcp.flightclaw.com/mcp`

If the FlightClaw tools are not available, ask the user to connect the server first.

## Tool flow

1. **Track.** Call `track_flight` with `origin`, `destination` (IATA), `date` (YYYY-MM-DD) and `target_price`. Optional: `return_date`, `cabin` (ECONOMY, PREMIUM_ECONOMY, BUSINESS, FIRST), `adults`, `currency` (3-letter code; defaults to the route's offer currency).
2. **List.** Call `list_tracked` to show tracked flights with the last and lowest prices seen. Each item has an `id`.
3. **Stop.** Call `remove_tracked` with the `id` from `list_tracked`.
4. **Book on a drop.** When the user wants to buy, follow the book-flight skill.

## How alerts work

- FlightClaw checks each tracked flight once a day.
- It emails the account email when the cheapest bookable total (including the booking fee) is at or below `target_price`, or drops 10% or more since the last check.
- Alerts compare prices in one currency only.
- Tracking stops after the departure date.
- Each user can have at most 5 active tracked flights.

## Rules

- Ask for a target price if the user does not give one. Suggest the current cheapest fare as a starting point.
- Tell the user that alerts go to their account email. Use `get_me` to show that email if needed.
- If the user already tracks 5 flights, show `list_tracked` and ask which one to remove.
- Prices include the booking fee (5% + $3, minimum $12).

## Example prompts

- "Track LHR to JFK on 1 July and email me if it drops below $450."
- "Watch business class fares from Sydney to Tokyo in November."
- "Which flights am I tracking?"
- "Stop tracking the Lisbon flight."

Learn more at [flightclaw.com](https://flightclaw.com).
