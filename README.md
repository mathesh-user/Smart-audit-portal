# Smart Secretarial Audit Tracking &amp; Reporting Portal

A full-stack portal for company secretaries / compliance teams to track secretarial audits
across clients: standard compliance checklists, document evidence, automatic compliance
scoring, overdue-audit alerts, and PDF report generation.

**Stack:** Spring Boot 3 (Java 17) + Spring Security/JWT + JPA · React 18 (Vite) + React Router + Recharts

---

## 1. Backend — `backend/`

### Requirements
- Java 17+
- Maven 3.9+ (or use the included `mvnw` if you add one)
- Internet access to Maven Central on first build (to download dependencies)

### Run (H2, zero config — default profile)
```bash
cd backend
mvn spring-boot:run
```
The API starts on `http://localhost:8080`. An H2 file database is created at `backend/data/secaudit.mv.db`.
The H2 console is available at `http://localhost:8080/h2-console` (JDBC URL: `jdbc:h2:file:./data/secaudit`, user `sa`, no password).

On first run, a `DataSeeder` creates two demo accounts:

| Username | Password    | Role    |
|----------|-------------|---------|
| admin    | Admin@123   | ADMIN   |
| auditor  | Auditor@123 | AUDITOR |

**Change these before any real deployment.**

### Run against PostgreSQL instead
```bash
cd backend
mvn spring-boot:run -Dspring-boot.run.profiles=postgres \
  -DDB_HOST=localhost -DDB_NAME=secaudit_portal -DDB_USER=postgres -DDB_PASSWORD=postgres
```
Or set the equivalent environment variables (`DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`)
and pass `--spring.profiles.active=postgres`.

### Key environment variables
| Variable            | Default                                              | Purpose                        |
|---------------------|-------------------------------------------------------|---------------------------------|
| `JWT_SECRET`         | dev placeholder (32+ chars)                          | HMAC signing key — **set in prod** |
| `JWT_EXPIRATION_MS`  | `86400000` (24h)                                     | JWT token lifetime              |
| `UPLOAD_DIR`         | `uploads`                                            | Where documents & PDFs are stored |
| `SERVER_PORT`        | `8080`                                               | API port                        |

### API surface (all under `/api`, JWT-protected unless noted)
- `POST /auth/register`, `POST /auth/login` — public
- `GET/POST/PUT/DELETE /clients`, `/clients/{id}`
- `GET/POST/PATCH/DELETE /audits`, `/audits/{id}`, `/audits/{id}/status`, `/audits/{id}/recalculate`
- `GET/POST/PUT/DELETE /audits/{id}/checklist`, `/audits/{id}/checklist/{itemId}`
- `GET/POST/DELETE /audits/{id}/documents`, `GET .../download`
- `GET /audits/{id}/reports`, `POST /audits/{id}/reports/generate`, `GET /reports`, `GET /reports/{id}/download`
- `GET /dashboard/summary`

### What makes it "smart"
- **Auto-scoped checklist** — creating a new engagement seeds a standard secretarial compliance
  checklist (Companies Act 2013, SEBI LODR, FEMA, RBI, Secretarial Standards, labour law, CSR)
  instead of starting from a blank sheet.
- **Live compliance scoring** — `compliant / applicable × 100`, recalculated on every checklist change.
- **Automatic overdue flagging** — a daily scheduled job (`ScheduledTasks`) flags any engagement
  past its due date that isn't completed.
- **One-click PDF report generation** — produces a compliance summary + itemised checklist PDF
  (iText) with an auto-computed overall rating (Satisfactory / Needs Attention / Critical).

---

## 2. Frontend — `frontend/`

### Requirements
- Node.js 18+

### Setup
```bash
cd frontend
npm install
cp .env.example .env   # adjust VITE_API_URL if your backend isn't on localhost:8080
npm run dev
```
Opens at `http://localhost:5173`, proxying API calls to `VITE_API_URL` (defaults to `http://localhost:8080/api`).

### Build for production
```bash
npm run build   # outputs static assets to frontend/dist
```
Serve `dist/` with any static host (Nginx, S3+CloudFront, etc.) and point `VITE_API_URL`
at your deployed backend before building.

### Design system
A distinct "Compliance Ledger" visual identity — deep ink-navy panels, a brass/gold seal accent
(the circular compliance-score stamp used throughout), Fraunces for display type and IBM Plex
Sans/Mono for UI and figures — built to feel like an official audit record rather than a generic
SaaS dashboard.

---

## 3. Suggested next steps
- Add role-based UI gating (e.g. CLIENT role sees a read-only view of their own engagements).
- Email reminders for upcoming/overdue deadlines (a `spring-boot-starter-mail` dependency is
  already wired in, just needs an SMTP config + `@Scheduled` job to call it).
- Swap local file storage for S3/Azure Blob for documents and generated PDFs in production.
- Add pagination/server-side search once client and audit volumes grow.
