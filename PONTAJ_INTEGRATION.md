# PONTAJ — Integration & API Guide (English / Română)

This guide explains how to use the Pontaj API, how the server is organized, and how a mobile client should integrate with it. It keeps technical details compact and highlights the exact requests, headers, and configuration that matter for integration.

Links
- API base: https://api.pontaj.binarysquad.club/
- API interactive docs (Swagger/OpenAPI): https://api.pontaj.binarysquad.club/docs
- Server repo: https://github.com/xynnpg/ServerAppPontaj
- Mobile example repo: https://github.com/some-randomGuy03/pontaj_mobile

Executive summary
Pontaj is a FastAPI-based service used by mobile and web clients. Mobile users enroll using a student code (codmatricol). Enrollment issues a long-lived JWT (stored in the DB), which the mobile app uses for protected calls and to request short-lived QR tokens for presence scanning.

Architecture (short)
- Server: FastAPI (app/main.py) with routers for /mobile, /frontend, /admin and health endpoints.
- Database: MySQL (async driver asyncmy + SQLAlchemy async engine).
- Auth: combination of per-client static bearer tokens (for frontend/admin/mobile health checks) and JWTs for user sessions and QR tokens.

Authentication (quick reference)
- Static client tokens: set via env vars in app/core/config.py:
  - API_TOKEN_FRONTEND, API_TOKEN_ADMIN, API_TOKEN_MOBILE
  These are validated with make_bearer_verifier(...) and applied to routes that require client-level access.

- Enroll JWT (user token): POST /mobile/enroll issues a long-lived JWT (in current code: ~1 year) signed with SECRET_KEY and saved to elevi.Token in the DB.

- Short-lived QR JWT: POST /mobile/qr and /mobile/qr_image return a short-lived token (about 30s) encoded with SECRET_KEY; the token payload contains a DB identifier that a verifier can use to lookup the student.

Important notes
- /mobile/status requires the static API_TOKEN_MOBILE bearer token.
- /mobile/enroll is currently callable without a static bearer token — it expects a JSON body {"codmatricol":"..."} and returns a long-lived access_token on success.
- Verify whether admin CRUD routes must require static admin bearer token or JWT; code has a possible mismatch (verify_elevi defined but CRUD uses verify_jwt_token).

API highlights
1) Health
- GET /health/live — liveness probe, 200: {"status":"alive"}
- GET /health/ready — readiness probe (checks DB), 200: {"status":"ready","dependencies":{"mysql":"ok"}} or 503 on failure.

2) Mobile
- POST /mobile/enroll
  - Body: {"codmatricol":"<string>"}
  - Auth: none (current code)
  - Success: 200 with {codmatricol, access_token, token_type, name, expires_in}
  - Errors: 401 invalid, 409 already enrolled

- GET /mobile/status
  - Header: Authorization: Bearer <API_TOKEN_MOBILE>
  - Response: {"client":"mobile","status":"ok"}

- GET /mobile/verifyToken
  - Header: Authorization: Bearer <ENROLL_JWT>
  - Response: {"Activ": <int>, "Name": "..."}

- POST /mobile/qr
  - Header: Authorization: Bearer <ENROLL_JWT>
  - Response: {"expires_in":30, "token":"<short_jwt>", "qr_png_base64":"...", "exp":<unix_ts>}

- POST /mobile/qr_image
  - Header: Authorization: Bearer <ENROLL_JWT>
  - Response: raw image/png

3) Frontend
- Routes protected by API_TOKEN_FRONTEND (static bearer)
  - GET /frontend/status
  - GET /frontend/items

4) Admin (elevi)
- CRUD endpoints for elevi are present (GET, POST, PUT, DELETE). Current implementation uses JWT verify_jwt_token; consider whether these should require API_TOKEN_ADMIN or an admin JWT.

Database notes (elevi)
Common columns used in code:
- ID (int PK), Token (string — enroll JWT), Email, Name, Activ (0/1), CodMatricol, DataActivare (datetime).

Mobile integration flow (concise)
A. Enrollment
1. User submits CodMatricol in the mobile app.
2. Mobile calls POST /mobile/enroll with JSON {"codmatricol":"<code>"}.
3. On success, store access_token securely (Keychain/Keystore).

B. Using enroll token
- Include header: Authorization: Bearer <access_token>
- To get a QR for presence: POST /mobile/qr or /mobile/qr_image.

C. Verifying scanned QR
1. Extract short-lived_jwt from the QR.
2. Verify JWT signature using SECRET_KEY and check exp.
3. Use payload value (DB id) to find the student.

Security considerations
- Do NOT commit SECRET_KEY or API tokens to VCS.
- Store mobile tokens in secure storage (Keychain/Keystore).
- Consider shortening the enroll token lifetime (current code uses ~1 year) and adding refresh tokens.
- Rotate static API tokens periodically and consider additional transport security (mTLS) for sensitive traffic.

Examples (cURL)
Enroll:
```
curl -X POST 'https://api.pontaj.binarysquad.club/mobile/enroll' \
  -H 'Content-Type: application/json' \
  -d '{"codmatricol":"123456"}'
```

Request QR (base64):
```
curl -X POST 'https://api.pontaj.binarysquad.club/mobile/qr' \
  -H 'Authorization: Bearer <ENROLL_JWT>'
```

Request QR image (save to file):
```
curl -X POST 'https://api.pontaj.binarysquad.club/mobile/qr_image' \
  -H 'Authorization: Bearer <ENROLL_JWT>' --output qr.png
```

Running locally (short)
1. Create `.env` with DB and secret values: MYSQL_*, SECRET_KEY, API_TOKEN_*.
2. Create venv, install dependencies, and run uvicorn (see README for steps).
3. Or run with Docker Compose: `docker compose up --build`.

Appendix — code locations
- app/main.py, app/routers/mobile.py, app/routers/frontend.py, app/routers/admin_elevi.py, app/core/config.py, app/core/security.py

Next steps (optional)
- Pull the mobile repo (if public) and extract exact usage snippets to include here.
- Create a Postman collection or OpenAPI examples from the live API.

---

If anything should remain in Romanian-only, or you want fuller bilingual sections, say so and the doc will be adjusted.
