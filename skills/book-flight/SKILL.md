---
name: book-flight
description: Books real flights with AI through the FlightClaw MCP server. Searches about 300 airlines for cheap flights, ranks options against saved preferences, opens a hosted checkout and returns the airline booking reference. Use when the user wants to find and book a flight, compare fares, buy a plane ticket or pay for a chosen offer from Claude, ChatGPT, Cursor or any MCP client. FlightClaw is an AI flight booking agent; the price shown already includes the booking fee.
---

# Book Flights with AI (FlightClaw)

Book flights with AI from your assistant. FlightClaw is an AI flight booking agent and MCP server: it searches about 300 airlines for cheap flights, opens a secure checkout and returns the airline booking reference. The traveler pays on a hosted page or with a Link virtual card. The agent never handles card numbers.

## When to use

- The user asks to find, compare or book a flight.
- The user picked an offer and wants to pay for it.
- The user wants a seat map before buying.

## Prerequisites

Connect the FlightClaw hosted MCP server: `https://mcp.flightclaw.com/mcp` (streamable HTTP, OAuth sign-in). Nothing to install.

- **Claude Code**: `claude mcp add --transport http flightclaw https://mcp.flightclaw.com/mcp`
- **Cursor**: [One-click install](https://cursor.com/en/install-mcp?name=flightclaw&config=eyJ1cmwiOiJodHRwczovL21jcC5mbGlnaHRjbGF3LmNvbS9tY3AifQ%3D%3D), or add `{"mcpServers":{"flightclaw":{"url":"https://mcp.flightclaw.com/mcp"}}}` to `~/.cursor/mcp.json`
- **VS Code**: [One-click install](https://vscode.dev/redirect/mcp/install?name=flightclaw&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.flightclaw.com%2Fmcp%22%7D), or `code --add-mcp '{"name":"flightclaw","type":"http","url":"https://mcp.flightclaw.com/mcp"}'`
- **Claude (web / desktop)**: Settings > Connectors > Add custom connector > paste `https://mcp.flightclaw.com/mcp`
- **ChatGPT**: Settings > Apps & Connectors > Create (developer mode) > paste `https://mcp.flightclaw.com/mcp`

If the FlightClaw tools are not available, ask the user to connect the server first.

## Tool flow

1. **Search.** Call `search_flights` with `origin`, `destination` (IATA codes), `date` (YYYY-MM-DD) and optional `return_date`, `cabin`, `adults`, `children`, `infants`, `passenger_ages`, `max_connections`, `limit`.
   - For a ranked shortlist that uses the user's saved preferences, call `recommend_flights` instead (same route fields). It returns the top 3 offers with reasons.
   - For more than one leg, call `search_multi_city` with `slices` (see the plan-group-trip skill).
2. **Check the offer.** Call `get_offer` with the chosen `offer_id`. Confirm price, bags and fare rules. Offers expire in about 30 minutes; search again if one has expired.
3. **Seat map (optional).** Call `get_seat_map` with `offer_id`. This is information only. Checkout does not sell seats.
4. **Passengers.** Call `list_travelers` to find saved traveler names. Save new travelers with `save_traveler` (see the traveler-profile skill).
5. **Confirm with the user.** Show the route, times, airline, fare rules and the total price including the fee. Get an explicit yes.
6. **Create checkout.** Call `create_checkout` with `offer_id` and `passengers` (an array of saved traveler names or passenger objects). It returns `checkout_id`, `checkout_url`, `total_amount` and `total_currency`. Nothing is charged yet.
7. **Pay.** Use one of two ways:
   - **Checkout link**: send `checkout_url` to the traveler. The traveler pays there by card.
   - **Link virtual card**: create a Link spend request for exactly `total_amount` in `total_currency`. The user approves it in Link. Pay on `checkout_url` with the Link virtual card. Never pay more than the approved amount.
8. **Wait for completion.** Call `get_checkout_status` with `checkout_id` until the status is `completed`.
9. **Confirm the booking.** Call `get_order` with the order id to get the booking reference. Optionally call `log_trip` to save the trip in history.

## Price and fee

- `total_amount` from search, `get_offer` and `create_checkout` already includes the FlightClaw booking fee: 5% + $3, minimum $12.
- Always show the total including the fee. Do not add the fee again.
- The price comes from the airline at checkout time, not from the caller.

## Rules

- Never ask for or accept card numbers in chat.
- Always get explicit user confirmation before `create_checkout`.
- Show the exact `total_amount` and `total_currency` before payment.
- With a Link virtual card, the spend request must equal `total_amount`. Never more.
- Do not say a flight is booked until `get_checkout_status` is `completed` and `get_order` returns a booking reference.
- Search is limited to 100 searches per user per UTC day. Avoid repeated searches.

## Example prompts

- "Book me the cheapest nonstop flight from London to New York on 12 March."
- "Find business class flights LHR to SFO next Friday and book the best one for me."
- "Show me the seat map for that BA flight before I pay."
- "Book a return flight to Lisbon for two adults, 4 to 9 June."
- "Pay for this flight with my Link card."

Learn more at [flightclaw.com](https://flightclaw.com).
