# 🚀 Deployment Guide

Complete step-by-step instructions for deploying the India Digital Twin Dashboard on various platforms.

---

## 📋 Table of Contents

1. [GitHub Pages (Recommended)](#github-pages)
2. [Vercel](#vercel)
3. [Netlify](#netlify)
4. [AWS S3 + CloudFront](#aws)
5. [Docker & Self-Hosting](#docker)
6. [Local Development](#local)

---

## GitHub Pages

### Setup (5 minutes)

#### Step 1: Create GitHub Repository

```bash
# Create a new directory
mkdir india-digital-twin
cd india-digital-twin

# Initialize git
git init
git config user.name "Your Name"
git config user.email "your.email@example.com"
```

#### Step 2: Add Files

Copy these files to your directory:
- `index.html` - Main application
- `districts.json` - Districts data
- `README.md` - Documentation
- `.gitignore` (create):

```
.env
.DS_Store
node_modules/
*.log
```

#### Step 3: Commit & Push

```bash
# Add files
git add .
git commit -m "Initial commit: India Digital Twin Dashboard"

# Add remote (replace USERNAME with your GitHub username)
git remote add origin https://github.com/USERNAME/india-digital-twin.git

# Push to GitHub
git branch -M main
git push -u origin main
```

#### Step 4: Enable GitHub Pages

1. Go to **Settings** → **Pages**
2. Under "Source", select:
   - Branch: `main`
   - Folder: `/ (root)`
3. Click **Save**
4. GitHub will show: "Your site is published at `https://USERNAME.github.io/india-digital-twin`"

#### Step 5: Add Custom Domain (Optional)

In **Settings** → **Pages**, under "Custom domain":
1. Enter: `yourdomain.com` or `digital-twin.yourdomain.com`
2. Add DNS records:

```
Type    | Host          | Value
--------|---------------|--------------------------------------------
ALIAS   | @             | USERNAME.github.io
CNAME   | www           | USERNAME.github.io
```

3. Click **Save**

### Updating Content

```bash
# Make changes to files
# Edit index.html, districts.json, etc.

# Commit changes
git add .
git commit -m "Update: Add new features"
git push origin main

# Changes live within 1-2 minutes
```

### CI/CD with GitHub Actions (Optional)

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Validate HTML
        run: |
          npm install -g html-validate
          html-validate index.html
      - name: Validate JSON
        run: |
          python3 -m json.tool districts.json > /dev/null
      
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./
```

---

## Vercel

### Setup (3 minutes)

#### Step 1: Connect GitHub

1. Go to [vercel.com](https://vercel.com)
2. Click **Import Project**
3. Select **Import Git Repository**
4. Connect your GitHub account
5. Find and select `india-digital-twin`
6. Click **Import**

#### Step 2: Configure Project

Leave defaults as-is:
- **Framework**: None (Static Site)
- **Root Directory**: `./`
- **Build Command**: (empty)
- **Output Directory**: `./`

#### Step 3: Deploy

Click **Deploy** - Your site goes live in ~30 seconds

**Live URL**: `https://india-digital-twin-USERNAME.vercel.app`

#### Step 4: Custom Domain (Optional)

1. Go to **Project Settings** → **Domains**
2. Add your domain
3. Update DNS:

```
CNAME  digital-twin  india-digital-twin-USERNAME.vercel.app
```

#### Step 5: Environment Variables

For AI features with API key:

1. **Settings** → **Environment Variables**
2. Add:
   - Key: `NEXT_PUBLIC_ANTHROPIC_API_KEY`
   - Value: `sk-...` (your API key)

**Security Note**: Don't expose API keys in client-side code. Use a backend proxy instead.

---

## Netlify

### Setup (3 minutes)

#### Step 1: Connect Repository

1. Go to [netlify.com](https://netlify.com)
2. Click **Add new site** → **Import an existing project**
3. Select **GitHub**
4. Connect your GitHub account
5. Choose the `india-digital-twin` repository

#### Step 2: Configure Build

Set these values:
- **Base directory**: `/` (leave empty)
- **Build command**: (leave empty)
- **Publish directory**: `/`

#### Step 3: Deploy

Click **Deploy site** - Takes 30-60 seconds

**Live URL**: `https://india-digital-twin-RANDOM.netlify.app`

#### Step 4: Custom Domain

1. **Site settings** → **Domain settings**
2. Click **Change site name** or add custom domain
3. Configure DNS (CNAME)

#### Step 5: Environment Variables

For API keys:

1. **Site settings** → **Build & deploy** → **Environment**
2. Add:
   ```
   REACT_APP_ANTHROPIC_API_KEY = sk-...
   ```

---

## AWS

### S3 + CloudFront (Static Hosting)

#### Step 1: Create S3 Bucket

```bash
aws s3 mb s3://india-digital-twin --region us-east-1
```

Or via AWS Console:
1. S3 → Create bucket
2. Name: `india-digital-twin`
3. Uncheck "Block all public access"
4. Create

#### Step 2: Enable Static Website Hosting

1. **Bucket** → **Properties** → **Static website hosting**
2. Enable hosting
3. Index document: `index.html`
4. Error document: `index.html`

#### Step 3: Upload Files

```bash
aws s3 cp index.html s3://india-digital-twin/
aws s3 cp districts.json s3://india-digital-twin/
aws s3 cp README.md s3://india-digital-twin/
```

Or via CLI:
```bash
aws s3 sync . s3://india-digital-twin --exclude ".git*"
```

#### Step 4: Set Bucket Policy

Create `bucket-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::india-digital-twin/*"
    }
  ]
}
```

Apply:
```bash
aws s3api put-bucket-policy \
  --bucket india-digital-twin \
  --policy file://bucket-policy.json
```

#### Step 5: Create CloudFront Distribution

1. CloudFront → Create distribution
2. Origin: `india-digital-twin.s3.amazonaws.com`
3. Origin access: Restrict
4. Viewer protocol: Redirect HTTP to HTTPS
5. Default root: `index.html`
6. Error pages: `/index.html` for 403 and 404
7. Create

**URL**: `https://d123abc.cloudfront.net`

#### Step 6: Custom Domain (Route53)

```bash
# Create Route53 record
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123ABC \
  --change-batch file://dns-change.json
```

---

## Docker

### Self-Hosting with Docker

#### Step 1: Create Dockerfile

```dockerfile
FROM nginx:alpine

# Copy files
COPY index.html /usr/share/nginx/html/
COPY districts.json /usr/share/nginx/html/
COPY README.md /usr/share/nginx/html/

# Configure nginx
RUN echo 'server {
  listen 80;
  location / {
    root /usr/share/nginx/html;
    try_files $uri $uri/ =404;
  }
}' > /etc/nginx/conf.d/default.conf

EXPOSE 80
```

#### Step 2: Build Image

```bash
docker build -t india-digital-twin:latest .
```

#### Step 3: Run Container

```bash
docker run -d \
  --name digital-twin \
  -p 80:80 \
  india-digital-twin:latest
```

Access at: `http://localhost`

#### Step 4: Push to Docker Hub

```bash
# Login
docker login

# Tag
docker tag india-digital-twin:latest USERNAME/india-digital-twin:latest

# Push
docker push USERNAME/india-digital-twin:latest

# Pull on any machine
docker pull USERNAME/india-digital-twin:latest
```

#### Step 5: Deploy to Production Server

```bash
# SSH into server
ssh user@server.com

# Pull image
docker pull USERNAME/india-digital-twin:latest

# Run with persistence
docker run -d \
  --name digital-twin \
  --restart unless-stopped \
  -p 80:80 \
  -p 443:443 \
  -v /etc/letsencrypt:/etc/letsencrypt \
  USERNAME/india-digital-twin:latest
```

---

## Local Development

### Quick Setup

```bash
# Clone or navigate to directory
cd india-digital-twin

# Python 3
python -m http.server 8000

# Or Node.js with http-server
npm install -g http-server
http-server

# Or Ruby
ruby -run -ehttpd . -p8000

# Visit: http://localhost:8000
```

### With Live Reload (Development)

```bash
# Install live-server
npm install -g live-server

# Run
live-server

# Automatically refreshes on file changes
```

### Testing with Different Browsers

```bash
# Test mobile viewport
# Chrome DevTools: Ctrl+Shift+M

# Test dark mode
# Chrome: Ctrl+Shift+P → "Emulate CSS media feature prefers-color-scheme"

# Test performance
# Chrome DevTools → Lighthouse → Generate report
```

---

## Environment-Specific Configurations

### Development (Local)

```html
<!-- In index.html -->
<script>
  const ENV = 'development';
  const API_BASE = 'http://localhost:3000';
</script>
```

### Staging

```html
<script>
  const ENV = 'staging';
  const API_BASE = 'https://api-staging.yourdomain.com';
</script>
```

### Production

```html
<script>
  const ENV = 'production';
  const API_BASE = 'https://api.yourdomain.com';
</script>
```

---

## Monitoring & Analytics

### Add Google Analytics

```html
<!-- In index.html, before </head> -->
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'GA_MEASUREMENT_ID');
</script>
```

### Error Tracking (Sentry)

```html
<script src="https://browser.sentry-cdn.com/7.x.x/bundle.min.js"></script>
<script>
  Sentry.init({
    dsn: 'https://key@sentry.io/project',
    environment: 'production'
  });
</script>
```

---

## SSL/TLS Certificates

### Let's Encrypt (Free)

```bash
# Using Certbot
sudo certbot certonly --standalone -d yourdomain.com

# Renew (automatic with most platforms)
sudo certbot renew
```

### CloudFlare (Free)

1. Add domain to CloudFlare
2. Update nameservers
3. CloudFlare automatically manages SSL
4. Enable: SSL/TLS → Full Strict

---

## Performance Optimization

### Enable Compression

**Nginx**:
```nginx
gzip on;
gzip_types text/plain text/css text/javascript application/json;
gzip_min_length 1000;
```

**Apache**:
```apache
<IfModule mod_deflate.c>
  AddOutputFilterByType DEFLATE text/html text/plain text/xml text/javascript
</IfModule>
```

### Caching Headers

```
# Cache static assets for 1 year
Cache-Control: public, max-age=31536000

# Cache HTML for 1 hour
Cache-Control: public, max-age=3600
```

### CDN Setup

- **CloudFlare**: Free CDN with caching
- **Cloudfront**: AWS CDN with EdgeLocations
- **Fastly**: Premium CDN for high performance

---

## Troubleshooting Deployments

### Issue: "404 Not Found"

**Cause**: Missing `index.html` or wrong file structure

**Fix**:
```bash
# Verify files
ls -la
# Should show: index.html, districts.json, README.md

# Check deployment logs
# GitHub: Actions tab
# Vercel: Deployments tab
```

### Issue: "CORS Error"

**Cause**: API calls blocked by browser

**Fix**: Use a backend proxy or enable CORS headers

### Issue: "Blank Page"

**Cause**: JavaScript error or missing dependencies

**Fix**:
```bash
# Check console
F12 → Console → Look for red errors

# Validate HTML
html-validate index.html

# Validate JSON
python -m json.tool districts.json
```

### Issue: "Slow Load Time"

**Cause**: Large files or slow CDN

**Fix**:
```bash
# Minimize files
gzip -9 index.html > index.html.gz

# Check file sizes
du -h index.html districts.json

# Use CDN for external assets
# Already using: Chart.js, Font Awesome from CDN
```

---

## Backup & Disaster Recovery

### Backup Strategy

```bash
# Backup to GitHub (automatic with git)
git push -u origin main

# Manual backup
tar -czf backup-$(date +%Y%m%d).tar.gz .

# S3 backup
aws s3 cp backup-20240101.tar.gz s3://backups/india-digital-twin/
```

### Restore from Backup

```bash
# From GitHub
git clone https://github.com/USERNAME/india-digital-twin.git

# From tar
tar -xzf backup-20240101.tar.gz

# From S3
aws s3 cp s3://backups/india-digital-twin/backup-20240101.tar.gz .
tar -xzf backup-20240101.tar.gz
```

---

## Security Checklist

- [ ] API keys never committed to repo
- [ ] `.gitignore` includes `.env` and sensitive files
- [ ] HTTPS enabled for custom domains
- [ ] Security headers configured (HSTS, CSP)
- [ ] Rate limiting on backend proxy
- [ ] User inputs validated before API calls
- [ ] Regular backups maintained
- [ ] Dependencies kept up to date
- [ ] No console logs with sensitive data
- [ ] CORS properly configured

---

## Performance Benchmarks

Expected performance:
- **Page Load**: < 2s (with good internet)
- **API Calls**: < 1s (with configured API)
- **Chart Render**: < 500ms
- **Mobile**: < 3s (with 3G connection)

Test with:
```bash
# Lighthouse
npm install -g lighthouse
lighthouse https://yourdomain.com --view

# WebPageTest
# Visit webpagetest.org
```

---

## Maintenance & Updates

### Update Deployment

```bash
# Make code changes
# Test locally
python -m http.server 8000

# Commit and push
git add .
git commit -m "Update: Feature X"
git push origin main

# Verify deployment
# GitHub Pages: ~1 minute
# Vercel/Netlify: ~30 seconds
# Docker: Manual redeploy
```

### Keep Dependencies Updated

```bash
# Check external resources
curl -I https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.9.1/chart.min.js

# Update CDN links if new versions released
```

---

## Multi-Environment Deployment

### Development → Staging → Production

```yaml
# .github/workflows/deploy.yml
name: Multi-environment Deploy

on:
  push:
    branches:
      - develop  # Deploy to staging
      - main     # Deploy to production

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to Vercel (Staging)
        run: vercel deploy --prod --token=${{ secrets.VERCEL_TOKEN_STAGING }}

  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

---

## Questions?

Refer to:
- **GitHub Pages Help**: https://docs.github.com/en/pages
- **Vercel Docs**: https://vercel.com/docs
- **Netlify Docs**: https://docs.netlify.com
- **AWS Docs**: https://docs.aws.amazon.com

**Last Updated**: 2024 | v1.0
