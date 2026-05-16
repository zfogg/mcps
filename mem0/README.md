# mem0 Docker Compose Setup

Complete self-hosted mem0 stack with PostgreSQL+pgvector database and Next.js dashboard.

## Quick Start

### Local Development

```bash
# Copy env template
cp .env.example .env

# Update .env with your LLM provider key
# - OPENAI_API_KEY=sk-...
# - Or ANTHROPIC_API_KEY=... (Haiku recommended for cost)
# - Or GOOGLE_API_KEY=...

# Start stack
docker compose up

# Access:
# - Dashboard: http://localhost:3000
# - API Swagger UI: http://localhost:8888/docs
# - Postgres: localhost:8432
```

The first time, you'll hit the setup wizard at `http://localhost:3000/setup` to create an admin account and API key.

### Coolify Deployment

1. **Push to Git**: Commit `mem0/` to your repo
2. **Coolify Setup**:
   - New deployment → Docker Compose
   - Compose file path: `mem0/docker-compose.yml`
3. **Domains**: In Coolify UI, map:
   - `mem0.yourdomain.com` → `mem0-dashboard` port 3000
   - `mem0-api.yourdomain.com` → `mem0` port 8888
4. **Secrets**: Set in Coolify environment:
   - `OPENAI_API_KEY`
   - `JWT_SECRET` (generate: `openssl rand -base64 48`)
   - `POSTGRES_PASSWORD` (secure random string)
   - `DASHBOARD_URL=https://mem0.yourdomain.com`
   - `MEM0_API_URL=https://mem0-api.yourdomain.com`

## Services

| Service | Port | Purpose |
|---------|------|---------|
| `mem0` | 8888 | FastAPI REST API |
| `mem0-dashboard` | 3000 | Next.js web UI |
| `postgres` | 5432 (internal) | PostgreSQL + pgvector |

## API Usage

Once set up, get your API key from the dashboard. Store memories:

```bash
curl -X POST http://localhost:8888/memories \
  -H "X-API-Key: m0sk_..." \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "user", "content": "I am Bob, I love TypeScript"}
    ],
    "user_id": "user_123"
  }'
```

Retrieve memories:

```bash
curl "http://localhost:8888/memories?query=what+do+you+know+about+me&user_id=user_123" \
  -H "X-API-Key: m0sk_..."
```

See `/docs` (Swagger UI) for full API reference.

## Environment Variables

See `.env.example` for all options. Key ones:

- **`OPENAI_API_KEY`** — Required. OpenAI API key for LLM extraction and embeddings.
- **`POSTGRES_PASSWORD`** — Required for production. Change from default `testpassword123`.
- **`JWT_SECRET`** — Required when auth is enabled (default). Generate: `openssl rand -base64 48`.
- **`AUTH_DISABLED`** — Set `true` only for local dev. Never in production.
- **`DASHBOARD_URL`** — CORS origin for dashboard calls to API. Set to your dashboard domain.
- **`MEM0_API_URL`** — Public URL of the API. Used by dashboard in browser.

## Architecture

```
Dashboard (Next.js)
       ↓
  API (FastAPI)
       ↓
  PostgreSQL + pgvector
```

- Dashboard talks to API via `MEM0_API_URL`
- API uses PostgreSQL for both relational data and vector storage (pgvector)
- No separate vector DB needed
- Auth via JWT or per-user API keys (`m0sk_` prefix)

## Data Persistence

- **Database**: `postgres_data` Docker volume persists across restarts
- **History**: `/app/history/` in mem0 container

## Troubleshooting

**API won't start**: Check logs with `docker compose logs mem0`. Common issues:
- Missing `OPENAI_API_KEY` (or Anthropic/Google equivalent)
- PostgreSQL not healthy yet (wait 10s on first start)
- Port 8888 already in use

**Dashboard shows "API unreachable"**: Check `MEM0_API_URL` is set correctly in env and points to the API's public domain.

**PostgreSQL volume issues**: First start may take 30s for migrations. Be patient.

## Development

Edit code and restart:

```bash
# After changing Python code
docker compose up -d --build mem0

# After changing Next.js code
docker compose up -d --build mem0-dashboard
```

## Production Checklist

- [ ] Set `OPENAI_API_KEY` to a real key
- [ ] Set `POSTGRES_PASSWORD` to a strong random password
- [ ] Generate and set `JWT_SECRET` (example: `openssl rand -base64 48`)
- [ ] Set `DASHBOARD_URL` and `MEM0_API_URL` to your real domains
- [ ] Set `AUTH_DISABLED=false` (default; never disable in prod)
- [ ] Set `MEM0_TELEMETRY=false` if you prefer
- [ ] Enable Coolify automatic backups for the Postgres volume
- [ ] Monitor logs for errors

## Access from Multiple Computers

Once deployed to Coolify:
- **Dashboard**: `https://mem0.yourdomain.com`
- **API**: `https://mem0-api.yourdomain.com`
- **From anywhere**: Use these HTTPS URLs from any device with internet

Coolify handles TLS termination and reverse proxying automatically.
