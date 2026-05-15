# ⚙️ Variabile de Mediu / Environment Variables

[← Înapoi la index / Back to docs index](README.md)

---

## 🇷🇴 Introducere

Toate configurările sensibile ale aplicației sunt gestionate prin variabile de mediu, încărcate dintr-un fișier `.env` la pornire. Niciuna dintre aceste valori nu trebuie comisă în VCS.

## 🇬🇧 Introduction

All sensitive application configuration is managed through environment variables, loaded from a `.env` file at startup. None of these values should ever be committed to VCS.

---

## 📋 Referință completă / Full Reference

### Baza de date / Database

| Variabilă | Obligatorie / Required | Implicit / Default | Descriere (RO) | Description (EN) |
|---|---|---|---|---|
| `MYSQL_USER` | ✅ Da / Yes | `root` | Utilizatorul MySQL | MySQL username |
| `MYSQL_PASSWORD` | ✅ Da / Yes | _(gol / empty)_ | Parola MySQL | MySQL password |
| `MYSQL_HOST` | ✅ Da / Yes | `localhost` | Adresa serverului MySQL | MySQL server host |
| `MYSQL_PORT` | ⬜ Nu / No | `3306` | Portul MySQL | MySQL port |
| `MYSQL_DB` | ✅ Da / Yes | _(gol / empty)_ | Numele bazei de date | Database name |
| `DATABASE_URL` | ⬜ Nu / No | _(auto-construit / auto-built)_ | URL complet de conexiune (suprascrie variabilele de mai sus) | Full connection URL (overrides the above variables) |

> **🇷🇴** Dacă `DATABASE_URL` este setat, variabilele `MYSQL_*` individuale sunt ignorate. Formatul: `mysql+asyncmy://user:password@host:port/dbname`  
> **🇬🇧** If `DATABASE_URL` is set, the individual `MYSQL_*` variables are ignored. Format: `mysql+asyncmy://user:password@host:port/dbname`

---

### Securitate / Security

| Variabilă | Obligatorie / Required | Implicit / Default | Descriere (RO) | Description (EN) |
|---|---|---|---|---|
| `SECRET_KEY` | ✅ Da / Yes | _(gol / empty)_ | Cheia secretă pentru semnarea tuturor JWT-urilor | Secret key for signing all JWTs |
| `API_TOKEN_FRONTEND` | ✅ Da / Yes | _(gol / empty)_ | Token static pentru clientul web frontend | Static token for the web frontend client |
| `API_TOKEN_ADMIN` | ✅ Da / Yes | _(gol / empty)_ | Token static pentru clientul admin | Static token for the admin client |
| `API_TOKEN_MOBILE` | ✅ Da / Yes | _(gol / empty)_ | Token static pentru clientul mobil | Static token for the mobile client |

> **🇷🇴** `SECRET_KEY` este folosit pentru **toate** JWT-urile: token-ul de înregistrare al elevului (1 an), token-ul admin (1 oră) și token-ul QR (30 secunde). Folosește o valoare aleatoare lungă (minim 32 de caractere).  
> **🇬🇧** `SECRET_KEY` is used for **all** JWTs: the student enroll token (1 year), the admin token (1 hour), and the QR token (30 seconds). Use a long random value (minimum 32 characters).

---

### Connection Pool

| Variabilă | Obligatorie / Required | Implicit / Default | Descriere (RO) | Description (EN) |
|---|---|---|---|---|
| `DB_POOL_SIZE` | ⬜ Nu / No | `10` | Numărul de conexiuni permanente în pool | Number of persistent connections in the pool |
| `DB_MAX_OVERFLOW` | ⬜ Nu / No | `20` | Conexiuni suplimentare permise peste pool_size | Extra connections allowed above pool_size |
| `DB_POOL_TIMEOUT` | ⬜ Nu / No | `3` | Secunde de așteptare pentru o conexiune liberă | Seconds to wait for a free connection |
| `DB_POOL_RECYCLE` | ⬜ Nu / No | `1800` | Secunde după care o conexiune este reciclată | Seconds after which a connection is recycled |
| `SQL_ECHO` | ⬜ Nu / No | `0` | `1` = afișează query-urile SQL în log | `1` = log all SQL queries |

---

## 📄 Exemplu fișier `.env` / Example `.env` file

```env
# Baza de date / Database
MYSQL_USER=pontaj_user
MYSQL_PASSWORD=parola_secreta_puternica
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_DB=pontaj_db

# Securitate / Security
SECRET_KEY=schimba_asta_cu_o_valoare_aleatoare_lunga_de_minim_32_caractere
API_TOKEN_FRONTEND=token_static_frontend_schimba_asta
API_TOKEN_ADMIN=token_static_admin_schimba_asta
API_TOKEN_MOBILE=token_static_mobile_schimba_asta

# Connection pool (opțional / optional)
DB_POOL_SIZE=10
DB_MAX_OVERFLOW=20
DB_POOL_TIMEOUT=3
DB_POOL_RECYCLE=1800
SQL_ECHO=0
```

---

## ⚠️ Avertisment de securitate / Security Warning

### 🇷🇴
- **Nu comite niciodată** fișierul `.env` în Git. Verifică că `.gitignore` conține `.env`.
- **Generează valori unice** pentru `SECRET_KEY` și toate token-urile API. Poți folosi: `python -c "import secrets; print(secrets.token_hex(32))"`
- **Rotește token-urile** periodic, mai ales după ce un colaborator părăsește proiectul.
- În producție, consideră folosirea unui manager de secrete (AWS Secrets Manager, HashiCorp Vault, etc.) în loc de fișiere `.env`.

### 🇬🇧
- **Never commit** the `.env` file to Git. Verify that `.gitignore` contains `.env`.
- **Generate unique values** for `SECRET_KEY` and all API tokens. You can use: `python -c "import secrets; print(secrets.token_hex(32))"`
- **Rotate tokens** periodically, especially after a collaborator leaves the project.
- In production, consider using a secrets manager (AWS Secrets Manager, HashiCorp Vault, etc.) instead of `.env` files.

---

## 🔗 Vezi și / See Also

- [deployment.md](deployment.md) — Cum să pornești aplicația / How to start the application
- [auth-flow.md](auth-flow.md) — Cum sunt folosite token-urile și cheile / How tokens and keys are used
