# syntax=docker/dockerfile:1
# Production image for the poshtomat service (Node.js, no runtime dependencies).
# Build: docker build -t ghcr.io/<owner>/poshtomat:<tag> -f deploy/Dockerfile app

FROM node:22-alpine AS base

WORKDIR /app
ENV NODE_ENV=production \
    PORT=8080 \
    LOG_LEVEL=info

# --- dependencies -----------------------------------------------------------
FROM base AS deps
COPY package.json ./
RUN npm ci --omit=dev --ignore-scripts && npm cache clean --force

# --- tests run inside the image (fail the build if red) ---------------------
FROM base AS test
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN node --test

# --- release: lean image, non-root user -------------------------------------
FROM base AS release
RUN addgroup --system --gid 1000 poshtomat && \
    adduser --system --uid 1000 --ingroup poshtomat poshtomat

COPY --from=deps /app/node_modules ./node_modules
COPY package.json ./
COPY src ./src
COPY bin ./bin

USER poshtomat

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD wget -qO- http://localhost:8080/health || exit 1

CMD ["node", "src/server.js"]
