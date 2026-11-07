# Deploy guide (for the server AI agent)

Static website, no build step. Serve the contents of this repository's root as `https://cv.liquid1611.com/`.

## Files
- `index.html` — the portfolio page (single file, inline CSS)
- `Ali-Saad-Dawood-CV.pdf` — CV linked from the page
- `img/` — site photos
- `models/*.html` — three interactive 3D models (load `three@0.160.0` from `cdn.jsdelivr.net`; the server must not block that origin in any CSP)

## Steps
1. Make sure DNS: an `A` (and `AAAA` if used) record for `cv.liquid1611.com` points to this server.
2. Clone or update the repo into a directory outside the web root:
   ```bash
   sudo mkdir -p /var/www/cv.liquid1611.com
   git clone https://github.com/engalialbyati/portfolio.git /opt/cv-portfolio   # first time
   git -C /opt/cv-portfolio pull --ff-only   # later updates
   ```
3. Publish the files (everything except `.git` and this file):
   ```bash
   sudo rsync -a --delete --exclude '.git' --exclude 'deploy.md' --exclude 'README.md' /opt/cv-portfolio/ /var/www/cv.liquid1611.com/
   ```
4. Web server config. Use whichever server is already installed; if it is nginx:
   ```nginx
   server {
     listen 80;
     server_name cv.liquid1611.com;
     root /var/www/cv.liquid1611.com;
     index index.html;
     location / { try_files $uri $uri/ =404; }
     add_header X-Content-Type-Options nosniff;
     location ~* \.(jpg|jpeg|png|pdf)$ { expires 30d; }
   }
   ```
   Then: `sudo nginx -t && sudo systemctl reload nginx`.
   If the server uses Caddy: `cv.liquid1611.com { root * /var/www/cv.liquid1611.com; file_server }` (HTTPS is automatic).
   If another site already uses port 80/443, add this as a separate virtual host; do not change existing sites.
5. HTTPS (nginx): `sudo certbot --nginx -d cv.liquid1611.com --redirect`.
6. Verify:
   ```bash
   curl -sI https://cv.liquid1611.com/ | head -1                       # HTTP/2 200
   curl -sI https://cv.liquid1611.com/Ali-Saad-Dawood-CV.pdf | head -1 # 200, application/pdf
   curl -sI https://cv.liquid1611.com/models/slide-gate-3d.html | head -1
   ```
   Open the page and check the three 3D models load (they need internet access to jsdelivr).

## Updating later
Pull, rerun the `rsync` in step 3. No restart needed.

## Do not
- Do not delete or modify other sites on this server.
- Do not add analytics or secrets; the site is fully static.
