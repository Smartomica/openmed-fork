# syntax=docker/dockerfile:1.7
# Build context: repository root  (js/openmedkit-web/ will not reach tests/)
#
# Stages
#   deps     – install dev deps
#   builder  – tsc compile → dist/
#   test     – typecheck + node:test suite (needs tests/ from repo root)
#   release  – minimal image with dist/ and package.json only
#
# Build (default release target):
#   docker build -f js/openmedkit-web/Dockerfile -t openmedkit-web .
#
# Build and run tests:
#   docker build -f js/openmedkit-web/Dockerfile --target test -t openmedkit-web:test .
#
# Extract dist/ without starting a container:
#   id=$(docker create openmedkit-web)
#   docker cp $id:/app/js/openmedkit-web/dist ./dist && docker rm $id

ARG NODE_VERSION=20
ARG ALPINE_VERSION=3.21

# ── deps ──────────────────────────────────────────────────────────────────────
FROM node:${NODE_VERSION}-alpine${ALPINE_VERSION} AS deps

WORKDIR /app/js/openmedkit-web

COPY js/openmedkit-web/package.json ./

RUN npm install --ignore-scripts

# ── builder ───────────────────────────────────────────────────────────────────
FROM deps AS builder

COPY js/openmedkit-web/tsconfig.json ./
COPY js/openmedkit-web/src ./src

RUN npm run build

# ── test ──────────────────────────────────────────────────────────────────────
# test_npm_deidentify.spec.ts resolves paths from its own __filename via
# import.meta.url, so tests/ must sit two levels above js/openmedkit-web/
# (i.e. at /app/tests/) to match the repo layout the script expects.
FROM builder AS test

COPY tests/web /app/tests/web

RUN npm run typecheck && npm run test:node

# ── release ───────────────────────────────────────────────────────────────────
FROM node:${NODE_VERSION}-alpine${ALPINE_VERSION} AS release

LABEL org.opencontainers.image.title="openmedkit-web" \
      org.opencontainers.image.description="On-device medical AI de-identification library for browsers and Node.js" \
      org.opencontainers.image.source="https://github.com/maziyarpanahi/openmed" \
      org.opencontainers.image.licenses="Apache-2.0"

WORKDIR /app

COPY --from=builder /app/js/openmedkit-web/dist ./dist
COPY --from=builder /app/js/openmedkit-web/package.json ./

# Source included so declaration maps (.d.ts.map) resolve to original TS files
# when this image is used as a base in downstream Node.js applications.
COPY js/openmedkit-web/src ./src
