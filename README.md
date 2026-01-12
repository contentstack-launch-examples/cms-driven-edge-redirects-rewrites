# CMS-Driven Edge Redirects & Rewrites

A Next.js application that demonstrates edge-level redirects and rewrites powered by Contentstack CMS, designed to be hosted on Contentstack Launch. This solution allows content editors to manage URL redirects and rewrites directly from the CMS without any code changes.

## 🎯 Overview

This project implements a CMS-driven redirect/rewrite system where:

- **No code changes required** - Redirects and rewrites are managed entirely in Contentstack
- **Edge-level performance** - Rules execute at the edge before reaching the origin
- **Content editor friendly** - Marketing teams can manage redirects without developer help
- **Build-time injection** - Rules are fetched from CMS and injected into edge functions during build

## 🏗️ Architecture

This solution uses **Contentstack CMS to store redirect/rewrite rules** that are fetched at build time and injected into edge functions:

- ✅ **Edge performance** - Redirects execute at CDN edge, not at origin
- ✅ **No database needed** - Rules stored in Contentstack CMS
- ✅ **Version controlled** - Rules follow Contentstack's publish workflow
- ✅ **Automatic updates** - New deployments fetch latest rules from CMS

### How It Works

1. **Content editors** create redirect/rewrite entries in Contentstack
2. **Build script** fetches all rules from Contentstack via REST API
3. **Edge function** is auto-generated with rules embedded
4. **Contentstack Launch** deploys and executes edge function on all requests

```
┌─────────────────────┐
│  Add/Update entry   │
│  in Contentstack    │
└──────────┬──────────┘
           │ (deploy)
           ▼
┌─────────────────────┐
│  Build fetches      │
│  rules from CMS     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Edge function      │
│  auto-generated     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Redirects/Rewrites │
│  execute at edge    │
└─────────────────────┘
```

## 🚀 Getting Started

### 1. Install Dependencies

```bash
npm install
```

### 2. Set Up Environment Variables

Create a `.env.local` file in the root directory:

```env
CONTENTSTACK_API_KEY=your_api_key
CONTENTSTACK_DELIVERY_TOKEN=your_delivery_token
CONTENTSTACK_ENVIRONMENT=your_environment
CONTENTSTACK_APP_HOST=cdn.contentstack.io

# Content type UIDs (optional - defaults shown)
REDIRECT_CT=redirect
REWRITE_CT=rewrite

# Optional: For Live Preview
NEXT_PUBLIC_CONTENTSTACK_API_KEY=your_api_key
NEXT_PUBLIC_CONTENTSTACK_DELIVERY_TOKEN=your_delivery_token
NEXT_PUBLIC_CONTENTSTACK_ENVIRONMENT=your_environment
NEXT_PUBLIC_CONTENTSTACK_MANAGEMENT_TOKEN=your_management_token
NEXT_PUBLIC_CONTENTSTACK_PREVIEW_HOST=your_preview_host
NEXT_PUBLIC_CONTENTSTACK_LIVE_PREVIEW_HOST=your_live_preview_host
NEXT_PUBLIC_CONTENTSTACK_APP_HOST=your_app_host
```

### 3. Build Rules

```bash
# Fetch rules and generate edge function
npm run build:rules

# Or run dev (automatically runs prebuild)
npm run dev
```

## 📦 Contentstack Setup

### Content Types Required

#### 1. Redirect Content Type

Create a content type named **"redirect"** with the following fields:

| Field UID     | Display Name | Type            | Required | Description                          |
| ------------- | ------------ | --------------- | -------- | ------------------------------------ |
| `title`       | Title        | Text            | Yes      | Name of the redirect rule            |
| `source`      | Source       | Text            | Yes      | Source path (e.g., `/old-page`)      |
| `destination` | Destination  | Text            | Yes      | Destination path (e.g., `/new-page`) |
| `statuscode`  | Status Code  | Number/Dropdown | No       | HTTP status: 301, 302, 307, 308      |

**Status Code Options:**

- `301` - Permanent Redirect (default)
- `302` - Temporary Redirect
- `307` - Temporary Redirect (preserves method)
- `308` - Permanent Redirect (preserves method)

#### 2. Rewrite Content Type (Optional)

Create a content type named **"rewrite"** with the following fields:

| Field UID     | Display Name | Type | Required | Description                      |
| ------------- | ------------ | ---- | -------- | -------------------------------- |
| `title`       | Title        | Text | Yes      | Name of the rewrite rule         |
| `source`      | Source       | Text | Yes      | Source path (e.g., `/api/proxy`) |
| `destination` | Destination  | Text | Yes      | Destination URL to proxy to      |

### Creating Redirect Entries

1. **Go to Contentstack** → Your Stack → Entries
2. **Create new entry** in the "redirect" content type
3. **Fill in the fields**:
   - `source`: `/about` (the old URL)
   - `destination`: `/contact` (the new URL)
   - `statuscode`: `301`
4. **Publish the entry**
5. **Trigger a new deployment** on Contentstack Launch

The redirect will be active after deployment!

### Example Redirects

| Source        | Destination     | Status Code | Description        |
| ------------- | --------------- | ----------- | ------------------ |
| `/old-blog`   | `/blog`         | 301         | Permanent redirect |
| `/temp-promo` | `/sale`         | 302         | Temporary redirect |
| `/legacy/*`   | `/new/*`        | 301         | Wildcard redirect  |
| `/external`   | `https://x.com` | 302         | External redirect  |

### Automation Setup (Auto-Redeploy)

Set up Contentstack Automation to automatically redeploy your Launch project when redirect entries are published or updated:

1. **Go to Contentstack** → Your Stack → Automation Hub
2. **Create new automation** with:
   - **Trigger**: Entry Published / Entry Updated
   - **Content Type**: Select `redirect` (and `rewrite` if using)
3. **Add Action**: Trigger Contentstack Launch Deployment
   - Select your Launch project
4. **Save and Enable** the automation

Now when you publish or update any redirect entry, the project automatically redeploys with the latest rules!

## 🔄 How Edge Redirects Work

### Example: `/about` → `/contact`

1. **User visits** `https://yourdomain.com/about`
2. **Edge function executes** before reaching Next.js
3. **Matches redirect rule**: `source: /about` → `destination: /contact`
4. **Returns 301 response** with `Location: /contact` header
5. **Browser redirects** to `/contact`
6. **Next.js serves** the `/contact` page

## 🚢 Deployment to Contentstack Launch

### 1. Connect Your Repository

1. Go to Contentstack Launch dashboard
2. Connect your Git repository
3. Configure build settings:
   - **Build Command**: `npm run build`
   - **Output Directory**: `.next`
   - **Node Version**: 18.x or higher

### 2. Environment Variables

Add your Contentstack environment variables in Launch:

- `CONTENTSTACK_API_KEY`
- `CONTENTSTACK_DELIVERY_TOKEN`
- `CONTENTSTACK_ENVIRONMENT`
- `CONTENTSTACK_APP_HOST`

### 3. Edge Function

The edge function is automatically generated at `functions/[proxy].edge.js` during the build process. Contentstack Launch detects and deploys this edge function.

### 4. Updating Redirects

To add or update redirects:

1. **Create/Edit** redirect entry in Contentstack
2. **Publish** the entry
3. **Trigger deployment** on Contentstack Launch
4. New redirects are active after deployment completes

## 📁 Project Structure

```
├── app/
│   ├── page.tsx              # Home page
│   ├── about/page.tsx        # About page
│   ├── contact/page.tsx      # Contact page
│   └── services/page.tsx     # Services page
├── functions/
│   └── [proxy].edge.js       # Auto-generated edge function
├── scripts/
│   ├── fetch-rules.js        # Fetches rules from Contentstack
│   └── update-edge-file.js   # Generates edge function
├── lib/
│   └── contentstack.ts       # Contentstack SDK initialization
└── .env.local                # Environment variables
```

## 🛠️ Scripts

| Script                | Description                                    |
| --------------------- | ---------------------------------------------- |
| `npm run dev`         | Start dev server (runs prebuild automatically) |
| `npm run build`       | Build for production                           |
| `npm run prebuild`    | Fetch rules and generate edge function         |
| `npm run build:rules` | Manually regenerate edge function              |

## 📚 Contentstack Documentation

- [Next.js on Contentstack Launch](https://www.contentstack.com/docs/developers/launch/nextjs-on-launch)
- [Edge Functions](https://www.contentstack.com/docs/developers/launch/edge-functions)
- [Content Modeling](https://www.contentstack.com/docs/developers/create-content-types) 
- [Automation](https://www.contentstack.com/docs/developers/automation-hub-guides/create-an-automation)

## 🐛 Troubleshooting

### Redirects Not Working

- **Check entry is published**: Redirect entries must be published in Contentstack
- **Verify deployment**: Ensure a new deployment was triggered after publishing
- **Check build logs**: Look for "✅ Fetched X redirects" in build output

### 404 After Redirect

- **Check destination exists**: Ensure the destination page/route exists
- **Verify path format**: Destination should start with `/` for internal paths

### Edge Function Errors

- **Check environment variables**: Ensure all Contentstack credentials are set in Launch
- **Verify content type UID**: Default is `redirect` - check `REDIRECT_CT` env var if different
