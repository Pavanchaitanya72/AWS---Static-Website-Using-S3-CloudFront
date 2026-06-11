# AWS---Static-Website-Using-S3-CloudFront
# CONFIGURATION STEPS


# Secure Static Website Hosting on AWS (S3 + CloudFront)

## 📌 Project Overview
This project demonstrates how to host a secure, high-performance static website on AWS using **Amazon S3** for storage and **Amazon CloudFront** as a Content Delivery Network (CDN). 

By using **Origin Access Control (OAC)**, the S3 bucket remains completely private, forcing all user traffic through CloudFront to ensure HTTPS encryption and global caching.

---

## 🛠️ Architecture Steps (Manual Console Setup)

### Phase 1: Amazon S3 Configuration
1. Created a globally unique S3 bucket (`your-bucket-name`).
2. Kept **Block *all* public access** enabled to ensure data security.
3. Uploaded website core files 'index.html', style.css, script.js to the root of the bucket.

### Phase 2: Amazon CloudFront Configuration
1. Created a new CloudFront Distribution pointing to the S3 bucket origin.
2. Configured **Origin Access Control (OAC)** to securely allow CloudFront to read from the private S3 bucket.
3. Set the **Viewer Protocol Policy** to `Redirect HTTP to HTTPS`.
4. Configured the **Default Root Object** to `index.html`.

### Phase 3: Permissions
1. Copied the auto-generated OAC policy from CloudFront.
2. Attached it to the S3 **Bucket Policy** to grant `s3:GetObject` permissions explicitly to the CloudFront service principal.

---

## 🔍 Troubleshooting & Key Learnings

During deployment, I encountered a classic AWS obstacle and documented the root causes to solidify my cloud infrastructure knowledge:

### 🚨 The Problem: `AccessDenied` XML Error
Upon hitting the raw CloudFront Distribution URL (e.g., `https://dxxxxx.cloudfront.net`), the browser displayed an XML page with an `AccessDenied` error code, even though the S3 bucket policy was perfectly configured.

### 🛠️ The Resolution Workflow
1. **Isolating the Issue:** I appended `/index.html` to the end of the URL (`https://dxxxxx.cloudfront.net/index.html`), and the website loaded perfectly. This proved the S3 bucket policy and network path were working.
2. **Identifying Root Cause 1 (Missing Default Root Object):** Without a specific file requested, CloudFront attempted to list the contents of the S3 bucket root directory. S3 blocks list operations by default for security, throwing an `AccessDenied` error.
   * *Fix:* Updated CloudFront General Settings to set **Default Root Object** to `index.html`.
3. **Identifying Root Cause 2 (Edge Caching):** After updating the configuration, the error persisted because CloudFront cached the initial `AccessDenied` error response at its global edge locations.
   * *Fix:* Created a CloudFront **Invalidation** for `/*` to flush the cache, forcing CloudFront to fetch the fresh routing rule from the origin.

### 💡 Key Concept Takeaway
* **Default Root Object:** Acts as the automatic "homepage filler" when a user requests the root URL without specifying a file path (similar to entering `amazon.com` instead of `amazon.com/index.html`).
* **Cache Invalidation:** A reset command that clears a CDN's memory, ensuring updates to configurations or website assets are immediately visible to users globally.
