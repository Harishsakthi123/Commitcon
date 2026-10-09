# Smart Classroom Allocation System

FastAPI + SQLAlchemy backend with a pure-Python constraint-based greedy allocation engine, baseline comparison,
conflict detection, dynamic reallocation, analytics, CSV export, seed data and tests.
**Not included yet:** React/Tailwind frontend, Alembic migration files, PDF export, recurring-weekly schedules
(sessions are single dated intervals; create one per occurrence), PostgreSQL exclusion constraint.

## Run
```
cp .env.example .env            # set DATABASE_URL (SQLite default works without it) and SECRET_KEY
cd backend && pip install -r requirements.txt
python ../scripts/seed_data.py  # demo data; admin@demo.local / faculty@demo.local, password = SEED_ADMIN_PASSWORD
uvicorn app.main:app --reload   # Swagger UI: http://localhost:8000/docs
pytest                          # engine + API tests
```
Demo flow: login -> `POST /api/allocations/preview` -> `POST /api/allocations/confirm` -> add closure via
`POST /api/classrooms/{id}/unavailability` -> `GET /api/conflicts` -> preview with `?include_attention=true` -> confirm ->
`GET /api/analytics/compare`. The seed includes a 500-student session that no room can hold.

## Design notes
- Hard constraints (capacity, facilities, closures, overlaps) are checked before scoring and again on confirm, inside one
  transaction (row locks on PostgreSQL, unique `session_id` on allocations). Back-to-back sessions are allowed.
- Scoring weights and meanings are documented in `algorithms/allocator.py`. Greedy + one-level repair is a heuristic, not optimal.
- Utilisation uses an 08:00-18:00 Mon-Fri window per room minus closures. Cancelled sessions are excluded from eligible sessions.
- Datetimes are stored as naive UTC. In production use Alembic instead of `create_all`, and a strong `SECRET_KEY`.
