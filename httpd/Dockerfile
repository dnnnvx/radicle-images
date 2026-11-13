# radicle-httpd from the upstream static (musl) release.
# VERSION and the checksums are bumped by scripts/update after verifying the release signature.
FROM alpine:3.24 AS fetch
ARG TARGETARCH
ARG VERSION=0.31.0
ARG SHA256_AMD64=b837befda0a0ccdeab2e6fdb1362ee8aa785fd104643c8f3f102e6ce31f67cf2
ARG SHA256_ARM64=32e897a6855affe60b2f8443a5e02d7b44f8f5d7fad5dfed93ff316de7c7a939
RUN apk add --no-cache curl xz
WORKDIR /dl
RUN set -eu; \
    case "$TARGETARCH" in \
      amd64) t=x86_64;  sum=$SHA256_AMD64 ;; \
      arm64) t=aarch64; sum=$SHA256_ARM64 ;; \
      *) echo "unsupported arch: $TARGETARCH" >&2; exit 1 ;; \
    esac; \
    h=radicle-httpd-$VERSION-$t-unknown-linux-musl; \
    curl -fsSLO https://files.radicle.dev/releases/radicle-httpd/$VERSION/$h.tar.xz; \
    echo "$sum  $h.tar.xz" | sha256sum -c -; \
    mkdir /out; \
    tar -xJf $h.tar.xz --strip-components=2 -C /out $h/bin

FROM alpine:3.24
# git for repository reads; zstd for .tar.zst archive downloads
RUN apk add --no-cache git zstd ca-certificates \
 && adduser -D -u 1000 -h /var/lib/radicle radicle
COPY --from=fetch /out/ /usr/local/bin/
ENV RAD_HOME=/var/lib/radicle
USER 1000
EXPOSE 8080
ENTRYPOINT ["radicle-httpd"]
CMD ["--listen", "0.0.0.0:8080"]
