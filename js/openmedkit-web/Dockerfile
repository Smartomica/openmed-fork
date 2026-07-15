# Build context: this directory (js/openmedkit-web/)
#
# Stages
#   deps     – install dev deps
#   builder  – compile TypeScript → dist/ and typecheck
#   release  – minimal image with dist/, package.json, and src/
#
# Build (run from js/openmedkit-web/):
#   docker build -t openmedkit-web .
#
# Extract dist/ without starting a container:
#   id=$(docker create openmedkit-web)
#   docker cp $id:/app/dist ./dist && docker rm $id
#
# Note: the node test suite lives at the monorepo root (tests/web/) and is
# excluded from Docker builds; run `npm test` on the host instead.

ARG NODE_VERSION=20
ARG ALPINE_VERSION=3.21

# ── deps ──────────────────────────────────────────────────────────────────────
FROM node:${NODE_VERSION}-alpine${ALPINE_VERSION} AS deps

WORKDIR /app

COPY package.json ./

RUN npm install --ignore-scripts

# ── builder ───────────────────────────────────────────────────────────────────
FROM deps AS builder

COPY tsconfig.json ./
COPY src ./src

RUN npm run build && npm run typecheck

# ── release ───────────────────────────────────────────────────────────────────
FROM node:${NODE_VERSION}-alpine${ALPINE_VERSION} AS release

LABEL org.opencontainers.image.title="openmedkit-web" \
      org.opencontainers.image.description="On-device medical AI de-identification library for browsers and Node.js" \
      org.opencontainers.image.source="https://github.com/maziyarpanahi/openmed" \
      org.opencontainers.image.licenses="Apache-2.0"

WORKDIR /app

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

# Source included so declaration maps (.d.ts.map) resolve to original TS files
# when this image is used as a base in downstream Node.js applications.
COPY src ./src
