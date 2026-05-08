PONTAJ - Integration and API Documentation (English / Română)

Table of Contents
- Executive summary (EN / RO)
- Links
- Architecture overview
- Authentication model
- API Endpoints (detailed)
- Database notes (elevi table)
- Mobile integration guide (step-by-step)
- Running the server locally / Docker
- Troubleshooting & common errors
- Security notes & recommendations
- Appendix: source references

---

Executive summary (English)
This document describes the Pontaj API (https://api.pontaj.binarysquad.club/), the API docs page (https://api.pontaj.binarysquad.club/docs), and how the server repository (https://github.com/xynnpg/ServerAppPontaj) integrates with the mobile client (https://github.com/some-randomGuy03/pontaj_mobile). It contains endpoint specs, authentication flows (JWT + static bearer tokens), sample requests/responses, DB schema hints, and a step-by-step integration guide for mobile developers.

Rezumat (Română)
Acest document descrie API-ul Pontaj (https://api.pontaj.binarysquad.club/), pagina de documentație (https://api.pontaj.binarysquad.club/docs) și modul în care repo-ul server (https://github.com/xynnpg/ServerAppPontaj) se integrează cu clientul mobil (https://github.com/some-randomGuy03/pontaj_mobile). Include specificații de endpointuri, fluxuri de autentificare (JWT + token static Bearer), exemple request/response, informații despre schema DB și un ghid pas-cu-pas pentru integrarea din partea mobilă.

Links
- API base: https://api.pontaj.binarysquad.club/
- API docs (Swagger/OpenAPI): https://api.pontaj.binarysquad.club/docs
- Server repo: https://github.com/xynnpg/ServerAppPontaj
- Mobile repo: https://github.com/some-randomGuy03/pontaj_mobile

Architecture overview (EN)
- Server: FastAPI application (app/main.py) exposing multiple routers: /frontend, /mobile, /admin and health endpoints.
- DB: MySQL (async via asyncmy + SQLAlchemy asyncio engine).
- Clients: frontend (web admin), mobile app. The server uses both per-client static bearer tokens for some routes and JWT tokens for user enrollment and short-lived QR tokens.

Prezentare arhitectură (RO)
- Server: aplicație FastAPI (app/main.py) cu routere: /frontend, /mobile, /admin și endpoint-uri health.
- DB: MySQL (async cu asyncmy și SQLAlchemy asyncio).
- Clienți: frontend (web/admin), aplicație mobilă. Serverul folosește tokenuri Bearer statice pentru anumite rute și JWT pentru înrolare și tokenuri scurte (QR).

Authentication model / Model de autentificare

English
- Static client tokens: the app defines per-client static tokens loaded from environment variables in app/core/config.py:
  - API_TOKEN_FRONTEND
  - API_TOKEN_ADMIN
  - API_TOKEN_MOBILE
  These are validated with make_bearer_verifier(expected_token) (app/core/security.py) and used as dependencies for routes that include Depends(verify_xxx).

- JWT user token (enroll): the mobile enroll flow issues a JWT using SECRET_KEY (settings.SECRET_KEY) as a per-user enroll token. This token is stored in the elevi.Token DB column and used by mobile to access protected user endpoints and to produce short-lived QR tokens.

- Short-lived QR JWT: the /mobile/qr and /mobile/qr_image endpoints create a short-lived JWT (30 seconds in code) containing a DB value (ID). That token is encoded with SECRET_KEY and returned as a QR (base64 png or raw png). The QR token should be validated by whatever scans it (frontend or another server) with the same SECRET_KEY.

Română
- Token-uri statice per client: variabilele din mediu API_TOKEN_FRONTEND, API_TOKEN_ADMIN, API_TOKEN_MOBILE (app/core/config.py). Validatorul se obține cu make_bearer_verifier și este folosit ca dependency pentru anumite rute.

- Token JWT de utilizator (înrolare): fluxul de înrolare generează un JWT semnat cu SECRET_KEY și-l salvează în coloana elevi.Token. Acest token (access_token) este folosit de aplicația mobilă pentru operații protejate și pentru generarea de tokenuri QR efemere.

- QR JWT scurt: endpoint-urile /mobile/qr și /mobile/qr_image generează un JWT pe 30 sec cu valoarea DB (ID). Tokenul e returnat ca imagine QR. Scanning side trebuie să decripteze și să verifice tokenul cu același SECRET_KEY.

Important implementation notes (observed behavior)
- mobile_status (GET /mobile/status) requires the static API_TOKEN_MOBILE bearer token.
- /mobile/enroll (POST) is not decorated with make_bearer_verifier in the code and therefore appears callable without the static API_TOKEN_MOBILE. It accepts JSON body {"codmatricol": "..."} and, if successful, creates a long-lived JWT (1 year) that is stored in elevi.Token and returned as access_token.
- QR endpoints and verifyToken use JWT verification with SECRET_KEY (not the per-client static token).
- admin_elevi router defines verify_elevi = make_bearer_verifier(settings.API_TOKEN_ADMIN) but the CRUD routes use the JWT verify_jwt_token dependency; this may be an implementation inconsistency—review intended admin auth model.

API Endpoints (detailed)

1) Health
- GET /health/live
  - Description: liveness probe
  - Auth: none
  - Response 200: {"status": "alive"}

- GET /health/ready
  - Description: readiness probe; checks DB via db_ping
  - Auth: none
  - Response 200: {"status": "ready", "dependencies": {"mysql": "ok"}}
  - Response 503: {"mysql": "down", "error": "..."}

2) Mobile-related
- POST /mobile/enroll
  - Description: Enroll a student by codmatricol. Creates and stores a long-lived JWT (1 year) in DB.elevi.Token
  - Auth: none (code has no static bearer dependency)
  - Request JSON: {"codmatricol": "<string>"}
  - Success 200: {
      "codmatricol": "...",
      "access_token": "<jwt>",
      "token_type": "bearer",
      "name": "Name",
      "expires_in": 31536000
    }
  - Errors: 401 Invalid credentials, 409 User already enrolled
  - Notes: mobile should store access_token securely (keychain/keystore)

- GET /mobile/status
  - Description: simple status, requires static API_TOKEN_MOBILE using make_bearer_verifier
  - Headers: Authorization: Bearer <API_TOKEN_MOBILE>
  - Response: {"client": "mobile", "status": "ok"}

- GET /mobile/verifyToken
  - Description: verifies the enroll JWT and returns student name and active flag, compares DB Token field to provided credential
  - Headers: Authorization: Bearer <ENROLL_JWT>
  - Response 200: {"Activ": <int>, "Name": "..."}
  - Errors: 401 token expired/invalid

- POST /mobile/qr (returns base64 PNG and token)
  - Headers: Authorization: Bearer <ENROLL_JWT>
  - Response: {
      "expires_in": 30,
      "token": "<short_lived_jwt>",
      "qr_png_base64": "...",
      "exp": <unix ts>
    }

- POST /mobile/qr_image (returns image/png)
  - Headers: Authorization: Bearer <ENROLL_JWT>
  - Response: raw PNG with Cache-Control: no-store

3) Frontend (web)
- GET /frontend/status
  - Auth: static API_TOKEN_FRONTEND (Authorization: Bearer <API_TOKEN_FRONTEND>)
  - Response: {"client": "frontend", "status": "ok"}

- GET /frontend/items
  - Auth: static API_TOKEN_FRONTEND
  - Response sample: {"items": [{"val": 42}]}

4) Admin / elevi (protected by JWT verify_jwt_token currently)
- GET /admin/elevi?name=...
  - Auth: Authorization: Bearer <JWT>
  - Response: {"requested_by": <sub>, "count": N, "elevi": [...]}

- POST /admin/elevi
  - Body: {"nume": "", "email": "", "codmatricol": "", "activ": 1}
  - Response: Created ElevOut

- PUT /admin/elevi/{id}
  - Body: same as create
  - Response: updated ElevOut

- DELETE /admin/elevi/{id}
  - Response: 204 No Content on success

Database notes (elevi table) / Observații DB
Observed columns used in queries across the code:
- ID (int primary key)
- Token (string) — stores enroll JWT
- Email
- Name
- Activ (int) — 0/1 active flag
- CodMatricol (string) — enrollment code
- DataActivare (datetime) — activation date

Sample SQL snippets (from code):
- SELECT ID, Token, Email, Name, Activ FROM elevi WHERE CodMatricol = :u LIMIT 1;
- UPDATE elevi SET Token = :token, DataActivare = :dataactivare, Activ = :activ WHERE ID = :id
- SELECT ID FROM elevi WHERE Token = :tk LIMIT 1;

Mobile integration guide (step-by-step) / Ghid integrare mobil

English
A. Enrollment (first run):
1. User enters CodMatricol in mobile app.
2. Mobile POSTs to POST https://api.pontaj.binarysquad.club/mobile/enroll with JSON {"codmatricol":"<code>"}.
3. On success, server returns {access_token, expires_in, name}. Store access_token securely on device (Keychain/Keystore).

B. Using the enroll token:
1. For user-specific actions, include header Authorization: Bearer <access_token> (the one returned by enroll).
2. To request a QR for presence scanning, call POST /mobile/qr or /mobile/qr_image with that Authorization header. The response contains a short-lived JWT embedded in the QR.

C. Verifying QR on scanning side (e.g., frontend or separate validator):
1. Scan QR and extract token (short_lived_jwt).
2. Verify the JWT using SECRET_KEY (must be shared by verifier) and check exp claim.
3. The JWT payload includes: {"value": <db id>, "iat":..., "exp":...}. Use value to look up the student record in DB.

D. Token lifetime and refresh considerations:
- Enroll token in code is issued with exp = now + 31536000 (1 year). Consider implementing refresh or short-lived tokens with refresh flows for improved security.

Română
A. Înrolare:
1. Utilizatorul introduce CodMatricol în aplicație.
2. Mobile face POST la https://api.pontaj.binarysquad.club/mobile/enroll cu JSON {"codmatricol":"<cod>"}.
3. Dacă are succes, serverul returnează access_token; stocați-l securizat pe dispozitiv (Keychain/Keystore).

B. Utilizarea tokenului:
1. Pentru acțiuni specifice utilizatorului, trimiteți header Authorization: Bearer <access_token>.
2. Pentru generarea QR, apelați POST /mobile/qr sau /mobile/qr_image cu acel Authorization. Răspunsul conține un JWT efemer inclus în QR.

C. Validarea QR la scanning (frontend/validator):
1. Scanați QR și extrageți tokenul.
2. Verificați JWT cu SECRET_KEY și exp.
3. Payload conține {"value": <id DB>, ...} — folosiți value pentru a identifica utilizatorul.

D. Durata tokenului și reîmprospătare:
- Tokenul de înrolare are expirare 1 an în cod. Luați în considerare implementarea unui mecanism de refresh sau scurtarea duratei pentru securitate îmbunătățită.

Examples (cURL)

1) Enroll (EN / RO)
curl -X POST 'https://api.pontaj.binarysquad.club/mobile/enroll' \
  -H 'Content-Type: application/json' \
  -d '{"codmatricol":"123456"}'

Response (200):
{
  "codmatricol": "123456",
  "access_token": "<jwt>",
  "token_type": "bearer",
  "name": "John Doe",
  "expires_in": 31536000
}

2) Request QR (returns base64 PNG)
curl -X POST 'https://api.pontaj.binarysquad.club/mobile/qr' \
  -H 'Authorization: Bearer <ENROLL_JWT>'

Response (200): {"expires_in":30, "token":"<short_jwt>", "qr_png_base64":"...", "exp": 168...}

3) Request QR image (raw png)
curl -X POST 'https://api.pontaj.binarysquad.club/mobile/qr_image' \
  -H 'Authorization: Bearer <ENROLL_JWT>' --output qr.png

Running server locally / Rulare locală

English
1. Create .env with required values (example):
   MYSQL_USER=root
   MYSQL_PASSWORD=secret
   MYSQL_HOST=127.0.0.1
   MYSQL_PORT=3306
   MYSQL_DB=pontaj
   SECRET_KEY=<supersecret>
   API_TOKEN_FRONTEND=<token>
   API_TOKEN_ADMIN=<token>
   API_TOKEN_MOBILE=<token>

2. Install deps and run (from project root):
   python -m venv .venv
   .venv\Scripts\activate
   pip install -r app/requirements.txt
   uvicorn app.main:app --host 0.0.0.0 --port 8000

3. Or use Docker (project contains Dockerfile and docker-compose.yml):
   docker compose up --build

Română
1. Creați fișierul .env cu variabilele necesare (exemplu de mai sus).
2. Instalare și pornire local:
   python -m venv .venv
   .venv\Scripts\activate
   pip install -r app/requirements.txt
   uvicorn app.main:app --host 0.0.0.0 --port 8000
3. Sau Docker:
   docker compose up --build

Troubleshooting & common errors / Depanare
- 401 Invalid credentials: codmatricol incorect sau utilizatorul deja înrolat.
- 409 User already enrolled: elev.Token is not null; cannot enroll again.
- 503 readiness: DB not reachable — check DATABASE_URL and MySQL availability.
- JWT errors: ensure SECRET_KEY is identical wherever tokens are verified.

Security recommendations / Recomandări securitate
- Do NOT commit SECRET_KEY or API tokens to VCS.
- Store mobile tokens using secure platform storage (Keychain/Keystore).
- Consider shortening enroll token lifetime and implement token refresh.
- Consider rotating static API tokens and using mutual TLS for sensitive traffic.

Appendix: source references / Referințe cod
- app/main.py — routers and CORS config
- app/routers/mobile.py — enroll, qr, verifyToken, mobile flows
- app/routers/frontend.py — frontend routes
- app/routers/admin_elevi.py — admin CRUD for elevi
- app/core/config.py — env vars and DATABASE_URL construction
- app/core/security.py — make_bearer_verifier

Notes about mobile repository
- This document assumes the mobile app will call the endpoints as described above. If the mobile repo (https://github.com/some-randomGuy03/pontaj_mobile) is public, review its source to extract exact API call examples and embed snippets; if private, request access or provide sample code to include here.

---

If you want, next steps I can do:
- Pull the mobile repo (if public) and extract exact usage snippets to insert.
- Turn these examples into Postman collection or OpenAPI examples.

End of document.
