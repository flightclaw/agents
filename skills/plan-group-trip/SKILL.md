---
name: plan-group-trip
description: Plans and books group trips and multi-city flights with the FlightClaw MCP server. Saves each traveler once, groups them (family, friends, colleagues), searches multi-city and open-jaw itineraries for cheap flights and books everyone on one checkout. Use when the user wants to book flights with AI for several people, plan a family or team trip, or search a route with two to six legs from Claude, ChatGPT, Cursor or any MCP client.
---

# Plan Group Trips and Multi-City Flights with AI (FlightClaw)

Plan a group trip and book flights with AI for everyone in one flow. FlightClaw saves travelers and named groups, searches multi-city itineraries across about 300 airlines and books all passengers on one checkout.

## When to use

- The user books for more than one person (family, friends, team).
- The user wants to reuse a saved group, for example "the family".
- The trip has two to six legs, or flies into one city and out of another.

## Prerequisites

Connect the FlightClaw hosted MCP server: `https://mcp.flightclaw.com/mcp` (streamable HTTP, OAuth sign-in). Nothing to install.

- **Claude Code**: `claude mcp add --transport http flightclaw https://mcp.flightclaw.com/mcp`
- **Cursor**: [One-click install](https://cursor.com/en/install-mcp?name=flightclaw&config=eyJ1cmwiOiJodHRwczovL21jcC5mbGlnaHRjbGF3LmNvbS9tY3AifQ%3D%3D), or add `{"mcpServers":{"flightclaw":{"url":"https://mcp.flightclaw.com/mcp"}}}` to `~/.cursor/mcp.json`
- **VS Code**: [One-click install](https://vscode.dev/redirect/mcp/install?name=flightclaw&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.flightclaw.com%2Fmcp%22%7D), or `code --add-mcp '{"name":"flightclaw","type":"http","url":"https://mcp.flightclaw.com/mcp"}'`
- **Claude (web / desktop)**: Settings > Connectors > Add custom connector > paste `https://mcp.flightclaw.com/mcp`
- **ChatGPT**: Settings > Apps & Connectors > Create (developer mode) > paste `https://mcp.flightclaw.com/mcp`

If the FlightClaw tools are not available, ask the user to connect the server first.

## Tool flow

1. **Check travelers.** Call `list_travelers`. For each missing person, call `save_traveler` with `name` (short key, for example `jack`), `given_name`, `family_name` and, when known, `born_on`, `gender`, `title`, `email`, `phone_number`, `relationship`, `loyalty_programmes`.
2. **Save the group.** Call `save_group` with `name` and `members` (an array of saved traveler names). Use `list_groups` or `get_group` (by `name`) to reuse one later.
3. **Search.**
   - One route: `search_flights` or `recommend_flights` with `adults`, `children`, `infants` and `passenger_ages` (age of each traveler under 18) to match the group.
   - Several legs: `search_multi_city` with `slices` (2 to 6 items of `origin`, `destination`, `date`, in travel order) plus the same passenger fields, `cabin`, `max_connections` and `limit`.
4. **Check the offer.** Call `get_offer` with the chosen `offer_id`.
5. **Confirm and book.** Show the total for all passengers including the fee and get an explicit yes. Then call `create_checkout` with `offer_id` and `passengers` set to the group members' names. Continue with payment and `get_checkout_status` as in the book-flight skill.
6. **Log.** Optionally call `log_trip` with `route`, `depart_date` and `travelers`.

## Rules

- The passenger counts in the search must match the people in `create_checkout`.
- Always get explicit user confirmation before `create_checkout`.
- Show the total price for the whole group including the booking fee (5% + $3, minimum $12, already in `total_amount`).
- Never ask for card numbers. Each traveler's data goes in `save_traveler`, not in chat history.
- Offers expire in about 30 minutes. Search again if one has expired.

## Example prompts

- "Save my wife Anna and our two kids, ages 8 and 5, as 'family'."
- "Find flights for the family from Manchester to Malaga on 2 August, back on 16 August."
- "Plan a multi-city trip: London to Rome on 3 May, Rome to Athens on 7 May, Athens to London on 11 May."
- "Book the team offsite flights for Sam, Priya and me to Berlin next Tuesday."

Learn more at [flightclaw.com](https://flightclaw.com).
