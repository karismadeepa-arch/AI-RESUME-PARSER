# AI-Powered Resume Parser & Multi-Channel 2FA Onboarding Pipeline

A three-day internship starter project using FastAPI, PostgreSQL, SQLAlchemy Async, Pydantic v2, Argon2 password hashing, JWT, PDF/DOCX parsing, Email/WhatsApp OTP and TOTP.

## 1. Requirements
- Windows 10/11
- Python 3.11
- PostgreSQL 14+
- VS Code
- Postman (optional; Swagger is included)

## 2. Create the database
Open pgAdmin or psql and run:
```sql
CREATE DATABASE resume_onboarding;
```

The reference SQL schema is in `database/schema.sql`. The application uses SQLAlchemy `create_all()` for this starter; use Alembic revisions for a real migration workflow.\n\n## 3. Setup in PowerShell
Open this project folder in VS Code, then run:
```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
Copy-Item .env.example .env
```
Edit `.env` and set your PostgreSQL password in `DATABASE_URL`.

If `py -3.11` is not found, install Python 3.11 from python.org and select **Add python.exe to PATH**, then restart VS Code. Check with `py --list`.

## 4. Start the API
Make sure PostgreSQL is running:
```powershell
uvicorn app.main:app --reload
```
Open http://127.0.0.1:8000/docs

## 5. API endpoints
| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/` | Health check |
| POST | `/api/v1/resume/parse` | Upload PDF/DOCX and extract fields |
| POST | `/api/v1/auth/register` | Stage account and send email OTP |
| POST | `/api/v1/auth/send-2fa/{user_id}?channel=whatsapp` | Send WhatsApp OTP |
| POST | `/api/v1/auth/totp/setup/{user_id}` | Create demo TOTP secret |
| POST | `/api/v1/auth/verify-2fa` | Verify email/WhatsApp OTP |
| POST | `/api/v1/auth/totp/verify/{user_id}` | Verify TOTP |
| GET | `/api/v1/profile` | Read authenticated profile |
| PATCH | `/api/v1/profile` | Edit profile |
| GET | `/api/v1/dashboard` | Student dashboard routing payload |

## 6. Local OTP testing
The default `OTP_DELIVERY_MODE=console` prints OTPs in the VS Code terminal. This is strictly a local development mode. For real email, configure SMTP and set `OTP_DELIVERY_MODE=smtp` (the dispatch implementation uses SMTP when the mode is not console). For WhatsApp, install Twilio SDK with `pip install twilio`, configure Twilio credentials, and use the send endpoint. WhatsApp delivery requires an approved/connected Twilio WhatsApp sender and recipient setup.

## 7. End-to-end test
1. Upload a PDF or DOCX at `/api/v1/resume/parse`.
2. Copy the returned fields into `/api/v1/auth/register` and choose a password of at least 8 characters.
3. Copy the development OTP from the terminal.
4. Verify with `/api/v1/auth/verify-2fa` using `{"user_id": 1, "channel": "email", "code": "123456"}`.
5. Click **Authorize** in Swagger and enter `Bearer <access_token>`.
6. Call `/api/v1/profile` and `/api/v1/dashboard`.

The TOTP setup endpoint stores a demo secret on the user and returns an `otpauth://` URI that can be added to an authenticator app. The verify endpoint reads the stored secret.\n\n## 8. Important implementation notes before production
This is a learning starter, not production-ready identity infrastructure:
- Resume parsing uses transparent regex/keyword heuristics, not a trained AI model. Always let candidates review extracted values.
- Candidate profile metadata is persisted in the `users.profile_data` JSONB column. For a larger production system, consider a dedicated profile table and PostgreSQL JSONB migrations.
- TOTP secret is persisted for this demo but is not encrypted. Production must bind setup to an authenticated user, encrypt the secret at rest, and require confirmation before enabling TOTP.
- Add rate limits, OTP attempt counters, resend cooldowns, generic authentication errors, audit logs, file malware scanning, MIME/signature checks, and secure secret management before production.
- The simple OTP hash is suitable only as a teaching example; use a keyed HMAC with a server-side secret, avoid logging OTPs, and invalidate older outstanding challenges on resend.
- Use Alembic migrations instead of `create_all` for production schema changes.
- For real production OTP dispatch, use a background task/queue and a dedicated provider integration.
