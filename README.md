# radicle-images

Container images for running a [Radicle](https://radicle.dev) seed node, built from the
upstream releases and published to GHCR for `linux/amd64` and `linux/arm64`.

| Image | Contents | Tag |
|---|---|---|
| `ghcr.io/dnnnvx/radicle-node` | `radicle-node`, `rad`, `git-remote-rad` and git, on Alpine | radicle version, e.g. `1.10.3` |
| `ghcr.io/dnnnvx/radicle-httpd` | `radicle-httpd` (read-only HTTP API) and git, on Alpine | radicle-httpd version, e.g. `0.30.0` |
| `ghcr.io/dnnnvx/radicle-explorer` | the web explorer from [radicle-gui](https://github.com/radicle-dev/radicle-gui), on unprivileged nginx (port 8080) | the radicle-httpd release it ships with, e.g. `0.30.0` |

Tags are the upstream `x.y.z` and are never rebuilt or moved once published. There is no `latest`.

## Examples

### Docker Compose

[`examples/compose.yaml`](examples/compose.yaml) runs a seed node, its HTTP API and the explorer
(with [`examples/explorer.json`](examples/explorer.json) pointing at `localhost:8080`):

```sh
cd examples
docker compose up -d
docker compose logs init        # your node's DID
```

| | |
|---|---|
| Explorer | http://localhost:8081 |
| HTTP API | http://localhost:8080/api/v1 |
| P2P | `8776` (forward it and set `node.externalAddresses` for a public seed) |

A one-shot `init` service creates the node key in the `radicle` volume on first start;
the node and httpd share that volume. Run `rad` commands inside the node container, e.g.
`docker compose exec node rad seed <rid>` to start seeding a repository.

### Kubernetes

[`examples/kubernetes.yaml`](examples/kubernetes.yaml) is the same setup as plain manifests: one pod
with an init container (key), `radicle-node` and `radicle-httpd` on a shared PVC, the explorer with its
config in a ConfigMap, and a `LoadBalancer` Service for P2P. Everything runs non-root with a read-only
root filesystem and the `restricted` Pod Security profile's settings.

```sh
kubectl apply -f examples/kubernetes.yaml
kubectl -n radicle logs deploy/radicle-node -c init   # your node's DID
```

Then expose `radicle-httpd` and `radicle-explorer` (port 8080) through your Ingress or Gateway, and set the
httpd's public hostname in the explorer ConfigMap: visitors' browsers call the httpd directly.

## How releases get here

A daily workflow (`update.yaml`) runs `scripts/update`, which reads the `latest` pointers on
`files.radicle.dev`. For a new release it:

1. downloads each tarball and verifies its SSH signature against [`allowed_signers`](allowed_signers)
   (the maintainers' keys from heartwood's `.gitsigners`, the same check as `radicle.dev/install`),
2. pins the version and sha256s in the Dockerfile (the build checks them again),
3. commits the bump to `main` and dispatches `publish.yaml`, which builds every version that isn't on GHCR yet.

A release signed by a key outside `allowed_signers` fails the workflow rather than being published:
review the new key and add it by hand. The explorer has no releases of its own, so it is built at the
radicle-gui commit of each radicle-httpd release.

To run it locally: `scripts/update` (needs bash ≥ 4.4, curl, jq, ssh-keygen).

## radicle-node

Runs as uid 1000 with `RAD_HOME=/var/lib/radicle`; the entrypoint is `radicle-node`.

```sh
# one-off: create the node key (no passphrase)
docker run --rm -v radicle:/var/lib/radicle --entrypoint sh ghcr.io/dnnnvx/radicle-node:1.10.3 \
  -c 'echo | rad auth --alias my-seed --stdin'
docker run -d --name radicle-node -v radicle:/var/lib/radicle -p 8776:8776 ghcr.io/dnnnvx/radicle-node:1.10.3 --force
```

Configure it in `$RAD_HOME/config.json` (see the [seeder guide](https://radicle.dev/guides/seeder)).

## radicle-httpd

Same uid and `RAD_HOME`, so it can share the node's volume. It listens on `0.0.0.0:8080` by default;
extra arguments replace that (e.g. `--listen 0.0.0.0:8080 --alias <name> <rid>`).

```sh
docker run -d -v radicle:/var/lib/radicle -p 8080:8080 ghcr.io/dnnnvx/radicle-httpd:0.30.0
```

## radicle-explorer

Built with run-time configuration: mount a `config.json` shaped like upstream's
[`config/default.json`](https://github.com/radicle-dev/radicle-gui/blob/master/explorer/config/default.json)
at `/usr/share/nginx/html/config.json`, with your seed first in `preferredSeeds`.

```sh
docker run -d -p 8080:8080 -v ./config.json:/usr/share/nginx/html/config.json:ro ghcr.io/dnnnvx/radicle-explorer:0.30.0
```
