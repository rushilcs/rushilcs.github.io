# Logging Setup for Vercel Deployment

This guide explains how to set up and access logs for your deployed Vercel site.

## ✅ What's Already Done

- ✅ Logging code is integrated into both API endpoints
- ✅ Logs are automatically saved for every interaction
- ✅ Admin endpoint is created to view logs

## 🔧 Setup Steps

### 1. Add Admin Key to Vercel

1. Go to your Vercel dashboard: https://vercel.com/dashboard
2. Select your project (`rushilcs`)
3. Go to **Settings** → **Environment Variables**
4. Add a new variable:
   - **Name:** `ADMIN_LOG_KEY`
   - **Value:** (choose a strong secret key, e.g., generate one at https://randomkeygen.com/)
   - **Environment:** Select all (Production, Preview, Development)
5. Click **Save**

### 2. Redeploy Your Site

After adding the environment variable, you need to redeploy:

1. In Vercel dashboard, go to **Deployments**
2. Click the **⋯** menu on your latest deployment
3. Click **Redeploy**
4. Wait for deployment to complete (~2 minutes)

## 📊 Viewing Logs

### Option 1: Via Browser/curl

```bash
# View all logs
curl "https://rushilcs.vercel.app/api/admin-logs?key=YOUR_ADMIN_KEY"

# View only plan generator logs
curl "https://rushilcs.vercel.app/api/admin-logs?key=YOUR_ADMIN_KEY&type=plan"

# View only chatbot logs
curl "https://rushilcs.vercel.app/api/admin-logs?key=YOUR_ADMIN_KEY&type=chatbot"
```

### Option 2: Save to File

```bash
# Save all logs to a file
curl "https://rushilcs.vercel.app/api/admin-logs?key=YOUR_ADMIN_KEY" > logs.json

# Pretty print with jq (if installed)
curl "https://rushilcs.vercel.app/api/admin-logs?key=YOUR_ADMIN_KEY" | jq '.' > logs.json
```

### Option 3: Browser

Just open this URL in your browser (replace `YOUR_ADMIN_KEY`):
```
https://rushilcs.vercel.app/api/admin-logs?key=YOUR_ADMIN_KEY
```

## 📝 What Gets Logged

### Plan Generator Logs
- User inputs (company name, job description)
- Generated plan (full text)
- Job fit analysis
- Metadata (model used, tokens, latency, cost)
- Errors (if any)

### Chatbot Logs
- User messages
- Bot responses
- Conversation history length
- Errors (if any)

## ⚠️ Important Notes

### Vercel `/tmp` Directory Limitations

- Logs are stored in `/tmp/logs/` on Vercel
- `/tmp` is **ephemeral** - logs are lost when:
  - Function goes cold (after ~10 minutes of inactivity)
  - New deployment happens
  - Function is restarted

### Recommendations

1. **Export logs regularly:**
   - Set up a cron job or scheduled task to fetch logs daily
   - Or manually export logs periodically

2. **For persistent storage, consider:**
   - **Vercel Postgres** (free tier available)
   - **Supabase** (free tier available)
   - **MongoDB Atlas** (free tier available)
   - **Airtable** (free tier available)

3. **For production use:**
   - I can help you migrate to a database-based logging system
   - This would require updating the logger to use HTTP APIs instead of file system

## 🔍 Testing

1. Make a test request to your plan generator
2. Send a test message to your chatbot
3. Wait a few seconds
4. Fetch logs using the admin endpoint
5. You should see your test interactions in the logs

## 🛡️ Security

- The admin endpoint is protected by `ADMIN_LOG_KEY`
- Only you know the key
- Logs are not accessible to the public
- Keep your admin key secret!

## 📈 Log Format

Each log entry looks like:

```json
{
  "timestamp": "2024-01-15T10:30:00.000Z",
  "userInput": {
    "companyName": "Example Corp",
    "jobDescription": "...",
    "jobDescriptionLength": 1234,
    "isUrl": false
  },
  "modelOutput": {
    "plan": "...",
    "planLength": 5678,
    "jobFit": "...",
    "jobFitLength": 234
  },
  "metadata": {
    "latency": 2345,
    "model": "gpt-4o",
    "tokens": {
      "input": 1000,
      "output": 500,
      "total": 1500
    },
    "cost": 0.05
  },
  "error": null
}
```

## 🚀 Next Steps

1. Add `ADMIN_LOG_KEY` to Vercel environment variables
2. Redeploy your site
3. Test the logging by making some requests
4. Fetch logs to verify they're working
5. (Optional) Set up automated log exports for persistence
