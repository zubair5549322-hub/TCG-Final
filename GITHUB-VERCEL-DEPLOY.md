# GitHub + Vercel deployment

## Architecture

- **GitHub** — source control and automatic deployment trigger.
- **Vercel** — React/Vite frontend.
- **Persistent Node host (Render recommended)** — Express API + SQLite database.
- The browser talks to the API through `VITE_API_URL`.

### Why it is split

The application uses `better-sqlite3`, which is a persistent local SQLite database. Vercel Functions are ephemeral/serverless and are not a suitable persistent home for this database. Moving the database to a serverless SQL provider would require rewriting the application's SQLite-specific database layer.

## 1. Publish to GitHub

Create a GitHub repository and upload/push the contents of this `cafe_final` folder.

Do not commit `backend/data`, `.env`, or `node_modules`.

## 2. Deploy the backend

Render is one supported option because it can run the existing Node/Express backend and attach a persistent disk.

1. In Render, choose **New -> Blueprint** and connect the GitHub repository.
2. Select the repository's `render.yaml`.
3. Create the service.
4. Set `CORS_ORIGINS` to the Vercel URL after the frontend is deployed.
5. Keep the generated `JWT_SECRET`.
6. The database is stored under `/var/data/cafe`.

Backend health check:

`https://YOUR-BACKEND-DOMAIN/api/health`

It should return JSON containing `ok: true`.

## 3. Deploy the frontend to Vercel

1. In Vercel, choose **Add New -> Project**.
2. Import the same GitHub repository.
3. Vercel uses the included `vercel.json`.
4. Add this Environment Variable:

`VITE_API_URL=https://YOUR-BACKEND-DOMAIN`

Do not add `/api`; the application adds it automatically. A value already ending in `/api` is also accepted.

5. Deploy.

Future GitHub pushes can trigger new Vercel deployments.

## 4. Connect CORS

After Vercel gives you a domain, set the backend variable:

`CORS_ORIGINS=https://YOUR-VERCEL-DOMAIN.vercel.app`

For a custom domain such as `https://thecommongrounds.pk`, include both origins separated by commas:

`https://YOUR-VERCEL-DOMAIN.vercel.app,https://thecommongrounds.pk`

Redeploy/restart the backend after changing the variable.

## 5. Database note

Do not deploy the SQLite database on Vercel. The persistent database remains on the backend host's mounted disk.

The application's existing backup/export features continue to operate against that persistent backend storage.

## 6. Local use still works

The original local Windows setup remains supported. `VITE_API_URL` defaults to `/api`, so when Express serves the frontend locally, no extra frontend environment variable is required.
