# ledger-keepalive

This repo has exactly one job: keep the Render-hosted backend for The
Ledger from going to sleep, by pinging it every 10 minutes, forever.

It's deliberately **separate** from the actual backend/frontend repos, and
deliberately **public**, for two reasons:

1. A sleeping Render process can't wake itself up, and a ping triggered
   from the frontend app only fires while someone has it open — neither
   works given how infrequent traffic actually is. The ping needs to come
   from something that runs independently of both apps.
2. GitHub Actions minutes are unlimited and free on public repos. Running
   this same schedule inside a private repo could eventually cost money
   once free private-repo minutes run out — a public repo with no actual
   application code in it sidesteps that entirely.

## Setup

1. Create a new **public** GitHub repo and push this folder's contents to it.
2. In the repo's **Settings → Secrets and variables → Actions → Variables**
   tab, add a repository variable:
   ```
   Name:  RENDER_HEALTH_URL
   Value: https://your-app-name.onrender.com/health
   ```
   (use your actual Render backend URL from the main deployment)
3. That's it. Both workflows are already scheduled and will start running
   automatically. You can trigger either one manually from the repo's
   **Actions** tab (each has a "Run workflow" button) to confirm they work
   without waiting for the schedule.

## What's in here

- `.github/workflows/ping.yml` — runs every 10 minutes, hits your Render
  backend's `/health` endpoint. This is the actual keep-alive mechanism.
- `.github/workflows/keep-repo-active.yml` — runs once a month, makes a
  trivial commit. Exists only so GitHub never sees 60 days of silence in
  this repo and auto-disables the schedule above.

## Checking it's working

The Actions tab shows a history of every run of both workflows. If
`ping.yml` starts silently failing (network issue, Render URL changed,
etc.), GitHub will email the repo owner after a few consecutive failures
by default — worth confirming that notification setting is on
(Settings → Notifications) since this repo has no other reason for you to
check in on it.
