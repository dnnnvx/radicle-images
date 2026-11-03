# rad, radicle-node and git-remote-rad from the upstream static (musl) release.
# VERSION and the checksums are bumped by scripts/update after verifying the release signature.
FROM alpine:3.24 AS fetch
ARG TARGETARCH
ARG VERSION=1.10.3
ARG SHA256_AMD64=3d78e3ffc17f9e6eca7fcd3acc439f2252060532eb457cf17f2747fed4cc6751
ARG SHA256_ARM64=822c1ea1da7dd3a2a7f98da6d93b95359b5b9b0e302b8535bea377a2441e596f
RUN apk add --no-cache curl xz
WORKDIR /dl
RUN set -eu; \
    case "$TARGETARCH" in \
      amd64) t=x86_64;  sum=$SHA256_AMD64 ;; \
      arm64) t=aarch64; sum=$SHA256_ARM64 ;; \
      *) echo "unsupported arch: $TARGETARCH" >&2; exit 1 ;; \
    esac; \
    r=radicle-$VERSION-$t-unknown-linux-musl; \
    curl -fsSLO https://files.radicle.dev/releases/$VERSION/$r.tar.xz; \
    echo "$sum  $r.tar.xz" | sha256sum -c -; \
    mkdir /out; \
    tar -xJf $r.tar.xz --strip-components=2 -C /out $r/bin

FROM alpine:3.24
# radicle-node shells out to git for fetches
RUN apk add --no-cache git ca-certificates \
 && adduser -D -u 1000 -h /var/lib/radicle radicle
COPY --from=fetch /out/ /usr/local/bin/
ENV RAD_HOME=/var/lib/radicle
USER 1000
EXPOSE 8776
ENTRYPOINT ["radicle-node"]
