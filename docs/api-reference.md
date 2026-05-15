# 📖 Referință API / API Reference

[← Înapoi la index / Back to docs index](README.md)

---

## 🇷🇴 Introducere

Documentație completă pentru toate endpoint-urile Pontaj API. Fiecare endpoint include metoda HTTP, calea, autentificarea necesară, corpul cererii, răspunsul și codurile de eroare posibile.

## 🇬🇧 Introduction

Complete documentation for all Pontaj API endpoints. Each endpoint includes the HTTP method, path, required authentication, request body, response, and possible error codes.

---

**Base URL:** `https://api.pontaj.binarysquad.club`  
**Swagger UI:** [api.pontaj.binarysquad.club/docs](https://api.pontaj.binarysquad.club/docs)

### Tipuri de autentificare / Authentication types

| Tip / Type | Cum se trimite / How to send | Descriere (RO) | Description (EN) |
|---|---|---|---|
| Static Bearer | `Authorization: Bearer <API_TOKEN_*>` | Token static din `.env` | Static token from `.env` |
| Enroll JWT | `Authorization: Bearer <access_token>` | JWT emis la `POST /mobile/enroll` | JWT issued at `POST /mobile/enroll` |
| Admin JWT | `Authorization: Bearer <access_token>` | JWT emis la `POST /admin/login` | JWT issued at `POST /admin/login` |
| QR JWT | `Authorization: Bearer <qr_token>` | JWT de 30s din codul QR | 30s JWT from the QR code |

---

## 🏥 Health

### `GET /health/live` — Liveness probe

**🇷🇴** Verifică dacă serverul rulează. Fără autentificare.  
**🇬🇧** Checks if the server is running. No authentication.

| | |
|---|---|
| Auth | Niciuna / None |
| Response 200 | `{"status": "alive"}` |

---

### `GET /health/ready` — Readiness probe

**🇷🇴** Verifică dacă serverul și baza de date sunt disponibile.  
**🇬🇧** Checks if the server and database are available.

| | |
|---|---|
| Auth | Niciuna / None |
| Response 200 | `{"status": "ready", "dependencies": {"mysql": "ok"}}` |
| Response 503 | `{"mysql": "down", "error": "..."}` |

---

## 📱 Mobile (`/mobile`)

### `GET /mobile/status`

**🇷🇴** Verifică că clientul mobil este autentificat corect.  
**🇬🇧** Verifies the mobile client is correctly authenticated.

| | |
|---|---|
| Auth | Static Bearer (`API_TOKEN_MOBILE`) |
| Response 200 | `{"client": "mobile", "status": "ok"}` |
| Response 401 | Token invalid sau lipsă / Invalid or missing token |

---

### `POST /mobile/enroll`

**🇷🇴** Înregistrează un elev în aplicație folosind codul matricol. Emite un JWT de 1 an.  
**🇬🇧** Enrolls a student in the app using their matriculation code. Issues a 1-year JWT.

| | |
|---|---|
| Auth | Niciuna / None |
| Body | `{"codmatricol": "123456"}` |
| Response 200 | `{"codmatricol": "...", "access_token": "...", "token_type": "bearer", "name": "...", "expires_in": 31536000}` |
| Response 401 | Cod matricol inexistent / Matriculation code not found |
| Response 409 | Elevul este deja înregistrat / Student already enrolled |

---

### `GET /mobile/verifyToken`

**🇷🇴** Verifică dacă token-ul de înregistrare este valid și returnează statusul elevului.  
**🇬🇧** Verifies the enroll token is valid and returns the student's status.

| | |
|---|---|
| Auth | Enroll JWT |
| Response 200 | `{"Activ": 1, "Name": "Ion Popescu"}` |
| Response 401 | Token invalid sau expirat / Invalid or expired token |

---

### `POST /mobile/qr`

**🇷🇴** Generează un cod QR cu un JWT de 30 de secunde. Returnează imaginea QR ca base64 PNG.  
**🇬🇧** Generates a QR code with a 30-second JWT. Returns the QR image as base64 PNG.

| | |
|---|---|
| Auth | Enroll JWT |
| Response 200 | `{"expires_in": 30, "token": "...", "qr_png_base64": "...", "exp": 1234567890}` |
| Response 401 | Token invalid sau utilizator negăsit / Invalid token or user not found |

---

### `POST /mobile/qr_image`

**🇷🇴** Identic cu `/mobile/qr`, dar returnează direct imaginea PNG (nu base64).  
**🇬🇧** Same as `/mobile/qr`, but returns the raw PNG image (not base64).

| | |
|---|---|
| Auth | Enroll JWT |
| Response 200 | `image/png` (binary) |
| Response 401 | Token invalid / Invalid token |

---

### `GET /mobile/scans?start=&end=`

**🇷🇴** Returnează toate scanările din sistem într-un interval de timp.  
**🇬🇧** Returns all scans in the system within a time range.

| | |
|---|---|
| Auth | Enroll JWT |
| Query params | `start` (ISO 8601), `end` (ISO 8601) |
| Response 200 | `{"count": 5, "start": "...", "end": "...", "data": [...]}` |
| Response 400 | `start` ≥ `end` |

---

### `GET /mobile/enrolled_student_scans?start=&end=`

**🇷🇴** Returnează scanările elevului autentificat într-un interval de timp.  
**🇬🇧** Returns the authenticated student's own scans within a time range.

| | |
|---|---|
| Auth | Enroll JWT |
| Query params | `start` (ISO 8601), `end` (ISO 8601) |
| Response 200 | `{"count": 3, "start": "...", "end": "...", "data": [{"scan_time": "...", "name": "..."}]}` |
| Response 400 | `start` ≥ `end` |

---

## 🔧 Admin (`/admin`)

### `POST /admin/login`

**🇷🇴** Autentifică un profesor cu email și parolă. Emite un JWT de 1 oră.  
**🇬🇧** Authenticates a teacher with email and password. Issues a 1-hour JWT.

| | |
|---|---|
| Auth | Niciuna / None |
| Body | `{"username": "email@exemplu.com", "password": "parola"}` |
| Response 200 | `{"username": "...", "access_token": "...", "token_type": "bearer", "expires_in": 3600}` |
| Response 401 | Credențiale invalide / Invalid credentials |

---

### `GET /admin/profesori`

**🇷🇴** Returnează lista tuturor profesorilor (fără parole).  
**🇬🇧** Returns the list of all teachers (without passwords).

| | |
|---|---|
| Auth | Admin JWT |
| Response 200 | `{"requested_by": "...", "count": 5, "admins": [{"ID": 1, "Name": "...", "Email": "...", "Admin": 1}]}` |

---

### `POST /admin/profesori`

**🇷🇴** Adaugă un profesor nou. Parola este hash-uită automat cu bcrypt.  
**🇬🇧** Adds a new teacher. Password is automatically hashed with bcrypt.

| | |
|---|---|
| Auth | Admin JWT |
| Body | `{"nume": "...", "email": "...", "password": "...", "admin": 0}` |
| Response 201 | `{"ID": 5, "Name": "...", "Email": "...", "Admin": 0}` |
| Response 409 | Email duplicat / Duplicate email |

---

### `PUT /admin/profesori/{id}`

**🇷🇴** Actualizează datele unui profesor (fără parolă).  
**🇬🇧** Updates a teacher's data (excluding password).

| | |
|---|---|
| Auth | Admin JWT |
| Body | `{"nume": "...", "email": "...", "admin": 0}` |
| Response 200 | `{"ID": 5, "Name": "...", "Email": "...", "Admin": 0}` |
| Response 404 | Profesor negăsit / Teacher not found |

---

### `PUT /admin/changepassword/{id}`

**🇷🇴** Resetează parola unui profesor.  
**🇬🇧** Resets a teacher's password.

| | |
|---|---|
| Auth | Admin JWT |
| Body | `{"password": "noua_parola"}` |
| Response 200 | `{"ID": 5, "Name": "...", "Email": "...", "Admin": 0}` |
| Response 404 | Profesor negăsit / Teacher not found |

---

### `DELETE /admin/profesori/{id}`

**🇷🇴** Șterge permanent un profesor.  
**🇬🇧** Permanently deletes a teacher.

| | |
|---|---|
| Auth | Admin JWT |
| Response 204 | Șters cu succes / Successfully deleted |
| Response 404 | Profesor negăsit / Teacher not found |
| Response 409 | Constrângere FK / FK constraint violation |

---

### `GET /admin/elevi?name=`

**🇷🇴** Returnează toți elevii. Filtrare opțională după nume.  
**🇬🇧** Returns all students. Optional filter by name.

| | |
|---|---|
| Auth | Admin JWT |
| Query params | `name` (opțional / optional, partial match) |
| Response 200 | `{"requested_by": "...", "count": 30, "elevi": [...]}` |

---

### `GET /admin/elevi_enrolled`

**🇷🇴** Returnează doar elevii care s-au înregistrat în aplicație (Token ≠ NULL).  
**🇬🇧** Returns only students who have enrolled in the app (Token ≠ NULL).

| | |
|---|---|
| Auth | Admin JWT |
| Response 200 | `{"requested_by": "...", "count": 20, "elevi": [...]}` |

---

### `POST /admin/elevi`

**🇷🇴** Adaugă un elev nou în sistem (fără înregistrare — Token rămâne NULL).  
**🇬🇧** Adds a new student to the system (without enrollment — Token stays NULL).

| | |
|---|---|
| Auth | Admin JWT |
| Body | `{"nume": "...", "email": "...", "codmatricol": "123456", "activ": 1}` |
| Response 201 | `{"ID": 10, "Name": "...", "Email": "...", "CodMatricol": "...", "Activ": 1, "DataActivare": null}` |
| Response 409 | CodMatricol duplicat / Duplicate matriculation code |

---

### `PUT /admin/elevi/{id}`

**🇷🇴** Actualizează datele unui elev.  
**🇬🇧** Updates a student's data.

| | |
|---|---|
| Auth | Admin JWT |
| Body | `{"nume": "...", "email": "...", "codmatricol": "...", "activ": 1}` |
| Response 200 | `{"ID": 10, "Name": "...", "Email": "...", "CodMatricol": "...", "Activ": 1}` |
| Response 404 | Elev negăsit / Student not found |

---

### `DELETE /admin/elevi/{id}`

**🇷🇴** Șterge permanent un elev.  
**🇬🇧** Permanently deletes a student.

| | |
|---|---|
| Auth | Admin JWT |
| Response 204 | Șters cu succes / Successfully deleted |
| Response 404 | Elev negăsit / Student not found |

---

### `GET /admin/scans_by_date?start=&end=`

**🇷🇴** Returnează toate scanările din sistem într-un interval de timp.  
**🇬🇧** Returns all scans in the system within a time range.

| | |
|---|---|
| Auth | Admin JWT |
| Query params | `start` (ISO 8601), `end` (ISO 8601) |
| Response 200 | `{"count": 10, "start": "...", "end": "...", "data": [{"id_elev": 1, "scan_time": "...", "token": "...", "name": "..."}]}` |

---

## 🖥️ Frontend (`/frontend`)

### `GET /frontend/status`

**🇷🇴** Verifică că clientul frontend este autentificat corect.  
**🇬🇧** Verifies the frontend client is correctly authenticated.

| | |
|---|---|
| Auth | Static Bearer (`API_TOKEN_FRONTEND`) |
| Response 200 | `{"client": "frontend", "status": "ok"}` |

---

### `GET /frontend/items`

**🇷🇴** Endpoint de test — returnează o valoare din baza de date.  
**🇬🇧** Test endpoint — returns a value from the database.

| | |
|---|---|
| Auth | Static Bearer (`API_TOKEN_FRONTEND`) |
| Response 200 | `{"items": [{"val": 42}]}` |

---

### `POST /frontend/scan`

**🇷🇴** Procesează un cod QR scanat. Verifică JWT-ul, înregistrează prezența și returnează datele elevului.  
**🇬🇧** Processes a scanned QR code. Verifies the JWT, records attendance, and returns student data.

| | |
|---|---|
| Auth | QR JWT (token-ul din codul QR / token from the QR code) |
| Response 200 | `{"Activ": 1, "Name": "Ion Popescu"}` |
| Response 401 | JWT expirat sau invalid / Expired or invalid JWT |
| Response 409 | Cod QR deja folosit / QR code already used |
| Response 429 | Elevul a fost scanat în ultima oră / Student scanned in the last hour |

---

## 🧪 Exemple cURL / cURL Examples

### Înregistrare elev / Enroll student
```bash
curl -X POST 'https://api.pontaj.binarysquad.club/mobile/enroll' \
  -H 'Content-Type: application/json' \
  -d '{"codmatricol": "123456"}'
```

### Generare QR (base64) / Generate QR (base64)
```bash
curl -X POST 'https://api.pontaj.binarysquad.club/mobile/qr' \
  -H 'Authorization: Bearer <ENROLL_JWT>'
```

### Generare QR (imagine PNG) / Generate QR (PNG image)
```bash
curl -X POST 'https://api.pontaj.binarysquad.club/mobile/qr_image' \
  -H 'Authorization: Bearer <ENROLL_JWT>' \
  --output qr.png
```

### Scanare QR / Scan QR
```bash
curl -X POST 'https://api.pontaj.binarysquad.club/frontend/scan' \
  -H 'Authorization: Bearer <QR_JWT>'
```

### Login admin / Admin login
```bash
curl -X POST 'https://api.pontaj.binarysquad.club/admin/login' \
  -H 'Content-Type: application/json' \
  -d '{"username": "admin@scoala.ro", "password": "parola"}'
```

---

## 🔗 Vezi și / See Also

- [auth-flow.md](auth-flow.md) — Cum să obții fiecare tip de token / How to obtain each token type
- [qr-flow.md](qr-flow.md) — Fluxul complet de prezență QR / Full QR presence flow
