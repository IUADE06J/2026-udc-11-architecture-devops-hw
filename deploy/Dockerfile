# syntax=docker/dockerfile:1
# Production image for the poshtomat service (Node.js, no runtime dependencies).
# Build: docker build -t ghcr.io/<owner>/poshtomat:<tag> -f deploy/Dockerfile app

FROM node:22-alpine AS base

WORKDIR /app
ENV NODE_ENV=production \
    PORT=8080 \
    LOG_LEVEL=info

# --- tests run inside the image (fail the build if red) ---------------------
FROM base AS test
COPY . .
RUN node --test

# --- release: lean image, non-root user -------------------------------------
FROM base AS release

COPY package.json ./
COPY src ./src
COPY bin ./bin

USER node

EXPOSE 8080

CMD ["node", "src/server.js"]
