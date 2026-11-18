# Veloura

Veloura is an editable React + TypeScript product showcase with username/password registration, login, and a product page available only to signed-in accounts. Its API is built with FastAPI `APIRouter`; passwords are hashed in Python with `hashlib.scrypt`. This is source code, not a ZIP package.

## Open and run in VS Code

1. In GitHub Desktop, choose **File → Clone repository…**, select the private Veloura repository after its connection is complete, and choose a folder on your computer.
2. Choose **Open in Visual Studio Code** from GitHub Desktop (or open the cloned folder from VS Code).
3. Install Node.js 22 and pnpm 10.18.0, then install the frontend dependencies with `pnpm install`.
4. Install Python 3.12. From the project folder, create a virtual environment and install the Python requirements:

   ```sh
   python3 -m venv .venv
   .venv/bin/python -m pip install -r requirements.txt
   ```

   On Windows PowerShell, use `py -3.12 -m venv .venv` and `.\.venv\Scripts\python.exe -m pip install -r requirements.txt`.
5. Copy `.env.example` to `.env` and set `DATABASE_URL` to a MySQL-compatible database. For plain-HTTP local development set `VELOURA_COOKIE_SECURE=false`; hosted HTTPS Preview and production use secure cookies by default. Never commit `.env` or real database credentials.
6. Apply the committed schema migration with `pnpm db:migrate`, then start with `pnpm dev`. The Express/Vite Preview server starts the FastAPI child process and proxies `/api/v1/*` to it. The child process uses `.venv` automatically; set `VELOURA_PYTHON` to a different interpreter path if needed.

In Windows PowerShell, set `$env:NODE_ENV = "development"` and `$env:VELOURA_PYTHON = ".venv\Scripts\python.exe"`, then run `pnpm exec tsx watch server/_core/index.ts`.

Cloud development supplies the managed `DATABASE_URL`. A local clone needs its own MySQL-compatible database and connection string; the database migration is additive and reuses the `veloura_accounts` and `veloura_sessions` tables.

## Edit the product shown after login

Update `backend/veloura_api/product.py`. The sample is **Veloura Lumière Eau de Parfum**, priced at **$68.00**, with an editable description and note. FastAPI returns this record only from the authenticated `/api/v1/products/featured` endpoint.

## Account behavior

Visitors can register with a username and a password of at least eight characters, then log in and sign out. Usernames are trimmed and normalized to lowercase. Password hashes use `hashlib.scrypt` with a random salt, `N=16384`, `r=8`, `p=1`, and a 64-byte derived key; the stored hash contains no plaintext password. The browser receives an opaque session token in an HttpOnly cookie, while only its SHA-256 digest is stored in MySQL. The database migration is additive and leaves the starter OAuth user table intact.

Authentication POST requests require a same-origin custom header and use process-local throttles. For a multi-instance deployment, add trusted edge rate limiting because in-memory limits are not shared across workers and proxied clients may share a source address.

The development wrapper keeps the browser on same-origin relative API URLs and forwards the API calls to FastAPI. The production Docker image builds the React assets with Node, then serves the static app and FastAPI API from the Python runtime. The health endpoint is `/api/health`.

## Useful commands

- `pnpm dev` — run the development server, Vite, and FastAPI.
- `pnpm check` — run TypeScript checks.
- `pnpm test` — run the existing TypeScript starter tests.
- `.venv/bin/python -m unittest discover -s backend/tests -t backend -v` — run Python backend tests. In Windows PowerShell, use `.\.venv\Scripts\python.exe -m unittest discover -s backend/tests -t backend -v`.
- `pnpm build` — build the frontend and starter server bundle.
- `pnpm db:migrate` — apply committed Drizzle migrations.

Use GitHub Desktop to review source changes, create a commit, and push to the private repository after the platform completes the repository connection.
