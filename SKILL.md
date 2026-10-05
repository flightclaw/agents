---
name: flightclaw
description: Personal travel-booking agent. Onboard a traveler once (who you are, companions, loyalty programs, credit cards, travel preferences), then chat about where you want to go and get a few preference-ranked flight options, book and pay for them, and follow up after the trip to learn how you like to travel. Uses the hosted FlightClaw MCP (https://mcp.flightclaw.com/mcp). A local open-source server in this repo searches and price-tracks flights via Google Flights.
---

# flightclaw

FlightClaw is a personal travel-booking agent. It remembers who you are, who you
travel with, the loyalty programs and cards you hold, and how you like to fly —
then recommends, books, pays for, and learns from each trip.

Profiles, booking and payment run on the hosted server
(`https://mcp.flightclaw.com/mcp`, OAuth sign-in). The local server in this repo
only searches and price-tracks flights.

## The flow

### 1. Onboarding (one time)
Set this up once, then reuse forever.

1. **Who you are** — `save_traveler` for yourself (full passport-accurate name,
   DOB, contact, loyalty programmes), then `set_me` to mark that profile as you
   and record home airports.
2. **Companions** — `save_traveler` for each person you travel with
   (`relationship`: spouse/partner/child/parent/friend/colleague). Group them
   with `save_group` (e.g. `family` = `jack,jane`).
3. **Preferences** — `set_preferences`: cabin by haul (e.g. short-haul ECONOMY,
   long-haul BUSINESS), preferred/avoided airlines, alliance, seat, departure
   window, max stops, red-eye tolerance, baggage, meal, and budget sensitivity
   (`cheapest` / `balanced` / `comfort`).
4. **Cards & points** — `save_card` for each card; `set_points_balance` for each
   loyalty/transfer program. Then enrich with the **Card Links** MCP
   (`find_transfer_programs_for_airline`, `list_transfer_partners`) so you know
   which airlines each card's points can reach.

### 2. Planning a trip
1. Ask where they want to go and **who's coming** — reuse a group with
   `get_group` (it returns the exact `passengers` string for booking) or make a
   new one with `save_group`.
2. **Recommend** — `recommend_flights(origin, destination, date, ...)`. It loads
   the saved preferences, picks the cabin by haul, drops avoided airlines, and
   ranks options on price/duration/stops/preferred-airline/departure-window/
   red-eye, returning the top 3 with a "why this fits you" for each.
3. **Awards / points option** — if they want to spend points, call the **Award
   Travel Finder** MCP (`search_availability`, `search_all_airlines`,
   `get_pricing`) using their stored loyalty programs and points balances, and
   present award options alongside the cash fares ("best overall / cheapest /
   best points value").

### 3. Booking & paying

**Hosted server (`https://mcp.flightclaw.com/mcp`)**
1. `search_flights` / `search_multi_city` / `recommend_flights` → pick an
   offer → `get_offer` to confirm price, bags and fare rules. Offers expire in
   about 30 minutes.
2. Confirm the choice with the user, then `create_checkout(offer_id,
   passengers)`. It returns `checkout_id`, `checkout_url` and the exact
   `total_amount` + `total_currency` (airline fare + FlightClaw booking fee).
   Nothing is charged yet.
3. Pay one of two ways:
   - **Checkout link** — give the user `checkout_url` and `total_amount`. The
     user pays there by card.
   - **Link virtual card** — create a Link spend request for exactly
     `total_amount` in `total_currency`. The amount must include the booking
     fee; the fare alone is too low. The user approves the spend in Link. Then
     pay on `checkout_url` with the Link virtual card.
4. `get_checkout_status(checkout_id)` until `completed`, then `get_order`.
   `failed` means no payment was taken and you can retry.

Payment rules:
- Use the `total_amount` from `create_checkout`. Never compute the price yourself.
- Never pay more than the amount the user approved in Link. If the total
  changes, stop, create a new checkout and ask for a new approval.
- Never ask for or type card numbers from the user.

After booking, call **`log_trip`** (route, dates, travelers, cabin, price,
`order_id`) so the trip enters history and the follow-up queue.

### 4. Post-trip follow-up & learning (the real magic)
1. `trips_pending_followup` surfaces trips that have completed/returned.
2. Ask how each went, then `record_trip_feedback(id, feedback, learnings=...)`.
   Durable lessons (e.g. "prefers window on long-haul", "dislikes early
   departures") are appended to the user's preferences, so the **next**
   `recommend_flights` is sharper. Over time FlightClaw learns the traveler.

## Tools

**Personalization (hosted)**
- Travelers: `save_traveler`, `list_travelers`, `get_traveler`, `delete_traveler`,
  `set_me`, `get_me`.
- Preferences: `set_preferences`, `get_preferences`, `update_preferences`.
- Cards/points: `save_card`, `list_cards`, `delete_card`, `set_points_balance`,
  `list_points`.
- Groups: `save_group`, `list_groups`, `get_group`, `delete_group`.
- Trips: `log_trip`, `list_trips`, `get_trip`, `trips_pending_followup`,
  `record_trip_feedback`.
- Recommendation: `recommend_flights`.

**Search & tracking** — `search_flights`, `search_dates`, `track_flight`,
`check_prices`, `list_tracked`, `remove_tracked`.

**Hosted booking** — `get_offer`, `get_seat_map`, `create_checkout`,
`get_checkout_status`, `list_orders`, `get_order`, `request_change`,
`cancel_order`. Hosted price tracking: `track_flight`, `list_tracked`,
`remove_tracked` (checked daily, email alert on a drop).

## External MCP integration

FlightClaw stores the user's cards/points; the agent enriches and acts on them
using two other MCP servers when present:
- **Card Links** — transfer partners and card comparisons for the user's stored
  cards.
- **Award Travel Finder** — award availability and points pricing across
  airlines/programs.

When surfacing card recommendations from Card Links, always include its
disclaimers: not financial advice; affiliate links may earn commission; card
terms change — verify current offers with the issuer.

## Setup

Hosted: `claude mcp add --transport http flightclaw https://mcp.flightclaw.com/mcp`
(sign in with OAuth; no keys needed).

Local (search and price tracking only, no account):

```bash
pip install "flights==0.9.0" "mcp[cli]<2" fastmcp pydantic-settings
claude mcp add flightclaw -- python3 /path/to/agents/server.py
```

## Data

The local server stores price-tracking history in `data/tracked.json`
(gitignored).
