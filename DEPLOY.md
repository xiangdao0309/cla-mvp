# Deployment Guide — Cla MVP to Vercel

## GitHub Repo
✅ Code already pushed: https://github.com/xiangdao0309/cla-mvp

## Step 1: Create a PostgreSQL Database

**Recommended: Neon (free tier, optimized for Vercel)**

1. Go to https://neon.tech and sign up with GitHub
2. Create a new project:
   - Name: `cla-mvp`
   - Region: pick closest to your users
3. Copy the connection string from the Neon dashboard
   - Format: `postgresql://user:password@ep-xxx.neon.tech/cla-mvp?sslmode=require`

**Alternative: Supabase or Railway**
- Both offer free PostgreSQL tiers
- Just need the full `DATABASE_URL` connection string

## Step 2: Set Up Google OAuth (for login)

1. Go to https://console.cloud.google.com/
2. Create a new project (or select existing)
3. Go to **APIs & Services > Credentials**
4. Create **OAuth 2.0 Client ID**:
   - Application type: Web application
   - Authorized redirect URIs: add `https://your-vercel-url.vercel.app/api/auth/callback/google`
5. Copy the **Client ID** and **Client Secret**

## Step 3: Deploy to Vercel

1. Go to https://vercel.com and log in (GitHub sign-in recommended)
2. Click **Add New > Project**
3. Import **xiangdao0309/cla-mvp** from GitHub
4. Framework: **Next.js** (auto-detected)
5. Root directory: `.` (default)
6. Build command: `npx prisma migrate deploy && next build` (or just `npm run build` and migrate separately)
7. Click **Deploy**

## Step 4: Configure Environment Variables

In Vercel project dashboard > **Settings > Environment Variables**, add:

| Name | Value | Notes |
|------|-------|-------|
| `DATABASE_URL` | Your Neon/Supabase connection string | Required — includes `?sslmode=require` |
| `AUTH_SECRET` | Random 32+ byte string | Generate: `openssl rand -base64 32` |
| `AUTH_JWT_SECRET` | Random 32+ byte string | Generate: `openssl rand -base64 32` |
| `AUTH_GOOGLE_ID` | From Google Cloud Console | Client ID |
| `AUTH_GOOGLE_SECRET` | From Google Cloud Console | Client Secret |
| `ANTHROPIC_API_KEY` | From console.anthropic.com | For AI suggestions + briefing |
| `ANTHROPIC_MODEL` | `claude-3-5-haiku-latest` | Optional, has fallback |
| `NEXTAUTH_URL` | Your Vercel URL | e.g., `https://cla-mvp.vercel.app` |
| `NODE_ENV` | `production` | Auto-set by Vercel |

⚠️ After adding `NEXTAUTH_URL`, **redeploy** the project (Vercel > Deployments > top right dots > Redeploy)

## Step 5: Run Database Migration

After first deploy, run Prisma migration on production:

```bash
DATABASE_URL="your-production-url" npx prisma migrate deploy
```

Or from Vercel dashboard: **Deployments > latest > Logs** to verify migration ran.

## Step 6: Smoke Test

1. Open your Vercel URL (e.g., `https://cla-mvp.vercel.app`)
2. Click "Sign in with Google" — verify OAuth flow works
3. Create a task, complete it
4. Check `/api/health` returns `{"status":"ok"}`
5. Log out, log back in — session persists

## Update Vercel Redirect URI

After getting your Vercel URL, update Google OAuth:
- Add `https://your-vercel-url/api/auth/callback/google` to authorized redirect URIs
- Redeploy if needed

## Troubleshooting

**"Cannot find module '@prisma/client'"**
- Run `npx prisma generate` locally, commit the generated client, and redeploy

**"Authentication error" on login**
- Verify `NEXTAUTH_URL` matches your exact Vercel URL (no trailing slash)
- Check `AUTH_SECRET` is set

**Database connection error**
- Verify `DATABASE_URL` has `?sslmode=require` for Neon
- Check Neon IP allowlist (Neon blocks by default; use connection string with SSL)

**Build fails**
- Check build logs in Vercel dashboard
- Ensure all env vars are set before deploying

## Production Checklist
- [ ] Database created (Neon/Supabase/Railway)
- [ ] Google OAuth client created with correct redirect URI
- [ ] Vercel project linked to GitHub
- [ ] All env vars configured
- [ ] Initial migration ran
- [ ] Smoke test passed (login, task CRUD, logout/login)
