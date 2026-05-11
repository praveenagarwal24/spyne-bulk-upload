# Spyne Bulk VIN Upload

A browser-based tool for uploading bulk VINs and images to the Spyne console — no server, no installs. Just open the URL.

---

## 🚀 Live on GitHub Pages

Once deployed, your team accesses it at:
```
https://<your-org>.github.io/<repo-name>/
```

---

## Setup (5 minutes)

### 1. Create the repo

1. Go to [github.com/new](https://github.com/new)
2. Name it `spyne-bulk-upload` (or anything you like)
3. Set visibility to **Private** (recommended — contains your workflow)
4. Click **Create repository**

### 2. Upload the files

Upload both files from this folder:
- `index.html`
- `sample.csv`
- `README.md`

You can drag-and-drop them directly in the GitHub UI, or use git:

```bash
git clone https://github.com/<your-org>/spyne-bulk-upload
cd spyne-bulk-upload
# copy index.html, sample.csv, README.md here
git add .
git commit -m "init: bulk VIN upload tool"
git push
```

### 3. Enable GitHub Pages

1. Go to your repo → **Settings** → **Pages**
2. Under **Source**, select `main` branch, `/ (root)` folder
3. Click **Save**
4. Wait ~60 seconds, then visit `https://<your-org>.github.io/spyne-bulk-upload/`

---

## How to use

### Step 1 — Configure credentials

| Field | What to enter |
|---|---|
| **Bearer token** | Copy from the `Authorization` header in the Spyne console (open DevTools → Network tab → any request → copy the `authorization` value) |
| **Account ID** | The UUID prefix in your image file paths, e.g. `09d472cd-c057-4f93-9bc9-1bb9d4a251b6` (appears before `_test/` in filenames) |
| **Environment** | `Test` for staging, `Production` for live |

The tool will auto-decode your bearer token and show your `enterprise_id` and `team_id` for verification.

---

### Step 2 — Upload CSV

Prepare a `.csv` with these 4 columns (order doesn't matter):

| Column | Description | Example |
|---|---|---|
| `enterprise_id` | Your enterprise ID | `TaD1VC1Ko` |
| `team_id` | Your team ID | `dfd6464a-b` |
| `dealer_group` | Dealer group name (folder in path) | `Koons_01` |
| `vin` | Vehicle VIN | `1FM5K8F80GGB42986` |

**Example CSV:**
```csv
enterprise_id,team_id,dealer_group,vin
TaD1VC1Ko,dfd6464a-b,Koons_01,1FM5K8F80GGB42986
TaD1VC1Ko,dfd6464a-b,Koons_01,1C6RRFJG6MN627703
EnterpriseB,team-xyz,Metro_02,2T1BURHE0JC034466
```

Multiple enterprises and teams in the same file are fully supported.

Use the included `sample.csv` as a starting template.

---

### Step 3 — Review VINs

Filter by enterprise or team, search by VIN. Verify everything looks correct before proceeding.

---

### Step 4 — Attach images

Click **+ add** next to each VIN to select JPEG/PNG image files. You can attach multiple images per VIN (exterior, interior, focus shots, etc.).

VINs with no images attached will be skipped during upload.

---

### Step 5 — Submit

Click **Start upload**. The tool will:

1. Call `POST /console/v1/util/gen-presigned` with the image list for each VIN
2. Receive pre-signed S3 URLs from Spyne
3. `PUT` each image directly to S3 via the presigned URL
4. Show real-time progress and a per-VIN log

When done, you can **download a results CSV** showing success/failure per VIN.

---

## File path structure

Images are uploaded to:
```
{accountId}_{env}/{dealer_group}/{vin}/{generated_filename}x.jpg
```

Example:
```
09d472cd-c057-4f93-9bc9-1bb9d4a251b6_test/Koons_01/1FM5K8F80GGB42986/upload_abc123_def456x.jpg
```

---

## Sharing with teammates

Just send them the GitHub Pages URL. They need:
- The GitHub Pages URL (from you)
- Their own Bearer token (from Spyne console)
- Their Account ID
- A CSV with their VINs
- The image files

No login, no account needed for the tool itself.

---

## Troubleshooting

| Issue | Fix |
|---|---|
| `gen-presigned` returns 401 | Bearer token is expired — get a fresh one from DevTools |
| `gen-presigned` returns 403 | Token doesn't have access to that enterprise/team |
| S3 upload fails | Presigned URL expired (>15 min elapsed) — re-run |
| CORS error | Your browser blocks cross-origin requests — try Chrome |
| Missing columns error | Check your CSV header row has all 4 required columns |

---

## Security notes

- The bearer token is **never stored** — it lives only in the browser tab's memory and is gone when you close the tab
- The repo can be **private** — GitHub Pages works for private repos on paid plans, or keep it public if the tool itself has no secrets
- Never commit actual bearer tokens or account IDs to the repo

---

## API reference

This tool calls the following Spyne endpoint:

```
POST https://api.spyne.ai/console/v1/util/gen-presigned

Headers:
  Authorization: Bearer <token>
  Content-Type: application/json

Body:
{
  "imageList": [
    { "fileName": "<path>", "fileType": "image/jpeg" },
    ...
  ]
}
```

Followed by `PUT` to each returned presigned S3 URL with the image binary as the body.
