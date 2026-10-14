## 1. Run it

Choose the path that matches your OS. All three reach the same app at the end — pick one.

### Option A — One-command bootstrap (recommended for Ubuntu and macOS)

Works on a **fresh Ubuntu 22.04 / 24.04** or **macOS** with nothing pre-installed except Git and internet. Installs PostgreSQL, MongoDB, Node 20, Python; writes `.env`; creates the database; runs the schema, seeders, and ledger back-fill; installs Python and npm dependencies. Takes 15–25 min the first time.

```bash
git clone -b mandemo https://github.com/Palakshghildiyal17/DEMO.git
cd DEMO
sudo bash scripts/bootstrap.sh          # Ubuntu
# or, on macOS (no sudo needed):
bash scripts/bootstrap.sh
```

`scripts/bootstrap.sh` detects the OS and runs the right script:
`bootstrap-ubuntu.sh` on Linux, `bootstrap-macos.sh` on macOS.

When it prints **"RideSync is set up."**, start the app in two terminals:

```bash
# Terminal 1 — API
cd api && npm start                     # http://localhost:4000/api/health

# Terminal 2 — web
cd web && npm run dev                   # http://localhost:5173
```

**First thing to do in the app:** choose a rider in **"Acting as"** (top of the sidebar). There's no login; the choice is remembered in the browser.

### Option B — Docker (recommended for Windows and any other OS)

Requires only **Docker Desktop**. Works on Windows 10/11, macOS, Linux, and any cloud VM. No PostgreSQL, MongoDB, Node, or Python installs needed on the host.

```bash
git clone -b mandemo https://github.com/Palakshghildiyal17/DEMO.git
cd DEMO
docker compose up --build
```

- Web app: **http://localhost:8080** · API health: http://localhost:4000/api/health
- The first start builds the schema, seeds about 650k rows/documents and back-fills the wallet ledger. This takes a few minutes; watch `docker compose logs -f seed`.
- Data persists in Docker volumes. To start again from scratch: `docker compose run --rm -e RESEED=1 seed`.
- The containers' databases use host ports **5433** (PostgreSQL) and **27018** (MongoDB), so they don't clash with a PostgreSQL or MongoDB already installed on the PC.
- Stop everything with `docker compose down`. Start it again later with `docker compose up` (no re-seeding; the data stays in the volumes). `docker compose down -v` also deletes the data.

> On Windows, install Docker Desktop from https://www.docker.com/products/docker-desktop with the default WSL 2 backend. A best-effort PowerShell script (`scripts/bootstrap-windows.ps1`) exists but is untested; Docker is the recommended path on Windows.

### Option C — Manual local install (Ubuntu / macOS)

If you prefer to install each tool yourself, or you are on a machine where the bootstrap script cannot run:

**Prerequisites:** PostgreSQL 14+, MongoDB 7+ with `mongosh`, Python 3.10+, Node.js 20+ (`psql`, `mongosh`, `python`, `node` on PATH).

```bash
# 0. configuration
cp .env.example .env          # then set POSTGRES_PASSWORD

# 1. database (Ubuntu)
sudo -u postgres createdb ridesync_db
# macOS with brew:
createdb ridesync_db

# 2. Python deps for the seeders
python3 -m venv venv
source venv/bin/activate
pip install -r data_generation/requirements.txt

# 3. Mongo indexes first
mongosh ridesync_db mongo/01_collections_and_indexes.js

# 4. schema, seed data, ledger back-fill, DB tests (one script)
bash scripts/setup_db.sh

# 5. API (terminal 1)
cd api && npm install && npm start     # http://localhost:4000/api/health

# 6. Web (terminal 2)
cd web && npm install && npm run dev   # http://localhost:5173
```

`scripts/setup_db.sh` runs the SQL in order (`sql/01`–`05`, `07`) → `postgres_seeder.py` → `mongo_seeder.py` → `sql/08` → `REFRESH MATERIALIZED VIEW`. It is idempotent: re-running skips what is already done.

**Platform support:**

| OS | Recommended | Fallback |
|---|---|---|
| Ubuntu 22.04 / 24.04 | `sudo bash scripts/bootstrap.sh` | Docker |
| macOS (Intel / Apple Silicon) | `bash scripts/bootstrap.sh` | Docker |
| Windows 10 / 11 | Docker Desktop | `scripts/bootstrap-windows.ps1` (untested) |
| Any OS with Docker | `docker compose up --build` | — |
