# Cloud Deployment Guide: Campus Resource Engine

This guide walks you through deploying the `campus-resource-engine` project to the cloud. We'll use a modern, cost-effective stack:
- **GitHub** for version control
- **Supabase** for the PostgreSQL Database
- **Railway** for the Node.js Backend and Redis
- **Vercel** for the Next.js Frontend

---

## Step 1: Push Code to GitHub
Both Railway and Vercel will deploy automatically when you push code to your GitHub repository.

1. Go to [GitHub](https://github.com/) and create a new repository (e.g., `campus-resource-engine`).
2. Open your terminal in the root of your local project and run:
   ```bash
   git init
   git add .
   git commit -m "Initial commit for deployment"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/campus-resource-engine.git
   git push -u origin main
   ```

---

## Step 2: Setup Supabase (Database)
You are already using Supabase, so you just need to ensure your production database is set up.

1. Go to [Supabase](https://supabase.com/) and create a new Project.
2. Note down the **Database Password**, **Project URL**, and **anon key** (found in Project Settings -> API).
3. Get your **Database Connection string** (found in Project Settings -> Database -> URI). It should look like `postgresql://postgres.[ref]:[password]@aws-0-[region].pooler.supabase.com:6543/postgres`.

*Note: You will need to run your initial database migrations/seed scripts against this new Supabase production database.*

---

## Step 3: Setup Railway (Backend & Redis)
Railway is perfect for our Express backend and Redis cache.

### 3a. Provision Redis
1. Make an account on [Railway.app](https://railway.app/).
2. Click **New Project** -> **Provision Redis**.
3. Railway will provision a Redis instance. Note its internal connection URL (available in its Variables tab once created).

### 3b. Deploy the Backend Server
1. In the same Railway project, click **New** -> **GitHub Repo** and select your `campus-resource-engine` repository.
2. Railway might try to guess the deployment. We need to tell it to only build the backend.
3. Once the service appears, click on it -> Go to **Settings**:
   - Give it a name like `campus-backend`.
   - **Root Directory**: `server` (or leave as `/` but ensure the Dockerfile path is `server/Dockerfile`). Since you have a `railway.toml` at the root pointing to `server/Dockerfile`, Railway might auto-detect this correctly.
   - **Build Command**: Leave blank (it uses the Dockerfile).
4. Go to the **Variables** tab for the backend service. Add all variables from your `server/.env.example`:
   - `NODE_ENV=production`
   - `DATABASE_URL`: Your Supabase connection string.
   - `DIRECT_URL`: Same as above but port `5432` for direct access if needed.
   - `REDIS_URL`: Use the private Railway URL provided by the Redis service you just created (e.g., `redis://redis.railway.internal:6379`).
   - `JWT_SECRET` & `JWT_REFRESH_SECRET`: Generate long random strings for these.
   - `CORS_ORIGIN`: Set this to your frontend URL (you can update this *after* Step 4).
   - *(Add any other required vars like SMTP for emails).*
5. Go to the **Settings** tab -> **Networking** and click **Generate Domain**. This will be your `API_URL` (e.g., `https://campus-backend-production.up.railway.app`).

---

## Step 4: Setup Vercel (Frontend Next.js)
Vercel is the creator of Next.js and the absolute best place to host it.

1. Make an account on [Vercel.com](https://vercel.com/) and link your GitHub.
2. Click **Add New...** -> **Project**.
3. Import your `campus-resource-engine` repository.
4. In the **Configure Project** screen:
   - **Framework Preset**: Next.js
   - **Root Directory**: Click "Edit" and change it to `client`.
5. Open the **Environment Variables** section and add the variables from `client/.env.example`:
   - `NEXT_PUBLIC_API_URL`: The domain Railway generated for you + `/api/v1` (e.g., `https://campus-backend-production.up.railway.app/api/v1`).
   - `NEXT_PUBLIC_SOCKET_URL`: The domain Railway generated for you (e.g., `https://campus-backend-production.up.railway.app`).
   - `NEXT_PUBLIC_SUPABASE_URL`: Your Supabase Project URL.
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Your Supabase anon key.
6. Click **Deploy**.

---

## Final Review
1. Ensure Vercel successfully builds and gives you a frontend URL (e.g., `https://campus-engine.vercel.app`).
2. **Crucial:** Go back to your Railway Backend service -> **Variables** tab, and update the `CORS_ORIGIN` to match your new Vercel frontend URL exactly (e.g., `https://campus-engine.vercel.app`).
3. Your app is now live in the cloud!

## Maintenance & Updates
Whenever you push changes to your `main` branch on GitHub:
- Vercel will automatically detect changes in the `client/` folder and rebuild the frontend.
- Railway will automatically detect changes and rebuild the backend.
