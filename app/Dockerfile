# Stage 1: Build

FROM node:20-bookworm AS builder

ARG THREAT_COMPOSER_REF=main

WORKDIR /build

# Shallow clone: we only need the latest snapshot, not the full git history.
RUN git clone --depth 1 --branch ${THREAT_COMPOSER_REF} \
    https://github.com/awslabs/threat-composer.git .

# pdk and yarn are required to install/build this monorepo (per upstream docs)
RUN npm install -g @aws/pdk yarn

RUN pdk install --frozen-lockfile


RUN pdk build


# Stage 2: Runtime

FROM nginx:1.27-alpine

RUN addgroup -g 1001 appgroup && \
    adduser -D -u 1001 -G appgroup appuser

# Replace the default server block with our own (listens on 8080, see below)
RUN rm -f /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Pull only the compiled static site from the build stage
COPY --from=builder /build/packages/threat-composer-app/build /usr/share/nginx/html

# nginx normally needs root to bind to its default paths and port 80.
# Since we run as non-root, hand ownership of the directories nginx writes
# to (cache, pid file) to our unprivileged user, and use a port above 1024.
RUN chown -R appuser:appgroup /usr/share/nginx/html /var/cache/nginx /var/run && \
    touch /var/run/nginx.pid && \
    chown appuser:appgroup /var/run/nginx.pid

USER appuser

EXPOSE 8080

ENTRYPOINT ["nginx", "-g", "daemon off;"]