---
layout: post
title: "Shipping sm2-flashcards: from LeetCode concept to live demo"
date: 2026-05-18
---

I wanted a portfolio that signals "I actually understand CS" instead of "I can follow a React tutorial." The plan: pick a data structure I'd been studying, build a small but production-shaped app around it, and document the algorithm choice in the README. First spoke: a spaced-repetition flashcards app with a from-scratch **Trie** for tag autocomplete and a **binary min-heap** driving an **SM-2** scheduler.

This post is the receipts — the exact websites I used, the commands that worked, and one pivot I had to make along the way.

## The shape

Hub-and-spoke. [jasmineanica.com/projects/](/projects/) is the hub — a Jekyll page driven by `_data/projects.yml`. Each spoke is its own GitHub repo with its own stack and live demo. Adding a new project is one YAML block on the hub plus one new repo.

## The stack

- **Python 3.12 + FastAPI** (async, typed, easy to deploy)
- **SQLAlchemy 2.0 async** + **Alembic** for migrations
- **HTMX + Jinja** for the UI — no separate frontend build
- **SQLite locally, Postgres in production** — same models, different driver
- **Docker** (multi-stage, non-root user, healthcheck)
- **GitHub Actions** for lint / typecheck / test / build
- **Render** for hosting, **Neon** for the database

The two algorithm modules — `app/trie.py` and `app/scheduler.py` — are hand-rolled with full type hints, plus a `pytest-benchmark` test comparing the Trie against a naive linear scan.

## The pivot: Fly.io → Render

I scaffolded the whole thing assuming Fly.io. Then I discovered Fly's free tier is gone — every machine costs money now, even small ones. So I pivoted to **Render** for the web service and **Neon** for Postgres. Render's free web tier is real (spins down after 15 min idle, cold-start ~30s) and Neon's free Postgres doesn't expire (Render's own free Postgres does, after 90 days).

The pivot was mechanical: delete `fly.toml`, write `render.yaml`, drop the deploy job from CI (Render auto-deploys on push), update three URLs in the docs.

## Websites I used

| Site | What for |
|---|---|
| [github.com](https://github.com) | Repos + Actions CI |
| [render.com](https://render.com) | Web service hosting (free tier, Docker) |
| [neon.tech](https://neon.tech) | Postgres (free tier, doesn't expire) |
| [shields.io](https://shields.io) | README badges (CI, license, Python version) |
| [htmx.org](https://htmx.org) | The whole frontend |

## Commands that mattered

Scaffold and push the new repo:
```bash
mkdir sm2-flashcards && cd sm2-flashcards
# (... wrote pyproject.toml, app/, tests/, Dockerfile, render.yaml ...)
git init -b main
git add . && git commit -m "Initial scaffold"
git remote add origin git@github.com:jasmineanica/sm2-flashcards.git
git push -u origin main
```

Local dev loop:
```bash
python3.12 -m venv .venv && source .venv/bin/activate
make install         # pip install -e ".[dev]"
make migrate         # alembic upgrade head
make dev             # uvicorn on :8000
make test            # pytest
make bench           # Trie vs naive linear scan
make lint typecheck  # ruff + mypy strict
```

Connecting Neon to Render — the one thing that bit me:

Neon hands you a `postgresql://` URL. SQLAlchemy's async driver wants `postgresql+asyncpg://`. Also strip `?sslmode=require` — asyncpg uses different SSL params. The final form:

```
postgresql+asyncpg://user:pass@ep-xxxx.region.aws.neon.tech/dbname
```

Paste that into Render → service → **Environment** tab → `DATABASE_URL`. Save triggers a redeploy.

## Things I learned

- **`ruff format` is opinionated about line wraps.** If a line fits under your `line-length`, it'll force you to keep it on one line. My CI failed twice on minor wrap inconsistencies before I got the muscle memory.
- **`Depends(get_session)` as a default argument is the old FastAPI idiom.** Modern style is `Annotated[AsyncSession, Depends(get_session)]`, and `ruff` flags the old form as `B008`. Define the alias once in your db module and import it everywhere.
- **PEP 695 generics need Python 3.12.** I tried local syntax checks with the system's Python 3.9 — they fail, but CI is on 3.12 so it doesn't matter. Just know your local interpreter is lying to you.
- **Render's blueprint syntax is forgiving.** It auto-detected my Dockerfile, used the healthcheck path from `render.yaml`, and respected the `sync: false` flag on `DATABASE_URL` so it wouldn't try to fill in a value itself.
- **Empty-state matters in a demo.** The README claimed "Trie autocomplete on tags," but with an empty DB the autocomplete did nothing. I added a lifespan-hook seeder that drops six interview-prep cards on first boot, plus a search box on the home page that actually exercises the Trie via HTMX. Now the claim is visible, not just true.

## What's next

The point of the hub-and-spoke setup is that the second project should be cheaper than the first. The template is dialed in — Dockerfile, CI, Render blueprint, README skeleton, MIT license, badges. The next spoke is probably a **rate limiter** (sliding window or token bucket, Redis-backed), and the loop is documented at [/docs/adding-a-project/](/docs/adding-a-project/).

Live demo: [sm2-flashcards.onrender.com](https://sm2-flashcards.onrender.com)
Code: [github.com/jasmineanica/sm2-flashcards](https://github.com/jasmineanica/sm2-flashcards)
