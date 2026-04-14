# 🌐 Hosting a Static Portfolio on AWS — S3 + CloudFront + Route 53

> **Full production-grade setup** | Private S3 → CloudFront CDN → Custom Domain → HTTPS
> Built for: [zoheb.site](https://zoheb.site)

---

## 📋 Table of Contents

| Phase | What You'll Do |
|-------|----------------|
| [Phase 1 — S3 Bucket Setup](#-phase-1--s3-bucket-setup) | Create bucket, upload portfolio files |
| [Phase 2 — Route 53 Hosted Zone](#-phase-2--route-53-hosted-zone) | Set up DNS, point GoDaddy to AWS |
| [Phase 3 — ACM SSL Certificate](#-phase-3--acm-ssl-certificate) | Issue HTTPS cert (must be us-east-1) |
| [Phase 4 — CloudFront Distribution](#-phase-4--cloudfront-distribution) | CDN with OAC, HTTPS redirect, caching |
| [Phase 5 — Route 53 Alias Records](#-phase-5--route-53-alias-records) | Wire domain to CloudFront |
| [Phase 6 — Test Everything](#-phase-6--test-everything) | Verify all endpoints work |
| [Phase 7 — Updating Content](#-phase-7--updating-content) | Cache invalidation + CLI automation |
| [Troubleshooting](#-troubleshooting) | Common errors and exact fixes |

---

## Architecture Overview

```
Browser
  │
  ▼
Route 53 (zoheb.site A Alias)
  │
  ▼
CloudFront Distribution (HTTPS, edge cache)
  │  ← Origin Access Control (OAC)
  ▼
S3 Bucket (private, zoheb.site)
```

---

## 🗂️ Phase 1 — S3 Bucket Setup

### 1.1 Create the Bucket

1. Log in → [console.aws.amazon.com](https://console.aws.amazon.com/)
2. Search bar → `S3` → click **S3** under Services
3. Click **Create bucket** (orange, top right)

Fill in the form exactly:

| Field | Value |
|-------|-------|
| **Bucket name** | `zoheb.site` *(must exactly match your domain)* |
| **AWS Region** | `ap-south-1` — Asia Pacific (Mumbai) |
| **Object Ownership** | ACLs disabled *(leave default)* |
| **Block Public Access** | ✅ Keep ALL checked — CloudFront uses OAC, not public access |
| **Bucket Versioning** | Disabled *(for now)* |
| **Default encryption** | Leave as is |

4. Click **Create bucket**

---

### 1.2 Upload Your Portfolio Files

1. Click **zoheb.site** in the bucket list
2. Click **Upload** → **Add files** → select `index.html`, CSS, JS, images
3. Click **Add folder** for any subdirectories (`assets/`, `css/`, `js/`)

> ⚠️ **Critical:** `index.html` must be at the **root level** — not inside any subfolder

4. Scroll down → Click **Upload**
5. Wait for the ✅ green success bar

---

## 🌐 Phase 2 — Route 53 Hosted Zone

### 2.1 Create the Hosted Zone

1. Search bar → `Route 53` → click it
2. Left sidebar → **Hosted zones**
3. Click **Create hosted zone**

| Field | Value |
|-------|-------|
| **Domain name** | `zoheb.site` *(no www, no https)* |
| **Type** | ✅ Public hosted zone |

4. Click **Create hosted zone**

You'll see 2 pre-created records: **NS** and **SOA** — this is expected.

---

### 2.2 Copy the AWS Nameservers

1. Click the **NS record** in the list
2. Copy all 4 values — they look like:

```
ns-123.awsdns-45.com.
ns-456.awsdns-67.net.
ns-789.awsdns-01.co.uk.
ns-012.awsdns-23.org.
```

> ⚠️ Remove the **trailing dot `.`** from each before pasting into GoDaddy

---

### 2.3 Update Nameservers in GoDaddy

1. [godaddy.com](https://godaddy.com/) → Sign in
2. Avatar (top right) → **My Products**
3. Find `zoheb.site` → click **DNS**
4. Scroll to **Nameservers** → **Change Nameservers**
5. Select **"I'll use my own nameservers"**
6. Delete GoDaddy's existing nameservers
7. Paste all 4 Route 53 nameservers
8. Click **Save** → click **Continue** on the warning

> ⏱️ **Wait 15–60 minutes.** Verify propagation at [dnschecker.org](https://dnschecker.org/) — search `zoheb.site` with type `NS`

---

## 🔒 Phase 3 — ACM SSL Certificate

### 3.1 Switch Region to `us-east-1` — MANDATORY

> Top-right region dropdown → select **US East (N. Virginia) `us-east-1`**

> ⚠️ **Non-negotiable.** CloudFront only reads ACM certificates from `us-east-1`. A cert in Mumbai will NOT work.

---

### 3.2 Request the Certificate

1. Search bar → `Certificate Manager` → click it
2. Click **Request a certificate**
3. Select **Request a public certificate** → **Next**

| Field | Value |
|-------|-------|
| **Domain 1** | `zoheb.site` |
| **Domain 2** | `www.zoheb.site` *(click "Add another name")* |
| **Validation method** | ✅ DNS validation |
| **Key algorithm** | RSA 2048 *(leave default)* |

4. Click **Request**

---

### 3.3 Validate Domain Ownership

1. Click the certificate ID showing **"Pending validation"**
2. In **Domains** section → click **"Create records in Route 53"**
3. Popup appears → click **Create records**

AWS automatically adds CNAME validation records to your hosted zone.

4. Refresh every 1–2 minutes → status changes: `Pending validation` → ✅ **Issued** *(usually 2–5 min)*

---

## ☁️ Phase 4 — CloudFront Distribution

### 4.1 Create the Distribution

Search bar → `CloudFront` → **Create a CloudFront distribution**

---

### 4.2 Configure the Origin (S3)

| Setting | Value |
|---------|-------|
| **Origin domain** | `zoheb.site.s3.amazonaws.com` *(dropdown — pick plain S3 endpoint, NOT website endpoint)* |
| **Origin path** | Leave blank |
| **Origin access** | **Origin access control settings (recommended)** |

Click **Create new OAC**:

| Field | Value |
|-------|-------|
| **Name** | `zoheb-site-oac` |
| **Signing behavior** | Sign requests (recommended) |

Click **Create** → your OAC is selected

---

### 4.3 Configure Default Cache Behavior

| Setting | Value |
|---------|-------|
| **Compress objects automatically** | ✅ Yes |
| **Viewer protocol policy** | **Redirect HTTP to HTTPS** |
| **Allowed HTTP methods** | GET, HEAD |
| **Cache key and origin requests** | CachingOptimized *(leave default)* |

---

### 4.4 Configure Settings

| Setting | Value |
|---------|-------|
| **WAF** | Do not enable *(saves cost for a portfolio)* |
| **Price class** | Use all edge locations *(fastest, includes India)* |
| **Alternate domain names (CNAME)** | `zoheb.site` and `www.zoheb.site` |
| **Custom SSL certificate** | Select your ACM cert `zoheb.site` from dropdown |
| **Security policy** | TLSv1.2_2021 |
| **HTTP versions** | ✅ HTTP/2 and HTTP/3 |
| **Default root object** | `index.html` |

Click **Create distribution**

> ⏱️ Wait **5–10 minutes** for status: `Deploying` → ✅ `Enabled`

---

### 4.5 Fix S3 Bucket Policy ⚠️ Critical Step

After creating, CloudFront shows a **yellow banner**: *"The S3 bucket policy needs to be updated"*

1. Click **Copy policy** in that banner
2. Open new tab → **S3** → `zoheb.site` bucket → **Permissions** tab
3. Scroll to **Bucket policy** → **Edit**
4. Delete all existing content
5. Paste the copied JSON → **Save changes**

The policy should look like:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::zoheb.site/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::YOUR_ACCOUNT_ID:distribution/YOUR_DIST_ID"
        }
      }
    }
  ]
}
```

---

## 🔗 Phase 5 — Route 53 Alias Records

### 5.1 Root Domain (`zoheb.site`)

Route 53 → Hosted zones → `zoheb.site` → **Create record**

| Field | Value |
|-------|-------|
| **Record name** | *(leave completely blank — this is the root)* |
| **Record type** | A |
| **Alias** | Toggle **ON** |
| **Route traffic to** | Alias to CloudFront distribution |
| **Distribution** | Select your CloudFront domain (e.g. `d1234abcd.cloudfront.net`) |
| **Routing policy** | Simple routing |

Click **Create records**

---

### 5.2 WWW Subdomain (`www.zoheb.site`)

Click **Create record** again:

| Field | Value |
|-------|-------|
| **Record name** | `www` |
| **Record type** | A |
| **Alias** | Toggle **ON** |
| **Route traffic to** | Alias to same CloudFront distribution |

Click **Create records**

---

## ✅ Phase 6 — Test Everything

| URL | Expected Result |
|-----|-----------------|
| `https://zoheb.site` | Portfolio loads with 🔒 padlock |
| `http://zoheb.site` | Auto-redirects to `https://` |
| `https://www.zoheb.site` | Also loads correctly |
| Click 🔒 padlock | Shows "Connection is secure" + Amazon certificate |
| Phone on 4G | Loads correctly (confirms global DNS + CDN) |

---

## 🔄 Phase 7 — Updating Content

Every time you push new files, old files are cached in CloudFront. You **must invalidate the cache**:

1. CloudFront → Your distribution → **Invalidations** tab
2. Click **Create invalidation**
3. In "Object paths" box, type `/*`
4. Click **Create invalidation**
5. Wait ~30 seconds → live site serves new files

### 💡 Automate with AWS CLI

```bash
# Upload all portfolio files to S3
aws s3 sync ./your-portfolio-folder s3://zoheb.site --delete

# Invalidate CloudFront cache (replace DIST_ID with your distribution ID)
aws cloudfront create-invalidation --distribution-id DIST_ID --paths "/*"
```

> This is the **foundation for a full CI/CD pipeline** — wire these two commands into GitHub Actions and every `git push` deploys your live portfolio automatically.

---

## 🛠️ Troubleshooting

### ❌ CloudFront not showing in Route 53 dropdown

The 3 root causes — in order of likelihood:

**Cause 1 — Missing Alternate Domain Names (most common)**

CloudFront won't appear in the dropdown if `zoheb.site` and `www.zoheb.site` were not added as CNAMEs inside the distribution.

Fix:
1. CloudFront → Your distribution ID → **General** tab → **Edit**
2. Scroll to **Alternate domain name (CNAME)**
3. Add `zoheb.site` and `www.zoheb.site`
4. Ensure ACM certificate is selected
5. **Save changes** → retry Route 53 dropdown

---

**Cause 2 — Distribution still deploying**

CloudFront won't appear until status is `Enabled`.

Fix: Wait until **Status = Enabled**, then retry.

---

**Cause 3 — Stale Route 53 page**

Fix: Hard refresh (`Ctrl+Shift+R`) → close the Create record panel → reopen it.

**Last resort — Type the domain manually:**
1. CloudFront → General tab → copy `d1234abcdefgh.cloudfront.net`
2. Route 53 → Create record → Alias ON → paste the domain manually

---

### ❌ Access Denied on CloudFront URL

**Root cause:** The S3 bucket policy from the OAC was never saved into the bucket.

Fix — 3 steps:
1. CloudFront → **Origins** tab → click your origin → **Edit** → copy the policy from the yellow banner
2. S3 → `zoheb.site` → **Permissions** → **Bucket policy** → paste and save
3. Test `https://d1234abcdef.cloudfront.net` directly — if it loads, S3 is fixed

**Remaining checklist if still failing:**

| Check | Fix |
|-------|-----|
| `index.html` not at S3 root | Re-upload — must not be inside a subfolder |
| Default root object missing | CloudFront → General → Edit → add `index.html` |
| CloudFront still deploying | Wait for status `Enabled` |
| Accessing S3 URL directly | Always use the CloudFront domain, never S3 URL |

---

## 📎 Quick Reference

| Service | Purpose | Region |
|---------|---------|--------|
| S3 | Store portfolio files (private) | `ap-south-1` |
| ACM | SSL/TLS certificate | `us-east-1` ⚠️ |
| CloudFront | CDN + HTTPS termination | Global |
| Route 53 | DNS + Alias records | Global |

---

*Built by [Zoheb Ashar](https://zoheb.site) · AWS S3 + CloudFront + Route 53 + ACM*
