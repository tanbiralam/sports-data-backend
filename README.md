# Sports Backend API

A real-time sports live data backend server built with Node.js, Express, PostgreSQL, and WebSockets.

## Features

- **RESTful API** for managing sports matches and commentary
- **Real-time Updates** via WebSocket for live match events
- **Database** PostgreSQL with Drizzle ORM for type-safe queries
- **Input Validation** using Zod schemas
- **Security** Arcjet protection (rate limiting, bot detection, shield)
- **Monitoring** APM Insight integration for performance tracking
- **Auto Status** Match status automatically transitions based on start/end times

## Tech Stack

| Technology     | Purpose                  |
| -------------- | ------------------------ |
| Node.js        | Runtime environment      |
| Express.js     | HTTP server & REST API   |
| PostgreSQL     | Database                 |
| Drizzle ORM    | Database ORM             |
| WebSocket (ws) | Real-time communication  |
| Zod            | Input validation         |
| Arcjet         | Security & rate limiting |
| APM Insight    | Application monitoring   |

## Project Structure

```
sports-backend/
├── drizzle/                    # Database migrations
│   ├── meta/                   # Migration metadata
│   ├── 0000_tearful_tusk.sql   # Initial migration
│   └── drizzle.config.js      # Drizzle configuration
├── src/
│   ├── arcjet.js               # Arcjet security configuration
│   ├── index.js                # Application entry point
│   ├── db/
│   │   ├── db.js               # Database connection
│   │   └── schema.js           # Database schema definitions
│   ├── routes/
│   │   ├── matches.js          # Match endpoints
│   │   └── commentary.js       # Commentary endpoints
│   ├── seed/
│   │   └── seed.js             # Database seeding script
│   ├── utils/
│   │   └── match-status.js     # Match status utilities
│   ├── validation/
│   │   ├── matches.js          # Match validation schemas
│   │   └── commentary.js       # Commentary validation schemas
│   └── ws/
│       └── server.js           # WebSocket server
├── package.json                # Project dependencies
└── README.md                   # This file
```

## Getting Started

### Prerequisites

- Node.js 18+
- PostgreSQL database
- npm or yarn

### Installation

1. Clone the repository:

```bash
git clone <repository-url>
cd sports-backend
```

2. Install dependencies:

```bash
npm install
```

3. Configure environment variables:

Create a `.env` file in the root directory:

```env
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/sports_db

# Server
PORT=8000
HOST=0.0.0.0

# Arcjet Security (get your key from https://arcjet.com)
ARCJET_KEY=your_arcjet_key
ARCJET_ENV=LIVE  # or DRY_RUN for testing
```

> **Note**: The `.env` file is gitignored. See `.env.example` for reference.

### Database Setup

1. Generate database migrations:

```bash
npm run db:generate
```

2. Run migrations:

```bash
npm run db:migrate
```

3. (Optional) Seed the database with sample data:

```bash
npm run seed
```

### Running the Server

**Development mode** (with auto-reload):

```bash
npm run dev
```

**Production mode:**

```bash
npm start
```

The server will start on `http://localhost:8000` (or the port specified in your `.env`).

## API Endpoints

### Health Check

| Method | Endpoint | Description         |
| ------ | -------- | ------------------- |
| GET    | `/`      | Server health check |

**Response:**

```json
"Sports live data server is running"
```

### Matches

| Method | Endpoint             | Description        |
| ------ | -------------------- | ------------------ |
| GET    | `/matches`           | List all matches   |
| POST   | `/matches`           | Create a new match |
| PATCH  | `/matches/:id/score` | Update match score |

#### GET /matches

Query parameters:

- `limit` (optional): Number of results (max 100, default 50)

**Response:**

```json
{
  "data": [
    {
      "id": 1,
      "sport": "football",
      "homeTeam": "Team A",
      "awayTeam": "Team B",
      "status": "live",
      "startTime": "2024-01-15T10:00:00Z",
      "endTime": "2024-01-15T12:00:00Z",
      "homeScore": 2,
      "awayScore": 1,
      "createdAt": "2024-01-15T09:00:00Z"
    }
  ]
}
```

#### POST /matches

Request body:

```json
{
  "sport": "football",
  "homeTeam": "Team A",
  "awayTeam": "Team B",
  "startTime": "2024-01-15T10:00:00Z",
  "endTime": "2024-01-15T12:00:00Z",
  "homeScore": 0,
  "awayScore": 0
}
```

**Validation:**

- `sport`: Required, string
- `homeTeam`: Required, string
- `awayTeam`: Required, string
- `startTime`: Required, ISO 8601 datetime
- `endTime`: Required, ISO 8601 datetime (must be after startTime)
- `homeScore`: Optional, non-negative integer (default: 0)
- `awayScore`: Optional, non-negative integer (default: 0)

#### PATCH /matches/:id/score

Request body:

```json
{
  "homeScore": 3,
  "awayScore": 1
}
```

### Commentary

| Method | Endpoint                  | Description               |
| ------ | ------------------------- | ------------------------- |
| GET    | `/matches/:id/commentary` | Get match commentary      |
| POST   | `/matches/:id/commentary` | Add commentary to a match |

#### GET /matches/:id/commentary

Query parameters:

- `limit` (optional): Number of results (max 100, default 10)

**Response:**

```json
{
  "data": [
    {
      "id": 1,
      "matchId": 1,
      "minute": 45,
      "sequence": 1,
      "period": "first_half",
      "eventType": "goal",
      "actor": "Player Name",
      "team": "home",
      "message": "GOAL! Team A takes the lead!",
      "metadata": {},
      "tags": ["goal", "important"],
      "createdAt": "2024-01-15T10:45:00Z"
    }
  ]
}
```

#### POST /matches/:id/commentary

Request body:

```json
{
  "minute": 45,
  "sequence": 1,
  "period": "first_half",
  "eventType": "goal",
  "actor": "Player Name",
  "team": "home",
  "message": "GOAL! Team A takes the lead!",
  "metadata": {},
  "tags": ["goal", "important"]
}
```

**Validation:**

- `minute`: Required, non-negative integer
- `sequence`: Optional, integer
- `period`: Optional, string
- `eventType`: Optional, string
- `actor`: Optional, string
- `team`: Optional, string
- `message`: Required, string
- `metadata`: Optional, JSON object
- `tags`: Optional, array of strings

## WebSocket API

Connect to: `ws://localhost:8000/ws`

### Client Messages

#### Subscribe to a match

```json
{
  "type": "subscribe",
  "matchId": 1
}
```

#### Unsubscribe from a match

```json
{
  "type": "unsubscribe",
  "matchId": 1
}
```

### Server Messages

#### Welcome message

```json
{
  "type": "welcome"
}
```

#### Subscription confirmation

```json
{
  "type": "subscribed",
  "matchId": 1
}
```

#### New match created (broadcast to all)

```json
{
  "type": "match_created",
  "data": { ... }
}
```

#### New commentary (broadcast to subscribers)

```json
{
  "type": "commentary",
  "data": { ... }
}
```

### WebSocket Example (JavaScript)

```javascript
const ws = new WebSocket("ws://localhost:8000/ws");

ws.onopen = () => {
  console.log("Connected to WebSocket");

  // Subscribe to match updates
  ws.send(JSON.stringify({ type: "subscribe", matchId: 1 }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log("Received:", data);

  switch (data.type) {
    case "welcome":
      console.log("Welcome! Connected to sports server.");
      break;
    case "subscribed":
      console.log(`Subscribed to match ${data.matchId}`);
      break;
    case "match_created":
      console.log("New match created:", data.data);
      break;
    case "commentary":
      console.log("New commentary:", data.data);
      break;
  }
};
```

## Match Status

Matches automatically transition between statuses based on the current time:

| Status      | Condition                                     |
| ----------- | --------------------------------------------- |
| `scheduled` | Current time is before startTime              |
| `live`      | Current time is between startTime and endTime |
| `finished`  | Current time is after endTime                 |

The status is automatically updated when:

- Creating a new match
- Updating match score
- Fetching match data

## Security (Arcjet)

This project uses [Arcjet](https://arcjet.com) for security:

### HTTP Protection

- **Shield**: Protection against common attacks
- **Bot Detection**: Blocks malicious bots (allows search engine crawlers)
- **Rate Limiting**: 50 requests per 10 seconds per IP

### WebSocket Protection

- **Shield**: Protection against common attacks
- **Bot Detection**: Blocks malicious bots
- **Rate Limiting**: 5 connections per 2 seconds per IP

To enable Arcjet, set the `ARCJET_KEY` environment variable. For testing, set `ARCJET_ENV=DRY_RUN` to log violations without blocking.

## NPM Scripts

| Script                | Description                                       |
| --------------------- | ------------------------------------------------- |
| `npm run dev`         | Start server in development mode with auto-reload |
| `npm start`           | Start server in production mode                   |
| `npm run db:generate` | Generate database migrations                      |
| `npm run db:migrate`  | Run database migrations                           |
| `npm run seed`        | Seed database with sample data                    |

## Error Responses

All error responses follow this format:

```json
{
  "error": "Error message",
  "details": "Additional error details (optional)"
}
```

Common status codes:

- `400` - Bad Request (validation error)
- `403` - Forbidden (security violation)
- `404` - Not Found
- `429` - Too Many Requests (rate limit)
- `500` - Internal Server Error
- `503` - Service Unavailable

## Database Schema

### matches table

| Column    | Type      | Description                            |
| --------- | --------- | -------------------------------------- |
| id        | serial    | Primary key                            |
| sport     | text      | Sport type (e.g., "football")          |
| homeTeam  | text      | Home team name                         |
| awayTeam  | text      | Away team name                         |
| status    | enum      | Match status (scheduled/live/finished) |
| startTime | timestamp | Match start time                       |
| endTime   | timestamp | Match end time                         |
| homeScore | integer   | Home team score                        |
| awayScore | integer   | Away team score                        |
| createdAt | timestamp | Record creation time                   |

### commentary table

| Column    | Type      | Description            |
| --------- | --------- | ---------------------- |
| id        | serial    | Primary key            |
| matchId   | integer   | Foreign key to matches |
| minute    | integer   | Match minute           |
| sequence  | integer   | Sequence number        |
| period    | text      | Match period           |
| eventType | text      | Type of event          |
| actor     | text      | Player/action actor    |
| team      | text      | Team involved          |
| message   | text      | Commentary text        |
| metadata  | jsonb     | Additional metadata    |
| tags      | text[]    | Array of tags          |
| createdAt | timestamp | Record creation time   |

## License

ISC
