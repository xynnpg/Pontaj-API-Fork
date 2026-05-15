# 🏗️ Arhitectura Sistemului / System Architecture

[← Înapoi la index / Back to docs index](README.md)

---

## 🇷🇴 Prezentare generală

**Pontaj API** rezolvă problema înregistrării prezenței elevilor în timp real. Un elev se înscrie o singură dată cu codul său matricol, primind un JWT de lungă durată. La fiecare oră de curs, elevul generează un cod QR din aplicația mobilă — profesorul sau panoul web scanează codul, iar prezența este înregistrată automat în baza de date.

## 🇬🇧 Overview

**Pontaj API** solves the problem of real-time student attendance tracking. A student enrolls once using their matriculation code, receiving a long-lived JWT. At each class, the student generates a QR code from the mobile app — the teacher or web frontend scans it, and attendance is automatically recorded in the database.

---

## 🗂️ Stack Tehnologic / Tech Stack

| Componentă / Component | Tehnologie / Technology | Versiune / Version |
|---|---|---|
| Framework API | FastAPI | 0.115 |
| Server ASGI | Uvicorn + Gunicorn | 0.32 / 23.0 |
| Bază de date / Database | MySQL | 8+ |
| Driver async DB | asyncmy + SQLAlchemy async | 0.2.9 / 2.0 |
| Autentificare / Auth | PyJWT + bcrypt (passlib) | latest |
| Generare QR / QR generation | qrcode + Pillow | latest |
| Containerizare / Containerization | Docker + Docker Compose | 3.8 |
| Limbaj / Language | Python | 3.12 |

---

## 📐 Diagrama de ansamblu / High-Level Diagram

```mermaid
graph TD
    MA[📱 Mobile App<br/>Aplicație mobilă] -->|JWT enroll| MOB[/mobile]
    WF[🖥️ Web Frontend<br/>Panou web] -->|API_TOKEN_FRONTEND| FE[/frontend]
    AP[🔧 Admin Panel<br/>Panou admin] -->|JWT admin| ADM[/admin]

    MOB --> API[⚡ FastAPI Server]
    FE --> API
    ADM --> API

    API -->|asyncmy / SQLAlchemy| DB[(🗄️ MySQL Database)]
    API --> HC[/health]

    HC -->|liveness| LIVE[GET /health/live]
    HC -->|readiness| READY[GET /health/ready]

    style API fill:#009688,color:#fff
    style DB fill:#2496ED,color:#fff
```

---

## 🔌 Componente / Components

### 🇷🇴 Router `/mobile`
Gestionează tot ciclul de viață al elevului: înregistrare cu cod matricol, verificare token, generare QR pentru prezență și interogare istorică a scanărilor. Autentificarea se face prin JWT de lungă durată emis la înscriere.

### 🇬🇧 `/mobile` Router
Handles the full student lifecycle: enrollment via matriculation code, token verification, QR generation for presence, and historical scan queries. Authentication uses the long-lived JWT issued at enrollment.

---

### 🇷🇴 Router `/admin`
Oferă autentificare pentru profesori (bcrypt + JWT de 1 oră) și operații CRUD complete pentru elevi și profesori. Include și interogarea scanărilor pe intervale de timp.

### 🇬🇧 `/admin` Router
Provides teacher authentication (bcrypt + 1-hour JWT) and full CRUD operations for students and teachers. Also includes scan queries by date range.

---

### 🇷🇴 Router `/frontend`
Expune endpoint-ul de scanare QR folosit de panoul web. Primește JWT-ul de scurtă durată din codul QR, verifică validitatea, protejează împotriva replay-ului și înregistrează prezența în tabelul `scanari`.

### 🇬🇧 `/frontend` Router
Exposes the QR scan endpoint used by the web panel. Receives the short-lived JWT from the QR code, verifies validity, protects against replay attacks, and records attendance in the `scanari` table.

---

### 🇷🇴 Endpoint-uri `/health`
Două probe pentru orchestrare: `/health/live` returnează imediat `{"status":"alive"}`, iar `/health/ready` verifică conectivitatea la baza de date înainte de a răspunde.

### 🇬🇧 `/health` Endpoints
Two probes for orchestration: `/health/live` returns `{"status":"alive"}` immediately, and `/health/ready` checks database connectivity before responding.

---

## 🔗 Vezi și / See Also

- [auth-flow.md](auth-flow.md) — Detalii despre autentificare / Authentication details
- [database.md](database.md) — Schema bazei de date / Database schema
- [deployment.md](deployment.md) — Ghid de deployment / Deployment guide
