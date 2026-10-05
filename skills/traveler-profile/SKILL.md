---
name: traveler-profile
description: Manages the traveler profile in the FlightClaw MCP server so an AI flight booking agent knows who flies and what they like. Saves the primary user, companions, cabin and airline preferences, credit cards, loyalty points balances and past trips with feedback. Use when the user wants to set up travel preferences, save passenger details, record points or cards, review trip history or give feedback after a trip in Claude, ChatGPT, Cursor or any MCP client.
---

# Traveler Profile, Preferences and Trip History (FlightClaw)

Set up a traveler profile once so FlightClaw can book flights with AI that fit the user. The profile holds the user, companions, travel preferences, cards, points balances and trip history. `recommend_flights` uses these preferences to rank cheap flights for the person asking.

## When to use

- First use: onboard the user and their companions.
- The user states a preference ("I never fly red-eyes", "aisle seat").
- The user adds a card or a points balance.
- The user asks about past trips, or gives feedback after a trip.

## Prerequisites

Connect the FlightClaw hosted MCP server: `https://mcp.flightclaw.com/mcp` (streamable HTTP, OAuth sign-in). Nothing to install.

- **Claude Code**: `claude mcp add --transport http flightclaw https://mcp.flightclaw.com/mcp`
- **Cursor**: [One-click install](https://cursor.com/en/install-mcp?name=flightclaw&config=eyJ1cmwiOiJodHRwczovL21jcC5mbGlnaHRjbGF3LmNvbS9tY3AifQ%3D%3D), or add `{"mcpServers":{"flightclaw":{"url":"https://mcp.flightclaw.com/mcp"}}}` to `~/.cursor/mcp.json`
- **VS Code**: [One-click install](https://vscode.dev/redirect/mcp/install?name=flightclaw&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.flightclaw.com%2Fmcp%22%7D), or `code --add-mcp '{"name":"flightclaw","type":"http","url":"https://mcp.flightclaw.com/mcp"}'`
- **Claude (web / desktop)**: Settings > Connectors > Add custom connector > paste `https://mcp.flightclaw.com/mcp`
- **ChatGPT**: Settings > Apps & Connectors > Create (developer mode) > paste `https://mcp.flightclaw.com/mcp`

If the FlightClaw tools are not available, ask the user to connect the server first.

## Tool flow

### Who you are

- `get_me`: show the primary user (linked traveler, account email, home airports).
- `set_me`: set `me_traveler` (a saved traveler name), and optional `account_email` and `home_airports` (array of IATA codes).

### Travelers

- `save_traveler`: `name` (short key), `given_name`, `family_name`; optional `born_on`, `gender`, `title`, `email`, `phone_number`, `relationship`, `loyalty_programmes`.
- `list_travelers`, `get_traveler` (by `name`), `delete_traveler` (by `name`).

### Preferences

- `get_preferences`: show saved preferences and learnings.
- `set_preferences`: patch any subset. Fields that `recommend_flights` uses: `cabin_rules` (short-haul and long-haul cabin), `preferred_airlines`, `avoid_airlines`, `depart_window`, `max_stops`, `redeye_ok`, `budget_sensitivity`. Other preferences such as seat, baggage and meal can be saved too.

### Cards and points

- `save_card`: `id`, `issuer`, `product`; optional `network`, `region`, `notes`. This records which cards the user holds for points awareness. It never stores card numbers.
- `list_cards`, `delete_card` (by `id`).
- `set_points_balance`: `program` and `balance`. `list_points` shows all balances.

### Trips

- `log_trip`: after a booking, record `route`, `depart_date`, `travelers`; optional `return_date`, `cabin`, `price`, `currency`, `order_id`.
- `list_trips`, `get_trip` (by `id`).
- `trips_followup`: list completed trips that need feedback.
- `record_trip_feedback`: `id`, `feedback` and optional `learnings` (array). Lasting lessons fold into preferences.

## Onboarding order

1. `save_traveler` for the user, then `set_me`.
2. `save_traveler` for each companion, and `save_group` if they travel together.
3. `set_preferences` with one or two questions per message.
4. Optional: `save_card` and `set_points_balance`.

## Rules

- Never ask for or save card numbers, CVCs or passwords. `save_card` stores only issuer and product.
- Ask before you save personal data, and save only what the user gives.
- Confirm before `delete_traveler`, `delete_card` or other deletes.
- When the user books, confirm before `create_checkout` and show the price including the fee (5% + $3, minimum $12). See the book-flight skill.

## Example prompts

- "Set me up: I'm Jack Culpan, home airports LHR and LGW."
- "I prefer economy on short flights and business on long-haul, and I avoid Ryanair."
- "I have an Amex Platinum and 120,000 Avios."
- "Show my past trips."
- "The Lisbon trip was great but the layover was too short. Remember that."

Learn more at [flightclaw.com](https://flightclaw.com).
