# Firebase Setup for Railway Deployment

## Overview

This Discord bot uses Firebase Admin SDK to manage licenses in Firestore. The code supports **two initialization methods**:

### Method 1: Individual Environment Variables (RECOMMENDED for Railway)
This is the most reliable method for Railway deployments, as it avoids issues with large JSON strings in environment variables.

**Required Variables:**
- `FIREBASE_PROJECT_ID` - Your Firebase project ID
- `FIREBASE_CLIENT_EMAIL` - Service account email
- `FIREBASE_PRIVATE_KEY` - Private key (with literal `\n` for newlines)

### Method 2: JSON Service Account Key (Legacy)
Falls back to a single JSON string if individual variables are not set.

**Required Variable:**
- `FIREBASE_SERVICE_ACCOUNT_KEY` - Complete service account JSON as a string

---

## How to Get Firebase Credentials

### Step 1: Go to Firebase Console
1. Visit [Firebase Console](https://console.firebase.google.com/)
2. Select your project (or create one)
3. Click **Settings** (gear icon) → **Project Settings**

### Step 2: Generate Service Account Key
1. Go to the **Service Accounts** tab
2. Click **Generate New Private Key**
3. A JSON file will download with your credentials

### Step 3: Extract Individual Credentials (Method 1 - Recommended)

Open the downloaded JSON file. It looks like:
```json
{
  "type": "service_account",
  "project_id": "your-project-id",
  "private_key_id": "...",
  "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n",
  "client_email": "firebase-adminsdk-xxxxx@your-project-id.iam.gserviceaccount.com",
  "client_id": "...",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "..."
}
```

Extract these three values:
- **FIREBASE_PROJECT_ID** = `project_id` field
- **FIREBASE_CLIENT_EMAIL** = `client_email` field
- **FIREBASE_PRIVATE_KEY** = `private_key` field (keep the literal `\n` characters)

---

## Setting Variables in Railway

### Via Railway Dashboard

1. Go to your **DiscordBot** service
2. Click **Variables**
3. Add these three variables:

| Variable Name | Value |
|---|---|
| `FIREBASE_PROJECT_ID` | `your-project-id` |
| `FIREBASE_CLIENT_EMAIL` | `firebase-adminsdk-xxxxx@your-project-id.iam.gserviceaccount.com` |
| `FIREBASE_PRIVATE_KEY` | `-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n` |

**Important:** Keep the literal `\n` characters in the private key. Do NOT convert them to actual newlines.

### Via Railway CLI

```bash
railway variable set FIREBASE_PROJECT_ID "your-project-id"
railway variable set FIREBASE_CLIENT_EMAIL "firebase-adminsdk-xxxxx@your-project-id.iam.gserviceaccount.com"
railway variable set FIREBASE_PRIVATE_KEY "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
```

---

## Other Required Variables

Make sure these are also set:

| Variable | Description |
|---|---|
| `DISCORD_TOKEN` | Your Discord bot token |
| `DISCORD_CLIENT_ID` | Your Discord application ID |
| `DISCORD_GUILD_ID` | (Optional) Guild ID for guild-specific commands |
| `DISCORD_ADMIN_ROLE_ID` | (Optional) Role ID for admin-only commands |

---

## Troubleshooting

### "Missing Firebase credentials" Error
- Verify all three variables are set: `FIREBASE_PROJECT_ID`, `FIREBASE_CLIENT_EMAIL`, `FIREBASE_PRIVATE_KEY`
- Check that the private key contains literal `\n` characters (not actual newlines)
- Ensure the service account has Firestore permissions

### "Invalid JSON" Error
- The private key must be a string with literal `\n` characters
- Do not paste the key with actual line breaks

### Firestore Permission Denied
- Go to [Firebase Console](https://console.firebase.google.com/)
- Select your project → **Firestore Database**
- Check that the service account has read/write permissions
- Default rules should allow the service account access

---

## Code Changes

The updated `index.js` now:
1. ✅ Tries Method 1 first (individual variables)
2. ✅ Falls back to Method 2 (JSON key) if Method 1 fails
3. ✅ Provides clear error messages if both fail
4. ✅ Does NOT crash if `.env` file is missing (Railway doesn't use `.env`)
5. ✅ Works reliably with Railway's environment variable system

