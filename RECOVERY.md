# Atlas Recovery Runbook

If Atlas (OpenClaw) goes dark, follow these steps in order.

## Quick Check

1. Visit: https://openclaw-railway-template-new-production-d980.up.railway.app/setup
   - If it loads: the wrapper is up but gateway failed. Try "Run Doctor" if available.
   - If it doesn't load: the whole container is down.

## Step 1: Check Railway Dashboard

Go to https://railway.app → jubilant-appreciation project → production environment.

- Check if the service is running or crashed
- Check deploy logs for errors

## Step 2: Redeploy (Fixes 90% of Issues)

In Railway dashboard:
1. Click into the OpenClaw service
2. Go to **Deployments** tab
3. Click **Redeploy** on the active deployment

This triggers the pre-deploy step which clears stale lock files and rebuilds cleanly.
Build takes ~2-3 minutes. Be patient.

## Step 3: Check Logs After Redeploy

After redeploy, check deploy logs. You should see:


If logs stop after `configured: true` without the `[gateway]` line, the gateway is failing silently.
This usually means a config issue. Go to /setup and run the wizard/doctor.

## Common Issues

### Stale Lock File
- **Symptom:** Gateway doesn't start after crash recovery
- **Cause:** Lock file from previous process not cleaned up
- **Fix:** Redeploy (pre-deploy script clears it)

### Gateway Silently Fails
- **Symptom:** Wrapper says `configured: true` but no `[gateway]` logs
- **Cause:** Config issue or missing API keys
- **Fix:** Visit /setup, check config, run doctor

### Config Patch Crash
- **Symptom:** Atlas dies right after a config change
- **Cause:** SIGUSR1 restart can be fragile
- **Fix:** Redeploy. The config change is already saved to disk.

## Architecture

- **Railway repo:** https://github.com/LevityLeads/clawdbot-railway-template
- **Forked from:** vignesh07/clawdbot-railway-template
- **Restart policy:** always (set in railway.toml)
- **Healthcheck:** /setup/healthz (300s timeout)
- **Persistent volume:** Mounted at /data (config, workspace, memory all survive redeploys)
- **Updates:** Done via /update command inside Atlas, NOT via git. The repo is just the build template.

## If All Else Fails

1. Check Railway status page: https://status.railway.app
2. Try a fresh deploy from the same repo (data is on the volume, won't be lost)
3. Message OpenClaw Discord for help: https://discord.com/invite/clawd
