# Vercel Postgres Setup for Logging

This guide walks you through setting up Vercel Postgres for persistent logging.

## 🎯 Why Vercel Postgres?

- ✅ **Persistent storage** - Logs never get lost
- ✅ **Integrated with Vercel** - Works seamlessly
- ✅ **Free tier available** - 256 MB storage, 60 hours compute/month
- ✅ **Easy to query** - Standard SQL
- ✅ **Automatic backups** - Vercel handles it

## 📋 Setup Steps

### Step 1: Create Vercel Postgres Database

1. Go to your Vercel dashboard: https://vercel.com/dashboard
2. Select your project (`rushilcs`)
3. Go to the **Storage** tab
4. Click **Create Database**
5. Select **Postgres**
6. Choose a name (e.g., `rushilcs-logs`)
7. Select a region (choose closest to your users)
8. Click **Create**

### Step 2: Link Database to Your Project

1. After creating the database, you'll see connection details
2. Vercel automatically creates environment variables:
   - `POSTGRES_URL`
   - `POSTGRES_PRISMA_URL`
   - `POSTGRES_URL_NON_POOLING`
3. These are automatically available to your serverless functions

### Step 3: Install @vercel/postgres Package

Add the package to your `package.json`:

```bash
cd client
npm install @vercel/postgres
```

Or add it manually to `package.json`:

```json
{
  "dependencies": {
    "@vercel/postgres": "^0.5.1"
  }
}
```

Then run:
```bash
npm install
```

### Step 4: Initialize Database Tables

The tables will be created automatically on first use, but you can also initialize them manually:

1. Go to your Vercel dashboard
2. Select your project → **Storage** → Your Postgres database
3. Click **Data** tab
4. You can run SQL queries here, or the tables will auto-create on first log

Alternatively, create an initialization endpoint (optional):

```javascript
// api/init-db.js (optional - for manual initialization)
import { initLogTables } from './logger.js';

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    await initLogTables();
    return res.status(200).json({ success: true, message: 'Tables initialized' });
  } catch (error) {
    return res.status(500).json({ error: error.message });
  }
}
```

### Step 5: Deploy

1. Commit and push your changes:
   ```bash
   git add .
   git commit -m "Add Vercel Postgres logging"
   git push
   ```

2. Vercel will automatically deploy
3. The database tables will be created on first API call

## ✅ Verify Setup

1. **Make a test request:**
   - Generate a plan on your site
   - Send a chatbot message

2. **Check logs:**
   ```bash
   curl "https://rushilcs.vercel.app/api/admin-logs?key=YOUR_ADMIN_KEY"
   ```

3. **View in Vercel Dashboard:**
   - Go to **Storage** → Your Postgres database → **Data** tab
   - You should see `plan_generator_logs` and `chatbot_logs` tables
   - Click on a table to see the data

## 📊 Viewing Logs

### Option 1: Admin API Endpoint

```bash
# All logs
curl "https://rushilcs.vercel.app/api/admin-logs?key=YOUR_ADMIN_KEY"

# Only plan generator logs
curl "https://rushilcs.vercel.app/api/admin-logs?key=YOUR_ADMIN_KEY&type=plan"

# Only chatbot logs
curl "https://rushilcs.vercel.app/api/admin-logs?key=YOUR_ADMIN_KEY&type=chatbot"

# With statistics
curl "https://rushilcs.vercel.app/api/admin-logs?key=YOUR_ADMIN_KEY&stats=true"

# Limit results (default: 1000)
curl "https://rushilcs.vercel.app/api/admin-logs?key=YOUR_ADMIN_KEY&limit=100"
```

### Option 2: Vercel Dashboard

1. Go to **Storage** → Your Postgres database
2. Click **Data** tab
3. Select a table (`plan_generator_logs` or `chatbot_logs`)
4. View, filter, and export data

### Option 3: SQL Queries

In Vercel dashboard → Storage → Your database → **Query** tab:

```sql
-- Get recent plan generator logs
SELECT * FROM plan_generator_logs 
ORDER BY timestamp DESC 
LIMIT 10;

-- Get recent chatbot logs
SELECT * FROM chatbot_logs 
ORDER BY timestamp DESC 
LIMIT 10;

-- Count logs by day
SELECT DATE(timestamp) as date, COUNT(*) as count
FROM plan_generator_logs
GROUP BY DATE(timestamp)
ORDER BY date DESC;

-- Get logs with errors
SELECT * FROM plan_generator_logs 
WHERE error IS NOT NULL
ORDER BY timestamp DESC;
```

## 🔍 Database Schema

### `plan_generator_logs` Table

| Column | Type | Description |
|--------|------|-------------|
| id | SERIAL | Primary key |
| timestamp | TIMESTAMP | When the log was created |
| company_name | TEXT | Company name from user input |
| job_description | TEXT | Full job description |
| job_description_length | INTEGER | Length of job description |
| is_url | BOOLEAN | Whether input was a URL |
| plan | TEXT | Generated plan (full text) |
| plan_length | INTEGER | Length of plan |
| job_fit | TEXT | Job fit analysis |
| job_fit_length | INTEGER | Length of job fit |
| metadata | JSONB | Model metadata (tokens, cost, etc.) |
| error | TEXT | Error message (if any) |

### `chatbot_logs` Table

| Column | Type | Description |
|--------|------|-------------|
| id | SERIAL | Primary key |
| timestamp | TIMESTAMP | When the log was created |
| message | TEXT | User message |
| message_length | INTEGER | Length of message |
| conversation_history_length | INTEGER | Number of previous messages |
| response | TEXT | Bot response |
| response_length | INTEGER | Length of response |
| metadata | JSONB | Model metadata |
| error | TEXT | Error message (if any) |

## 💾 Storage Limits

**Free Tier:**
- 256 MB storage
- 60 hours compute/month
- Should be plenty for logging thousands of interactions

**If you exceed limits:**
- Upgrade to Pro ($20/month) for more storage
- Or export old logs and delete them

## 🗑️ Managing Old Logs

To delete old logs (keep last 1000):

```sql
-- Delete old plan generator logs (keep last 1000)
DELETE FROM plan_generator_logs
WHERE id NOT IN (
  SELECT id FROM plan_generator_logs
  ORDER BY timestamp DESC
  LIMIT 1000
);

-- Delete old chatbot logs (keep last 1000)
DELETE FROM chatbot_logs
WHERE id NOT IN (
  SELECT id FROM chatbot_logs
  ORDER BY timestamp DESC
  LIMIT 1000
);
```

## 🔒 Security

- Database is only accessible from your Vercel serverless functions
- Admin endpoint is protected by `ADMIN_LOG_KEY`
- Connection strings are automatically managed by Vercel
- No public access to database

## 🚀 Next Steps

1. Create Postgres database in Vercel
2. Install `@vercel/postgres` package
3. Deploy your code
4. Test by making some requests
5. View logs via admin endpoint or Vercel dashboard

Your logs are now persistent and will never be lost! 🎉
