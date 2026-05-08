# Pontaj-API

Pontaj-API is a FastAPI service used by a mobile client and a web frontend to manage student enrollment and generate short-lived QR tokens for presence checks. This repo contains the server code, integration notes, and instructions to run locally or with Docker.

Key features
- Enrollment flow that issues long-lived JWTs for students
- Short-lived QR tokens (JWT) used for presence scanning
- Per-client static bearer tokens for frontend/admin/mobile checks

Quick start (local)
1. Create a `.env` file with the required variables (example in PONTAJ_INTEGRATION.md):
   - MYSQL_USER, MYSQL_PASSWORD, MYSQL_HOST, MYSQL_PORT, MYSQL_DB
   - SECRET_KEY, API_TOKEN_FRONTEND, API_TOKEN_ADMIN, API_TOKEN_MOBILE
2. Create and activate a virtual environment:
   python -m venv .venv
   .venv\Scripts\activate
3. Install dependencies and run the app:
   pip install -r app/requirements.txt
   uvicorn app.main:app --host 0.0.0.0 --port 8000

Docker
- Build and run with Docker Compose:
  docker compose up --build

Documentation and integration
- API interactive docs: https://api.pontaj.binarysquad.club/docs
- Integration guide: PONTAJ_INTEGRATION.md

Contributing
- For doc changes: open a small PR with readable, incremental updates.
- For code changes: run tests (if any) and describe the behavior in the PR body.
