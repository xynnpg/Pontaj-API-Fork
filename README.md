# Pontaj API

[![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=flat&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115-009688?style=flat&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Docker](https://img.shields.io/badge/Docker-ready-2496ED?style=flat&logo=docker&logoColor=white)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=flat)](LICENSE)
[![Live API](https://img.shields.io/badge/API-live-brightgreen?style=flat)](https://api.pontaj.binarysquad.club/health/live)

---

## Descriere

**Pontaj API** este un serviciu backend construit cu FastAPI, folosit de o aplicație mobilă și un panou web pentru gestionarea prezenței elevilor. Sistemul permite înscrierea elevilor prin cod matricol, generarea de token-uri QR cu durată scurtă de viață pentru verificarea prezenței și administrarea completă a elevilor și profesorilor.

## Description

**Pontaj API** is a FastAPI backend service used by a mobile app and a web frontend to manage student attendance. It handles student enrollment via matriculation code, generates short-lived QR tokens for presence scanning, and provides full CRUD administration for students and teachers.

---

## Funcționalități / Features

| Română | English |
|---|---|
| Înregistrare elevi cu cod matricol | Student enrollment via matriculation code |
| JWT de lungă durată pentru sesiuni mobile | Long-lived JWTs for mobile sessions |
| Token-uri QR cu expirare de 30 secunde | 30-second expiring QR tokens for presence |
| Protecție anti-replay și cooldown de 1 oră | Anti-replay protection + 1-hour scan cooldown |
| CRUD complet pentru elevi și profesori | Full CRUD for students and teachers |
| Autentificare admin cu bcrypt + JWT | Admin authentication with bcrypt + JWT |
| Token-uri statice per client (mobile/frontend/admin) | Per-client static bearer tokens |
| Health checks pentru orchestrare (liveness + readiness) | Liveness + readiness health check endpoints |
| Rulare cu Docker Compose | Docker Compose deployment |

---

## Quick Start

### Pornire rapidă

```bash
# 1. Clonează repo-ul
git clone https://github.com/xynnpg/Pontaj-API-Fork.git
cd Pontaj-API-Fork

# 2. Creează fișierul .env (vezi docs/environment.md pentru toate variabilele)
cp .env.example .env   # sau creează manual

# 3. Pornește cu Docker Compose
docker compose up --build
```

### Getting started

```bash
# 1. Clone the repo
git clone https://github.com/xynnpg/Pontaj-API-Fork.git
cd Pontaj-API-Fork

# 2. Create your .env file (see docs/environment.md for all variables)
cp .env.example .env   # or create manually

# 3. Start with Docker Compose
docker compose up --build
```

API disponibil / API available at: `http://localhost:8000`

> Pentru rulare locală fără Docker, vezi [docs/deployment.md](docs/deployment.md).  
> For local development without Docker, see [docs/deployment.md](docs/deployment.md).

---

## Documentație / Documentation

| Fișier / File | Descriere | Description |
|---|---|---|
| [docs/README.md](docs/README.md) | Index documentație | Documentation hub |
| [docs/architecture.md](docs/architecture.md) | Arhitectura sistemului + diagrame | System architecture + diagrams |
| [docs/auth-flow.md](docs/auth-flow.md) | Fluxuri de autentificare + JWT | Authentication flows + JWT |
| [docs/qr-flow.md](docs/qr-flow.md) | Fluxul de prezență QR | QR presence check flow |
| [docs/database.md](docs/database.md) | Schema bazei de date + ER | Database schema + ER diagram |
| [docs/api-reference.md](docs/api-reference.md) | Referință completă API | Full API endpoint reference |
| [docs/deployment.md](docs/deployment.md) | Ghid de deployment | Deployment guide |
| [docs/environment.md](docs/environment.md) | Variabile de mediu | Environment variables |

---

## Link-uri utile / Useful Links

- **Live API:** [api.pontaj.binarysquad.club](https://api.pontaj.binarysquad.club)
- **Swagger UI:** [api.pontaj.binarysquad.club/docs](https://api.pontaj.binarysquad.club/docs)
- **GitHub:** [github.com/xynnpg/Pontaj-API-Fork](https://github.com/xynnpg/Pontaj-API-Fork)

---

## Contribuții / Contributing

**RO** Pentru modificări de documentație, deschide un PR mic cu actualizări clare și incrementale. Pentru modificări de cod, descrie comportamentul în corpul PR-ului.

**EN** For documentation changes, open a small PR with clear, incremental updates. For code changes, describe the behavior in the PR body.
