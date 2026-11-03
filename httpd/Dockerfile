# radicle-httpd from the upstream static (musl) release.
# VERSION and the checksums are bumped by scripts/update after verifying the release signature.
FROM alpine:3.24 AS fetch
ARG TARGETARCH
ARG VERSION=0.30.0
ARG SHA256_AMD64=dd16f6f966e62fafee5a6547d1e31c8857c607c5d955380872c9e7a06500285f
ARG SHA256_ARM64=ab6ab9f073f9ebc58b3132f9686b65d882250dc434fb4fea32806481f665dd80
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
