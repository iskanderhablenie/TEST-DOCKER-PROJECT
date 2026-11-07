# ── Stage 1: build ───────────────────────────────────────────────────────────
FROM node:20-alpine AS build
WORKDIR /app

# No package-lock.json in the repo → npm install (switches to npm ci once a lockfile exists)
COPY package*.json ./
RUN if [ -f package-lock.json ]; then npm ci; else npm install; fi

COPY . .

# Build to a fixed path, then copy whichever folder holds index.html
# (dist/out/ for the browser builder, dist/out/browser/ for the application builder).
RUN npx ng build --configuration production --output-path=dist/out \
 && INDEX="$(find dist/out -name index.html | head -n1)" \
 && if [ -z "$INDEX" ]; then echo "ng build produced no index.html:"; find dist; exit 1; fi \
 && mkdir -p /app/out \
 && cp -r "$(dirname "$INDEX")"/. /app/out/

# ── Stage 2: runtime ─────────────────────────────────────────────────────────
FROM nginx:1.27-alpine

COPY nginx.conf /etc/nginx/nginx.conf
RUN rm -rf /usr/share/nginx/html/*
COPY --from=build /app/out/ /usr/share/nginx/html/

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
