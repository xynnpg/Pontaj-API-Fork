# 🚀 Ghid de Deployment / Deployment Guide

[← Înapoi la index / Back to docs index](README.md)

---

## 🇷🇴 Introducere

Pontaj API poate fi rulat în trei moduri: local cu Python (pentru dezvoltare), cu Docker Compose (recomandat) sau în producție cu scriptul `deploy.sh`. Toate modurile necesită un fișier `.env` cu variabilele de mediu configurate.

## 🇬🇧 Introduction

Pontaj API can be run in three ways: locally with Python (for development), with Docker Compose (recommended), or in production using the `deploy.sh` script. All modes require a `.env` file with the environment variables configured.

> Variabile de mediu necesare → [environment.md](environment.md)  
> Required environment variables → [environment.md](environment.md)

---

## 📋 Cerințe prealabile / Prerequisites

### 🇷🇴
- **Python 3.12+** — pentru rulare locală
- **Docker + Docker Compose** — pentru rulare containerizată
- **Instanță MySQL 8+** — baza de date (locală sau remote)
- **Git** — pentru clonarea repo-ului

### 🇬🇧
- **Python 3.12+** — for local development
- **Docker + Docker Compose** — for containerized deployment
- **MySQL 8+ instance** — the database (local or remote)
- **Git** — for cloning the repository

---

## 💻 Rulare locală / Local Development

### 🇷🇴

```bash
# 1. Clonează repo-ul
git clone https://github.com/xynnpg/Pontaj-API-Fork.git
cd Pontaj-API-Fork

# 2. Creează și activează un mediu virtual
python -m venv .venv
source .venv/bin/activate        # Linux / macOS
# .venv\Scripts\activate         # Windows

# 3. Instalează dependențele
pip install -r app/requirements.txt

# 4. Creează fișierul .env (vezi docs/environment.md)
cp .env.example .env   # sau creează manual

# 5. Pornește serverul de dezvoltare
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

Serverul va fi disponibil la `http://localhost:8000`.  
Swagger UI: `http://localhost:8000/docs`

### 🇬🇧

```bash
# 1. Clone the repository
git clone https://github.com/xynnpg/Pontaj-API-Fork.git
cd Pontaj-API-Fork

# 2. Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate        # Linux / macOS
# .venv\Scripts\activate         # Windows

# 3. Install dependencies
pip install -r app/requirements.txt

# 4. Create the .env file (see docs/environment.md)
cp .env.example .env   # or create manually

# 5. Start the development server
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

Server available at `http://localhost:8000`.  
Swagger UI: `http://localhost:8000/docs`

---

## 🐳 Docker Compose

### 🇷🇴

Modul recomandat pentru rulare. Containerul folosește Gunicorn cu 4 UvicornWorker-i.

```bash
# Construiește și pornește
docker compose up --build

# Rulare în fundal (detached)
docker compose up --build -d

# Oprire
docker compose down

# Vizualizare log-uri
docker compose logs -f api
```

**Ce face `docker-compose.yml`:**
- Construiește imaginea din `Dockerfile` (Python 3.12-slim, user non-root)
- Încarcă variabilele din `.env`
- Expune portul `8000`
- Configurează un health check pe `/health/live` la fiecare 5 secunde

### 🇬🇧

The recommended way to run the project. The container uses Gunicorn with 4 UvicornWorkers.

```bash
# Build and start
docker compose up --build

# Run in background (detached)
docker compose up --build -d

# Stop
docker compose down

# View logs
docker compose logs -f api
```

**What `docker-compose.yml` does:**
- Builds the image from `Dockerfile` (Python 3.12-slim, non-root user)
- Loads variables from `.env`
- Exposes port `8000`
- Configures a health check on `/health/live` every 5 seconds

---

## ⚙️ Deploy în producție / Production Deploy

### 🇷🇴

Scriptul `deploy.sh` automatizează actualizarea în producție:

```bash
# Asigură-te că scriptul este executabil
chmod +x deploy.sh

# Rulează deploy-ul
./deploy.sh
```

**Ce face `deploy.sh`:**
```bash
git pull                  # Trage ultimele modificări din repo
docker compose down       # Oprește containerele existente
docker compose build      # Reconstruiește imaginea
docker compose up -d      # Pornește în fundal
```

### 🇬🇧

The `deploy.sh` script automates production updates:

```bash
# Ensure the script is executable
chmod +x deploy.sh

# Run the deploy
./deploy.sh
```

**What `deploy.sh` does:**
```bash
git pull                  # Pull latest changes from repo
docker compose down       # Stop existing containers
docker compose build      # Rebuild the image
docker compose up -d      # Start in background
```

---

## 🏥 Health Checks

### 🇷🇴

Două endpoint-uri pentru monitorizare și orchestrare:

| Endpoint | Descriere | Răspuns OK | Răspuns Eroare |
|---|---|---|---|
| `GET /health/live` | Serverul rulează | `200 {"status":"alive"}` | — |
| `GET /health/ready` | Server + DB disponibile | `200 {"status":"ready","dependencies":{"mysql":"ok"}}` | `503 {"mysql":"down"}` |

```bash
# Verificare rapidă
curl https://api.pontaj.binarysquad.club/health/live
curl https://api.pontaj.binarysquad.club/health/ready
```

### 🇬🇧

Two endpoints for monitoring and orchestration:

| Endpoint | Description | OK Response | Error Response |
|---|---|---|---|
| `GET /health/live` | Server is running | `200 {"status":"alive"}` | — |
| `GET /health/ready` | Server + DB available | `200 {"status":"ready","dependencies":{"mysql":"ok"}}` | `503 {"mysql":"down"}` |

```bash
# Quick check
curl https://api.pontaj.binarysquad.club/health/live
curl https://api.pontaj.binarysquad.club/health/ready
```

---

## 🔗 Vezi și / See Also

- [environment.md](environment.md) — Toate variabilele de mediu necesare / All required environment variables
