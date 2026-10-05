# FlightClaw agents: book flights with AI

Search, price-track and book flights from your AI assistant. FlightClaw is an
AI flight booking agent and MCP server for Claude, ChatGPT, Cursor, VS Code and
any MCP client. It covers about 300 airlines, finds cheap flights, remembers
your travellers and preferences, and works as a flight price tracker that emails
you when a tracked fare drops. Learn more at [flightclaw.com](https://flightclaw.com).

## Skills

Agent skills for the hosted MCP server (`https://mcp.flightclaw.com/mcp`). Each
skill tells the agent which tools to call, in which order, and when to ask the
user first.

| Skill | Use it to |
|---|---|
| [book-flight](skills/book-flight/SKILL.md) | Search, compare and book flights with AI, then pay by checkout link or Link virtual card |
| [track-flight-price](skills/track-flight-price/SKILL.md) | Track a fare daily and get an email alert when the price drops |
| [plan-group-trip](skills/plan-group-trip/SKILL.md) | Save travellers and groups, search multi-city trips and book everyone at once |
| [manage-booking](skills/manage-booking/SKILL.md) | Find orders and booking references, quote changes and cancel with a refund quote |
| [traveler-profile](skills/traveler-profile/SKILL.md) | Save you, companions, preferences, cards, points balances and trip history |

The root [SKILL.md](SKILL.md) covers the local open-source server.

## Option 1: Hosted MCP (recommended)

Server URL: `https://mcp.flightclaw.com/mcp` (streamable HTTP, OAuth sign-in).
Nothing to install.

| Client | Setup |
|---|---|
| Claude (web / desktop) | Settings > Connectors > Add custom connector > paste `https://mcp.flightclaw.com/mcp` |
| ChatGPT | Settings > Apps & Connectors > Create (developer mode) > paste `https://mcp.flightclaw.com/mcp` |
| Claude Code | `claude mcp add --transport http flightclaw https://mcp.flightclaw.com/mcp` |
| Cursor | [One-click install](https://cursor.com/en/install-mcp?name=flightclaw&config=eyJ1cmwiOiJodHRwczovL21jcC5mbGlnaHRjbGF3LmNvbS9tY3AifQ%3D%3D), or add `{"mcpServers":{"flightclaw":{"url":"https://mcp.flightclaw.com/mcp"}}}` to `~/.cursor/mcp.json` |
| VS Code | [One-click install](https://vscode.dev/redirect/mcp/install?name=flightclaw&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.flightclaw.com%2Fmcp%22%7D), or `code --add-mcp '{"name":"flightclaw","type":"http","url":"https://mcp.flightclaw.com/mcp"}'` |
| Windsurf | Add `{"mcpServers":{"flightclaw":{"serverUrl":"https://mcp.flightclaw.com/mcp"}}}` to `~/.codeium/windsurf/mcp_config.json` |

### Hosted tools

| Group | Tools |
|---|---|
| Search | `search_flights`, `search_multi_city`, `recommend_flights`, `get_offer`, `get_seat_map` |
| Booking | `create_checkout`, `get_checkout_status`, `list_orders`, `get_order`, `request_change`, `cancel_order` |
| Price tracking | `track_flight`, `list_tracked`, `remove_tracked` (checked daily, email alert on a drop) |
| Profile | `get_me`, `set_me`, `save_traveler`, `list_travelers`, `get_traveler`, `delete_traveler`, `save_group`, `list_groups`, `get_group`, `delete_group`, `get_preferences`, `set_preferences`, `save_card`, `list_cards`, `delete_card`, `set_points_balance`, `list_points` |
| Trips | `log_trip`, `list_trips`, `get_trip`, `trips_followup`, `record_trip_feedback` |

### Booking and payment

1. `search_flights` returns offers. Each `total_amount` includes the FlightClaw booking fee.
2. `create_checkout` returns a `checkout_url` and the exact `total_amount` + `total_currency`. Nothing is charged yet.
3. Pay one of two ways:
   - **Checkout link**: the traveller opens `checkout_url` and pays by card.
   - **Link virtual card**: the agent creates a Link spend request for exactly
     `total_amount` (fare + booking fee), the user approves it in Link, and the agent pays
     on `checkout_url` with the Link virtual card. The agent never
     pays more than the approved amount.
4. `get_checkout_status` until `completed`, then `get_order` for the booking reference.

The agent never asks for card numbers.

## Option 2: Local open-source server

The rest of this README covers the self-hosted Python server in this repo.
It searches Google Flights and tracks prices locally with no account.

## Local MCP server

FlightClaw runs as a local [MCP](https://modelcontextprotocol.io) server, giving any MCP-compatible client (Claude Code, Claude Desktop, etc.) access to flight search and tracking tools.

### Setup

```bash
# Install dependencies
pip install "flights==0.9.0" "mcp[cli]<2" fastmcp pydantic-settings

# Add to Claude Code
claude mcp add flightclaw -- python3 /path/to/agents/server.py
```

Or in Claude Desktop, add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "flightclaw": {
      "command": "python3",
      "args": ["/path/to/agents/server.py"]
    }
  }
}
```

### Tools

| Tool | Description |
|------|-------------|
| `search_flights` | Search Google Flights for prices on a route |
| `search_dates` | Find cheapest dates to fly across a date range (calendar view) |
| `track_flight` | Add a route to price tracking with optional target price |
| `check_prices` | Check all tracked flights for price changes and alerts |
| `list_tracked` | List all tracked flights with price history |
| `remove_tracked` | Remove a route from tracking |

### Search filters

All search tools support:

- **Passengers** - adults, children, infants (in seat or on lap)
- **Airlines** - filter to specific carriers (e.g. `BA,AA,DL`)
- **Price limit** - max price in USD
- **Duration** - max total flight time in minutes
- **Times** - earliest/latest departure and arrival hours
- **Layovers** - max layover duration in minutes
- **Sorting** - by BEST, CHEAPEST, DEPARTURE, ARRIVAL, or DURATION
- **Multi-airport** - comma-separated codes (e.g. `LHR,MAN`)
- **Date ranges** - `date_to` for searching each day in a range

### Example prompts

- "Search flights from LHR to JFK on 2025-08-01 in business class"
- "Find nonstop BA or VS flights LHR to JFK departing after 8am"
- "What are the cheapest dates to fly LHR to JFK in July?"
- "Search for 2 adults and 1 child, LHR to JFK, under $500"
- "Track LHR to SFO on 2025-07-01 with a target price of $400"
- "Check my tracked flights for price drops"

## CLI Scripts

The original CLI scripts are still available in `scripts/`:

```bash
# Search flights
python scripts/search-flights.py LHR JFK 2025-07-01 --cabin BUSINESS

# Multiple airports and date ranges
python scripts/search-flights.py LHR,MAN JFK,EWR 2025-07-01 --date-to 2025-07-05

# Track a route
python scripts/track-flight.py LHR JFK 2025-07-01 --target-price 400

# Check for price drops (good for cron)
python scripts/check-prices.py --threshold 5

# List tracked flights
python scripts/list-tracked.py
```

## How it works

- Queries Google Flights via the `fli` library
- Prices returned in user's local currency (auto-detected from IP)
- Price history persists in `data/tracked.json`
- Supports one-way and round trips, all cabin classes (economy to first)
- Filter by airline, price, duration, departure/arrival times, layover duration
- Multi-airport and date-range searches expand into all combinations
- Date search finds the cheapest day to fly across a range

## Install (OpenClaw)

```bash
npx skills add flightclaw/agents
```

## Install (Grok)

```
/plugin marketplace add xai-org/plugin-marketplace
/plugin install flightclaw
```

## Install (Claude Code)

```
/plugin marketplace add flightclaw/agents
/plugin install flightclaw@flightclaw
```

## Install (Cursor)

```
/plugin marketplace add flightclaw/agents
/plugin install flightclaw
```

## Install (Gemini CLI)

```bash
gemini extensions install https://github.com/flightclaw/agents
```

Each client reads its own manifest from this repo — `.grok-plugin/`,
`.claude-plugin/`, `.cursor-plugin/` and `gemini-extension.json` — and they all
connect to the hosted server at `https://mcp.flightclaw.com/mcp`. To run the
local server instead, use the setup in "Local MCP server" above.

## The local server

The local server runs `server.py` through `uv`, which resolves the pinned
dependencies at start-up. Install [uv](https://docs.astral.sh/uv/) first; no
other setup step runs on your machine.

### What the local server reaches, and what it needs

| Endpoint | Purpose | Credentials |
|---|---|---|
| `google.com/travel/flights` (via the `fli` library) | Flight search and price tracking | none |

Search and price tracking need no credentials. `HOST` and `PORT` are only read in HTTP transport mode.

FlightClaw reads no other environment variable, writes only to its own `data/`
directory, and runs no install-time script.

## License

MIT — see [LICENSE](LICENSE).
