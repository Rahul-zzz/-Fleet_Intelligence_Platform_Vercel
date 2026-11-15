# Vercel + Render deployment

This repository is configured as a split deployment:

- Vercel hosts the React/Vite frontend.
- Render hosts the FastAPI backend using `backend/Dockerfile`.
- Render PostgreSQL stores application data.

## 1. Deploy the backend on Render

Create a **Web Service** from this GitHub repository:

- Runtime: Docker
- Branch: `main`
- Root Directory: `backend`
- Dockerfile Path: `Dockerfile`
- Health Check Path: `/api/health`

Set these environment variables in Render (do not commit secrets):

- `APP_ENV` = `production`
- `APP_VERSION` = `1.0.0`
- `DATABASE_URL` = Render PostgreSQL **Internal Database URL**, with the scheme changed from `postgresql://` to `postgresql+psycopg://` if needed
- `ADMIN_PASSWORD` = a strong password you create
- `ADMIN_TOKEN` = a separate long random secret
- `MAX_UPLOAD_MB` = `25`
- `CORS_ORIGINS` = your exact Vercel site origin, for example `https://your-project.vercel.app`

Optional AI settings (leave unset to use the built-in data-summary response):

- `LLM_BASE_URL` = `https://api.openai.com/v1`
- `LLM_API_KEY` = your provider API key
- `LLM_MODEL` = `gpt-4o-mini`

Deploy the service and confirm that `https://YOUR-RENDER-SERVICE.onrender.com/api/health` returns JSON with `"status":"ok"`.

## 2. Deploy the frontend on Vercel

Import the same GitHub repository as a Vercel project:

- Framework Preset: Vite
- Root Directory: `frontend`
- Build Command: `npm run build`
- Output Directory: `dist`
- Install Command: `npm install`

Add this Vercel environment variable before deploying:

- `VITE_API_BASE_URL` = `https://YOUR-RENDER-SERVICE.onrender.com`

Use the backend origin only (no `/api` suffix). Redeploy after changing environment variables.

## 3. Update backend CORS after Vercel deployment

If Vercel assigns a different production domain, update Render's `CORS_ORIGINS` to the exact production origin, such as `https://your-project.vercel.app`, then redeploy/restart the backend.

For a temporary preview deployment, add its exact origin as a comma-separated value only if you want that preview to access the backend.

## 4. Important operational limitations

- The backend currently stores pending import previews in process memory and uploaded preview files on the service filesystem. A restart or instance replacement can invalidate a pending preview; upload and confirm within the same running instance.
- Use one backend instance for this MVP unless pending-import state is moved to shared storage.
- Render filesystem storage may be ephemeral. The database is persistent, but uploaded source workbooks are not currently archived in durable object storage.
- The app's import mapping is generic. Validate it against your actual monthly workbook before relying on imported data.
- Review the utilization definition with the business owner; the current dashboard's running-share calculation is a simple running-record count divided by all status records, not a validated fleet utilization KPI.
- The current authentication is a single admin password/token, not a full multi-user identity system.
- Configure database backups and rotate credentials before production use.

## Local development

For local development, leave `VITE_API_BASE_URL` unset. Vite's dev server proxies `/api` to `http://localhost:8000`.
