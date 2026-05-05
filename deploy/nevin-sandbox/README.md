# Nevin sandbox deployment

Files to copy onto the Yopass instance after Phase 1 of the deployment plan
is applied (see `/Users/bjornahlander/.claude/plans/twinkly-puzzling-cosmos.md`,
Phase 4).

```
~/yopass/
├── docker-compose.yml   # this file from the repo
└── .env                 # copy of .env.example with REVERSE_PROXY_PRIVATE_IP filled in
```

## On the instance

```sh
mkdir -p ~/yopass && cd ~/yopass
# Copy docker-compose.yml and .env from this repo's deploy/nevin-sandbox/
# (e.g. via scp from your laptop, or `git clone` then symlink)
echo "$SCW_SECRET_KEY" | docker login rg.nl-ams.scw.cloud -u nologin --password-stdin
docker compose pull
docker compose up -d
```

## Branding flags explained

The Nevin fork has the License.Valid gate removed in `pkg/server/server.go`,
so the upstream branding flags work without a license. They're set here in
`command:` rather than baked into source so future tweaks don't need a code
change — bump the YAML, `docker compose up -d`, done.

| Flag | Source / why |
|---|---|
| `--memcached=memcached:11211` | Yopass storage backend, in-stack |
| `--port=1337 --address=0.0.0.0` | Listens on the private network interface |
| `--trusted-proxies=${REVERSE_PROXY_PRIVATE_IP}/32` | Trust X-Forwarded-For only from the Caddy box |
| `--force-onetime-secrets` | Compliance: every secret self-destructs after one read |
| `--app-name=Nevin ❤️ yopass` | Header/title (sent via /config to the React app); credits the upstream open-source project |
| `--logo-url=/nevin-logo.svg` | Logo path under the embedded `/public` directory |
| `--theme-light=nord --theme-dark=nord` | DaisyUI built-in base |
| `--theme-custom-light=...` | OKLCH overrides for primary/accent/base-content |

The OKLCH values were computed from Nevin hex (`#4d3671`, `#fcb245`, `#434343`)
via the standard sRGB → OKLab → OKLCH transform. Don't eyeball when bumping —
use `oklch.com` or a real conversion tool.

## Logo

The image expects `/nevin-logo.svg` to exist inside the container's `/public`
directory. The Dockerfile copies `website/public/` into `/public`, so drop
`nevin-logo.svg` into `website/public/` in the source repo before building.
Until then the navbar will show a broken-image icon.

## Caddy backend registration

Caddy reaches Yopass via the VPC private network, configured in
`nevin-platform-infra/environments/sandbox/main.tf` under
`module.reverse_proxy.services["yopass.sandbox.nevin.se"]`. No DNS record
needed — the `*.sandbox.nevin.se` wildcard already covers it.
