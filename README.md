# Bid Pricing Service

## What is this, in plain words?

Imagine you own a cricket website (like CricBuzz).

Someone opens an article: "India vs Australia Live Match Updates."

At the top of the page there is an empty ad space:

```text
+------------------------------------------+
|  CricBuzz                                |
|------------------------------------------|
|  India vs Australia                      |
|  Live Match Updates                      |
|                                          |
|  +------------------------------------+  |
|  |                                    |  |
|  |         empty AD SPACE             |  |
|  |                                    |  |
|  +------------------------------------+  |
|                                          |
|  Match score, commentary, etc.           |
+------------------------------------------+
```

Companies like Nike, Amazon, or Adidas want that space. They bid for it:

```text
              empty AD SPACE
                     |
        +------------+------------+
        |            |            |
      Nike        Amazon       Adidas
       ₹8          ₹10          ₹12
        |            |            |
        +------------+------------+
                     |
               highest bid
                     |
                  Adidas wins
                     |
            Adidas ad is shown
```

You (the website owner) earn money.

### Important detail

Nike does **not** visit your website and ask "how much should I pay?"

What really happens:

1. A visitor opens your website
2. An empty ad space becomes available
3. The ad marketplace tells advertisers: "this space is open for this visitor"
4. Each advertiser's system asks: "how much should we bid?"
5. A pricing system looks at past similar ads and suggests a price
6. The advertiser decides whether to bid that amount
7. The highest bid wins and that ad is shown

So:

| Who | Role |
|---|---|
| Your website visitor | Opens the page (this starts everything) |
| Ad marketplace | Connects the empty ad space with advertisers |
| This service (PubX pricing) | Suggests a good bid amount from past data |
| Advertiser (e.g. Nike) | Decides whether to actually bid |

### What this project builds

This repo is the **bid pricing** part.

When an ad opportunity appears, this service answers quickly:

> "For this visitor and this ad space, a good bid is about ₹11.50."

It uses things like:

- who the publisher is
- what kind of page it is
- device type (phone / desktop)
- past click rates
- how viewable the ad was before

It must be fast. A visitor will not wait several seconds for an ad.

---

## What the service can do

- **Sync pricing:** send details, get a recommended price right away (`POST /price`)
- **Async pricing:** submit a job, then check later (`POST /price/async` and `GET /jobs/{job_id}`)
- **Fake ML model:** simple model that calculates a price (stand-in for a real model)
- **Cache:** remembers recent answers so repeats are faster
- **Logs and metrics:** JSON logs and Prometheus metrics at `/metrics`
- **Health check:** `GET /health`

---

## Tech stack

| Piece | Tool |
|---|---|
| Language | Python 3.13+ |
| API | FastAPI + Uvicorn |
| Data checks | Pydantic v2 |
| Packages | [uv](https://github.com/astral-sh/uv) |
| Metrics | prometheus-client |
| Logging | python-json-logger |
| Tests | pytest + pytest-asyncio |

---

## Folder layout

```
bid-pricing-service/
├── main.py                      # API app and routes
├── schemas.py                   # Request and response shapes
├── ml_models.py                 # Fake ML model and cache
├── services/
│   ├── pricing_services.py      # Pricing logic
│   ├── observability.py         # Logs and metrics
│   └── queue_service.py         # Async jobs (in memory)
├── tests/
│   ├── test_pricing.py
│   ├── test_cache.py
│   ├── test_queue.py
│   └── test_integration.py
├── pyproject.toml
├── uv.lock
└── README.md
```

---

## What you need

- [uv](https://docs.astral.sh/uv/)
- Python 3.13+ (see `.python-version`)

---

## Setup

### 1. Go into the project

```bash
cd bid-pricing-service
```

### 2. Install packages

```bash
uv sync
```

This creates a `.venv` folder and installs everything from `pyproject.toml`.

### 3. Start the server

```bash
uv run python main.py
```

Or with auto-reload:

```bash
uv run uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

Open [http://localhost:8000](http://localhost:8000).

API docs: [http://localhost:8000/docs](http://localhost:8000/docs)

---

## API examples

### Get a recommended price now

Think of this request as: "an ad space just opened for this visitor. What should we bid?"

```bash
curl -X POST http://localhost:8000/price \
  -H "Content-Type: application/json" \
  -d '{
    "publisher_id": "pub_123",
    "domain": "news.com",
    "device_type": "desktop",
    "time_of_day": 14,
    "historical_ctr": 0.025,
    "viewability_score": 0.85,
    "timestamp": "2025-01-15T10:30:00Z"
  }'
```

### Get a price later (async)

Useful when you want to queue the work and check back.

```bash
# Start a job
curl -X POST http://localhost:8000/price/async \
  -H "Content-Type: application/json" \
  -d '{ ...same JSON as above... }'

# Check the job
curl http://localhost:8000/jobs/{job_id}
```

### Health and metrics

```bash
curl http://localhost:8000/health
curl http://localhost:8000/metrics
```

---

## Tests

Run all tests:

```bash
uv run pytest -v
```

Run one file:

```bash
uv run pytest tests/test_pricing.py -v
```

---

## Why we built it this way

This is a learning / interview-style project. We keep pieces simple on purpose.

| Topic | Choice | Reason |
|---|---|---|
| Kafka | Skipped for now | Easier to build and test with plain HTTP |
| Job queue | In memory | No extra tools needed. Can move to Redis later |
| Cache | Dict with TTL | Simple. Can move to Redis later |
| Model calls | Async | Keeps the server from freezing under load |

### What to improve later

- Use Redis or SQS instead of an in-memory queue
- Save jobs in a database
- Add a real Kafka consumer
- Run more than one server instance

---

## Tips

- Use `uv run ...` so you do not have to activate the venv by hand
- Add a package: `uv add <package>`
- Add a test package: `uv add --dev <package>`
- `.venv/` and `.pytest_cache/` are ignored by git

---

## License

Private for now. Add a license here when you pick one.
