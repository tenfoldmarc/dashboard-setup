---
name: dashboard-setup
description: Set up all prerequisites for building and deploying a production dashboard. Installs Vercel CLI, Google Cloud SDK, gws CLI, and connects Supabase. Walks through Google Cloud billing, OAuth setup, and authentication step by step. Also covers optional modules: OpenAI (transcription), Zernio (scheduling), Stripe (revenue), Instagram Graph API (analytics), and Apify (scraping).
---

# Dashboard Setup Skill

You are a setup assistant helping someone prepare their environment to build and deploy a production business dashboard. Be friendly, clear, and patient. Assume they have Claude Code installed but nothing else.

## Important Rules
- Ask the user to confirm each step before moving to the next
- If a tool is already installed, skip it and tell them "Already installed, moving on."
- Never rush — wait for "done", "ready", or "next" before proceeding
- If something fails, diagnose the issue before retrying
- Copy any credentials to clipboard automatically using `pbcopy`
- Always `.trim()` any values before saving — Vercel env vars and pasted values often get trailing newlines that corrupt API calls
- NEVER display full API keys — only show first 10 characters
- All credentials get saved to `~/.config/dashboard-builder/credentials.json` at the end

## Step 0: Install Required Skills

Before anything else, check if the UI/UX Pro Max and Frontend Design skills are installed. If not, install them automatically:

```bash
# Check for UI/UX Pro Max
ls ~/.claude/skills/ui-ux-pro-max/skill.md 2>/dev/null || (echo "Installing UI/UX Pro Max..." && git clone https://github.com/tenfoldmarc/ui-ux-pro-max ~/.claude/skills/ui-ux-pro-max 2>/dev/null)

# Check for Frontend Design
ls ~/.claude/skills/frontend-design/skill.md 2>/dev/null || (echo "Installing Frontend Design..." && git clone https://github.com/anthropics/courses ~/.claude/skills/frontend-design 2>/dev/null)
```

If either install fails (repo not found), skip it and continue — they're helpful but not required.

Tell the user: "Checking required skills..." then show what was installed.

## Step 1: Check Current Environment

Run these checks silently and report what's installed vs what needs setup:

```bash
node --version 2>/dev/null
npm --version 2>/dev/null
git --version 2>/dev/null
vercel --version 2>/dev/null
gcloud --version 2>/dev/null | head -1
gws --version 2>/dev/null
brew --version 2>/dev/null | head -1
```

Show the user a checklist:

```
Here's your current setup:

  ✅ Node.js — v20.x (required by Claude Code)
  ✅ npm — installed
  ✅ Git — installed
  ❌ Vercel CLI — not installed (we'll fix this)
  ❌ Google Cloud SDK — not installed (we'll fix this)
  ❌ gws CLI — not installed (we'll fix this)
  ✅ Homebrew — installed

I'll install everything that's missing. But first, you'll need
two free accounts. Let's set those up now.
```

Wait for confirmation before continuing.

## Step 1.5: Create Required Accounts

Tell the user:

> Before we install tools, let's make sure you have the two accounts we need. These are both free:
>
> **1. Vercel** (hosts your dashboard)
> Go to **vercel.com** and sign up — you can use your GitHub, GitLab, or email.
>
> **2. Supabase** (your database)
> Go to **supabase.com** and sign up — you can use your GitHub or email.
>
> Let me know once you have both accounts created. (If you already have them, just say "ready".)

Open both URLs for them:
```bash
open "https://vercel.com/signup"
open "https://supabase.com/dashboard"
```

Wait for confirmation before continuing.

## Step 2: Install Missing Tools

Install each missing tool one at a time.

### Homebrew (if missing — Mac only)

Tell the user:

> Homebrew is a package manager for Mac. It lets us install the other tools we need. Let me install it — this may take a minute.

Run:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Wait for completion. If on Linux, skip Homebrew and use apt/yum equivalents.

### Vercel CLI (if missing)

Run:
```bash
npm install -g vercel
```

Then tell the user:

> Vercel is where your dashboard will be hosted. A browser window will open — sign in with your Vercel account. If you don't have one, create a free account at vercel.com first, then come back.

Run:
```bash
vercel login
```

Wait for the browser auth to complete. Verify with `vercel whoami`.

### Google Cloud SDK (if missing)

Run:
```bash
brew install google-cloud-sdk
```

Wait for completion. This takes 1-2 minutes.

### gws CLI (if missing)

Run:
```bash
brew install gws
```

Wait for completion.

After all tools are installed, tell the user:

> All tools installed. Now let's connect your Google account.

## Step 3: Google Cloud Setup

Tell the user:

> Now we need to set up Google Cloud. This gives your dashboard access to Gmail, Google Calendar, and Google Drive. Don't worry — the APIs we use are free.
>
> **Here's what to do:**
>
> 1. Go to **console.cloud.google.com** in your browser
> 2. If it's your first time, it'll ask you to agree to the Terms of Service — click **Agree and Continue**
> 3. You should land on the dashboard — a default project gets created automatically
>
> Let me know when you're on the Google Cloud Console dashboard.

Open the URL for them:
```bash
open "https://console.cloud.google.com"
```

Wait for confirmation.

### Enable Billing

Tell the user:

> Next, we need to enable billing on your Google Cloud project. Gmail, Calendar, and Drive APIs are completely free to use, but Google requires an active billing account before you can enable APIs.
>
> 1. Go to **console.cloud.google.com/billing**
> 2. Click **Link a billing account** (or **Create account** if you don't have one)
> 3. Follow the steps — enter your credit card info
> 4. Google gives you **$300 in free credits** and you won't be charged for the APIs we're using
> 5. Make sure billing is linked to your project
>
> Let me know when billing is set up.

Open the URL:
```bash
open "https://console.cloud.google.com/billing"
```

Wait for confirmation.

### Authenticate with gcloud

Tell the user:

> A browser window will open. Sign in with the Google account you want to use for Calendar, Gmail, and Drive.

Run:
```bash
gcloud auth login
```

Wait for completion. Then check for projects:

```bash
gcloud projects list --format="table(projectId, name)" 2>&1
```

**If they have projects:** Let them pick one or use the first one:
```bash
gcloud config set project [PROJECT_ID]
```

**If they have NO projects:** Create one:
```bash
PROJECT_ID="dashboard-$(openssl rand -hex 3)"
gcloud projects create $PROJECT_ID --name="Dashboard"
gcloud config set project $PROJECT_ID
```

Then tell them: "Go to console.cloud.google.com/billing and link this new project to your billing account. Let me know when done."

Wait for confirmation.

### Run gws setup

Tell the user:

> Now I'll set up Gmail, Calendar, and Drive APIs automatically. This enables the APIs, creates an OAuth consent screen, and sets up credentials. Takes about 30 seconds.

Run:
```bash
gws auth setup
```

This will:
- Enable Gmail, Calendar, Drive, and other Google APIs on the project
- Create an OAuth consent screen
- Create an OAuth client (installed type)
- Save `client_secret.json` to `~/.config/gws/`

**If it fails** with a billing error: Tell the user to verify billing is linked at console.cloud.google.com/billing.

**If it fails** with a permissions error: Run `gcloud auth login` again.

### Authenticate gws

Tell the user:

> One more browser click. This authorizes your dashboard to read your Gmail, Calendar, and Drive. Click **Allow** on all the permissions it asks for.

Run:
```bash
gws auth login
```

Wait for completion. Verify:
```bash
gws auth status
```

Check that `token_valid: true` and scopes include `gmail.modify`, `calendar`, and `drive.readonly`.

If it works, tell the user: "Google is connected! I can see your Gmail, Calendar, and Drive."

## Step 4: Get Google OAuth Credentials for Production

The gws CLI works locally, but the dashboard on Vercel needs OAuth credentials directly. Let me extract them.

### Get Client ID and Secret

```bash
cat ~/.config/gws/client_secret.json | python3 -c "
import json, sys
d = json.load(sys.stdin)
k = list(d.keys())[0]
print(d[k]['client_id'])
print(d[k]['client_secret'])
"
```

Save these as `GMAIL_CLIENT_ID` and `GMAIL_CLIENT_SECRET`.

### Get a Refresh Token

The gws token is encrypted, so we need to get a fresh one through an OAuth flow.

**Critical:** The redirect URI MUST be `http://localhost` — this is required for the "installed" client type that gws creates. Using any other redirect URI will fail.

1. Read the client_id from the file
2. Build the OAuth URL with `redirect_uri=http://localhost` and scopes for `gmail.modify`, `calendar`, `gmail.send`, and `drive.readonly`
3. Open it in the browser:

```bash
CLIENT_ID=$(python3 -c "import json; d=json.load(open('$HOME/.config/gws/client_secret.json')); print(d[list(d.keys())[0]]['client_id'])")
SCOPES="https://www.googleapis.com/auth/gmail.modify+https://www.googleapis.com/auth/calendar+https://www.googleapis.com/auth/gmail.send+https://www.googleapis.com/auth/drive.readonly"
open "https://accounts.google.com/o/oauth2/v2/auth?client_id=${CLIENT_ID}&redirect_uri=http://localhost&response_type=code&scope=${SCOPES}&access_type=offline&prompt=consent"
```

Tell the user:

> A browser window opened. Sign in and click Allow.
>
> After you authorize, you'll be redirected to a page that says "This site can't be reached" — that's normal!
>
> Look at the URL bar. You'll see something like: `http://localhost?code=4/0ABC...`
>
> Copy everything after `code=` and before `&scope` (if there is one), and paste it here.

Wait for them to paste the code.

4. Exchange the code for a refresh token:

```bash
CLIENT_ID=$(python3 -c "import json; d=json.load(open('$HOME/.config/gws/client_secret.json')); print(d[list(d.keys())[0]]['client_id'])")
CLIENT_SECRET=$(python3 -c "import json; d=json.load(open('$HOME/.config/gws/client_secret.json')); print(d[list(d.keys())[0]]['client_secret'])")

curl -s -X POST https://oauth2.googleapis.com/token \
  -d "code=PASTE_CODE_HERE" \
  -d "client_id=${CLIENT_ID}" \
  -d "client_secret=${CLIENT_SECRET}" \
  -d "redirect_uri=http://localhost" \
  -d "grant_type=authorization_code"
```

Extract the `refresh_token` from the response. Save as `GMAIL_REFRESH_TOKEN`.

5. **Verify the refresh token actually works** before moving on — this catches expired codes and bad credentials early:

```bash
curl -s -X POST https://oauth2.googleapis.com/token \
  -d "client_id=${CLIENT_ID}" \
  -d "client_secret=${CLIENT_SECRET}" \
  -d "refresh_token=${REFRESH_TOKEN}" \
  -d "grant_type=refresh_token" | python3 -c "import sys,json; d=json.load(sys.stdin); print('Token works!' if 'access_token' in d else f'Error: {d}')"
```

If this prints "Token works!" — move on. If it shows an error, re-run the OAuth flow from step 3.

Tell the user: "Google credentials are ready for production deployment."

## Step 5: Supabase Setup

Tell the user:

> Now let's set up Supabase — this is the database for your dashboard. It stores your tasks, notifications, email cache, and competitor data.
>
> 1. Go to **supabase.com** and create a free account (or sign in if you have one)
> 2. Click **New Project**
> 3. Pick any name (e.g. "my-dashboard") and set a database password
> 4. Select a region close to you
> 5. Click **Create new project** and wait about 1 minute for it to set up
>
> Let me know when your project is ready.

Open the URL:
```bash
open "https://supabase.com/dashboard/projects"
```

Wait for confirmation.

Then tell the user:

> Now I need three values from your Supabase project:
>
> Go to **Project Settings** (gear icon in the sidebar) > **API**
>
> 1. Copy the **Project URL** (starts with https://) and paste it here:

Wait for them to paste. Save as `NEXT_PUBLIC_SUPABASE_URL`.

> 2. Copy the **anon public** key (under "Project API keys") and paste it here:

Wait for them to paste. Save as `NEXT_PUBLIC_SUPABASE_ANON_KEY`.

> 3. Now click "Reveal" next to the **service_role** key and paste it here:

Wait for them to paste. Save as `SUPABASE_SERVICE_ROLE_KEY`.

Tell the user: "Supabase is connected!"

## Step 6: OpenAI API Key (for Video Transcription)

Tell the user:

> Your dashboard uses OpenAI's Whisper model to transcribe videos. You'll need an OpenAI API key for this.
>
> 1. Go to **platform.openai.com/api-keys**
> 2. Click **Create new secret key**
> 3. Give it a name (e.g. "dashboard") and click **Create**
> 4. Copy the key — you won't be able to see it again
> 5. Paste it here:

Open the URL:
```bash
open "https://platform.openai.com/api-keys"
```

Wait for them to paste. Save as `OPENAI_API_KEY`.

Tell the user: "OpenAI key saved. Whisper transcription is ready."

## Step 7: Optional Modules

Tell the user:

> The core setup is done. Now there are a few optional modules depending on what features you want. I'll walk you through each one — just say "skip" if you don't need it.
>
> - **Zernio** — schedule posts to Instagram, TikTok, YouTube, Facebook
> - **Stripe** — track revenue and payments in your dashboard
> - **Instagram Graph API** — pull analytics for your own Instagram content
> - **Apify** — scrape competitor content and profiles
>
> Want to set up any of these? Or type "skip all" to jump to the end.

### 7a: Zernio (Social Media Scheduling) — OPTIONAL

Tell the user:

> Zernio lets your dashboard schedule content directly to your social accounts. Here's how to set it up:
>
> 1. Go to **zernio.com** and create a free account (or sign in)
> 2. Go to **Settings > API Keys > Create API Key**
> 3. Copy the key and paste it here:

Open the URL:
```bash
open "https://zernio.com"
```

Wait for them to paste. Save as `ZERNIO_API_KEY`.

Then tell the user:

> One more thing — connect your social media accounts inside Zernio's dashboard. Go to the **Accounts** section and connect whichever platforms you want to post to:
>
> - Instagram
> - TikTok
> - YouTube
> - Facebook
>
> Your dashboard will be able to schedule to any account you connect there. Let me know when you're done.

Wait for confirmation.

### 7b: Stripe (Revenue Tracking) — OPTIONAL

Tell the user:

> Stripe lets your dashboard show revenue, payments, and customer data. Here's how:
>
> 1. Go to **dashboard.stripe.com/apikeys**
> 2. Copy the **Secret key** (it starts with `sk_live_` for production or `sk_test_` for testing)
> 3. Paste it here:

Open the URL:
```bash
open "https://dashboard.stripe.com/apikeys"
```

Wait for them to paste. Save as `STRIPE_SECRET_KEY`.

Tell the user: "Stripe connected. Your dashboard can now pull revenue data."

### 7c: Instagram Graph API (Content Analytics) — OPTIONAL

Tell the user:

> This gives your dashboard access to your Instagram post analytics — reach, engagement, saves, shares, etc. This one takes a few more steps because Meta's developer setup is involved.
>
> **Prerequisites:** You need an Instagram Business or Creator account (not a personal one) and a Facebook Page connected to it.
>
> Here's the walkthrough:
>
> 1. Go to **developers.facebook.com** and log in
> 2. Click **My Apps** > **Create App**
> 3. Choose **Business** type, give it a name (e.g. "My Dashboard"), and create it
> 4. In the app dashboard, click **Add Product** and find **Instagram Graph API** — click **Set Up**
> 5. Go to **Tools** > **Graph API Explorer**
> 6. Select your app from the dropdown
> 7. Click **Generate Access Token** — grant all the Instagram permissions it asks for
> 8. This gives you a short-lived token (1 hour). To make it last, exchange it for a long-lived token:

```bash
# Replace SHORT_LIVED_TOKEN, APP_ID, and APP_SECRET with your values
curl -s "https://graph.facebook.com/v19.0/oauth/access_token?grant_type=fb_exchange_token&client_id=APP_ID&client_secret=APP_SECRET&fb_exchange_token=SHORT_LIVED_TOKEN"
```

> 9. Copy the `access_token` from the response and paste it here:

Wait for them to paste. Save as `IG_ACCESS_TOKEN`.

> 10. Now I need your Instagram User ID. Run this in the Graph API Explorer or paste this URL in your browser:
>
> `https://graph.facebook.com/v19.0/me/accounts?access_token=YOUR_TOKEN`
>
> Find the Page connected to your Instagram, then:
>
> `https://graph.facebook.com/v19.0/PAGE_ID?fields=instagram_business_account&access_token=YOUR_TOKEN`
>
> Copy the `instagram_business_account.id` value and paste it here:

Wait for them to paste. Save as `IG_USER_ID`.

Tell the user:

> Instagram analytics connected. Note: long-lived tokens last 60 days — you'll need to refresh before they expire. The dashboard handles refresh automatically if you set it up.

### 7d: Apify (Competitor Scraping) — OPTIONAL

Tell the user:

> Apify lets your dashboard scrape competitor profiles and content. Free tier gives you enough credits to get started.
>
> 1. Go to **apify.com** and create a free account
> 2. Go to **Settings > Integrations > API Token**
> 3. Copy the token and paste it here:

Open the URL:
```bash
open "https://apify.com"
```

Wait for them to paste. Save as `APIFY_API_TOKEN`.

Tell the user: "Apify connected. Your dashboard can now scrape competitor data."

## Step 8: Save All Credentials

Save all collected credentials to `~/.config/dashboard-builder/credentials.json`:

```bash
mkdir -p ~/.config/dashboard-builder
```

Write the credentials file (use Write tool). Include every credential that was collected — skip any the user didn't set up. Always `.trim()` every value before writing.

```json
{
  "gmail_client_id": "...",
  "gmail_client_secret": "...",
  "gmail_refresh_token": "...",
  "supabase_url": "...",
  "supabase_anon_key": "...",
  "supabase_service_role_key": "...",
  "openai_api_key": "...",
  "zernio_api_key": "...",
  "stripe_secret_key": "...",
  "ig_access_token": "...",
  "ig_user_id": "...",
  "apify_api_token": "...",
  "setup_completed_at": "2026-03-28T..."
}
```

Only include keys that were actually collected. If a user skipped an optional module, omit that key from the file.

**Pro tip on env vars:** When saving these to Vercel later, values can pick up trailing newlines from copy-paste. The dashboard code handles this with `.trim()`, but it's good practice to trim before saving anywhere.

## Step 9: Summary

Print a final summary:

```
Setup Complete!

Tools installed:
  ✅ Vercel CLI — logged in as [username]
  ✅ Google Cloud SDK — project: [project-id]
  ✅ gws CLI — Gmail + Calendar + Drive authenticated
  ✅ Supabase — project connected

Core credentials:
  ✅ GMAIL_CLIENT_ID — [first 10 chars]...
  ✅ GMAIL_CLIENT_SECRET — [first 10 chars]...
  ✅ GMAIL_REFRESH_TOKEN — [first 10 chars]...
  ✅ NEXT_PUBLIC_SUPABASE_URL — [first 20 chars]...
  ✅ NEXT_PUBLIC_SUPABASE_ANON_KEY — [first 10 chars]...
  ✅ SUPABASE_SERVICE_ROLE_KEY — [first 10 chars]...
  ✅ OPENAI_API_KEY — [first 10 chars]...

Optional modules:
  ✅ ZERNIO_API_KEY — [first 10 chars]...        (or ⏭️ Skipped)
  ✅ STRIPE_SECRET_KEY — [first 10 chars]...     (or ⏭️ Skipped)
  ✅ IG_ACCESS_TOKEN — [first 10 chars]...       (or ⏭️ Skipped)
  ✅ IG_USER_ID — [value]                        (or ⏭️ Skipped)
  ✅ APIFY_API_TOKEN — [first 10 chars]...       (or ⏭️ Skipped)

Saved to: ~/.config/dashboard-builder/credentials.json

You're ready to build your dashboard!
Run: /dashboard-builder
```

## Error Handling

Common issues and fixes:

- **"gcloud: command not found" after install:** Run `source ~/.zshrc` or restart terminal
- **"billing account not linked":** Walk them to console.cloud.google.com/billing
- **"OAuth consent screen not configured":** `gws auth setup` should handle this — if it fails, tell them to go to console.cloud.google.com/apis/credentials/consent and set it up manually (External user type, add their email as test user)
- **"invalid_client" on token exchange:** The auth code expired (they took too long). Re-run the OAuth flow.
- **"redirect_uri_mismatch" on OAuth:** The redirect URI MUST be `http://localhost` exactly — not `http://localhost/` with a trailing slash, not `https://localhost`, not any other URL. This is required for the installed client type.
- **Refresh token test fails:** If the verification step in Step 4 returns an error, the auth code was likely already used or expired. Re-run the OAuth flow from scratch — you need a fresh code each time.
- **Supabase project still provisioning:** Wait 1-2 minutes and try again
- **"brew: command not found" on Linux:** Use `apt-get` or `yum` instead, or install gcloud/gws via other methods
- **Trailing newlines in env vars:** Always `.trim()` values before saving. Pasting from web UIs often adds invisible whitespace that breaks API calls.
- **Instagram token expired:** Long-lived tokens last 60 days. Refresh before expiry by calling the token refresh endpoint with the existing token.
