# Radicle web explorer, built with run-time config: it loads /config.json, mount yours there.
# The explorer has no releases of its own: VERSION is the radicle-httpd release it ships with,
# REV that release's commit in radicle-gui. Both are bumped by scripts/update.
FROM node:24-alpine3.24 AS build
ARG VERSION=0.31.0
ARG REV=11cf896b903fafc8ff860f8547394d98664ecd99
RUN apk add --no-cache git
WORKDIR /src
RUN git init -q . \
 && git remote add origin https://github.com/radicle-dev/radicle-gui.git \
 && git fetch -q --depth 1 origin "$REV" \
 && git checkout -q FETCH_HEAD
# radicle-gui became an npm workspace (shared UI code) after radicle-httpd 0.30.0; older
# release commits have a standalone explorer/ package
RUN if [ -f package.json ]; then npm ci -w explorer; else cd explorer && npm ci; fi
WORKDIR /src/explorer
RUN VITE_RUNTIME_CONFIG=true npm run build

FROM nginxinc/nginx-unprivileged:1.31-alpine3.24
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /src/explorer/build /usr/share/nginx/html
