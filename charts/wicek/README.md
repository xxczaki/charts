# wicek

![Version: 0.1.323](https://img.shields.io/badge/Version-0.1.323-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: d9b52541c09e07417256e1590fbc5286cdb31db3](https://img.shields.io/badge/AppVersion-d9b52541c09e07417256e1590fbc5286cdb31db3-informational?style=flat-square)

Minimal Claude Code agent with Discord bot interface

**Homepage:** <https://github.com/xxczaki/wicek>

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| Antoni Kepinski |  | <https://github.com/xxczaki> |

## Source Code

* <https://github.com/xxczaki/charts/tree/main/charts/wicek>

## Values

### Broker

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| broker.enabled | bool | `false` | Run the credential broker sidecar (mitmproxy). Service credentials are mounted only into it; the agent container sends its traffic through it via HTTP(S)_PROXY, trusts its CA, and uses placeholder credentials that the broker replaces for listed hosts. |
| broker.hosts | list | `[]` | Hosts the broker intercepts and how it authenticates to them, rendered to /etc/broker/config.json (see the wicek README for the format) |
| broker.image.repository | string | `"ghcr.io/xxczaki/wicek-broker"` | Broker image repository |
| broker.image.tag | string | `""` | Broker image tag (defaults to image.tag, both are built from the same commit) |
| broker.noProxy | list | `[".anthropic.com",".claude.ai",".claude.com",".discord.com",".discord.gg",".discordapp.com"]` | Hosts that bypass the broker (NO_PROXY), in addition to localhost |
| broker.port | int | `3128` | Port the broker listens on (127.0.0.1 only) |
| broker.secrets | list | `[]` | Secrets mounted read-only into the broker at /run/broker/<name>/, e.g. `[{name: grafana-cloud, secretName: grafana-cloud}]` |
| broker.sshKeys | list | `[]` | SSH private key files (under /run/broker/) to load into ssh-agent. When set, the agent container gets SSH_AUTH_SOCK instead of a key. |

### Browser

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| chromium.enabled | bool | `false` | Enable headless Chrome sidecar for browser automation |

### Bot

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| env.allowedUserIds | string | `""` | Comma-separated Discord user IDs allowed to interact |
| env.clientId | string | `""` | Discord application client ID |

### Init Containers

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| initContainers.ghCli.version | string | `"2.87.3"` | GitHub CLI version to install |

### Network Policy

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| networkPolicy.cilium.allowedCIDRs | list | `[]` | CIDRs the agent may reach over HTTPS, e.g. a LAN controller (provider "cilium" only). |
| networkPolicy.cilium.allowedFQDNs | list | `[]` | Domains the agent may reach over HTTPS. Everything else is denied (provider "cilium" only). |
| networkPolicy.enabled | bool | `true` | Restrict pod traffic (the bot is outbound-only; ingress is always denied) |
| networkPolicy.extraEgressPorts | list | `[]` | Extra TCP egress ports, e.g. 993 for the broker's IMAP mail gateway. "standard" allows them to any host, "cilium" only to allowedFQDNs and allowedCIDRs. |
| networkPolicy.provider | string | `"standard"` | Policy provider: "standard" (portable, port-based NetworkPolicy) or "cilium" (DNS-aware FQDN egress allowlist with default-deny; requires the Cilium CNI). Use "cilium" to restrict egress to specific domains. |
| networkPolicy.tailscale.enabled | bool | `true` | Allow egress to a Tailscale namespace (SSH and sidecar service ports). Applies to both providers. |
| networkPolicy.tailscale.namespace | string | `"tailscale"` | Namespace running the Tailscale operator/egress proxies |
| networkPolicy.tailscale.ports | list | `[22,8123]` | Ports allowed to Tailscale destinations |

### Storage

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| persistence.size | string | `"5Gi"` | Size of the persistent volume for Claude Code state and app data |

### Readers

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| readers | list | `[]` | Isolated readers: each answers questions about one untrusted data source from its own VM (runtimeClassName) with Bash only, behind its own credential broker pod that reaches only the listed hosts (plus the Claude API). Answers go straight to the user, never to the agent. Requires Cilium and a VM RuntimeClass. Each entry: name, description (shown to the agent), prompt (the reader's instructions and API notes), hosts (broker host entries, same format as broker.hosts), secrets (mounted into the broker at /run/broker/<name>/, same format as broker.secrets), and optional memory and model. |
| readersDefaults.brokerPort | int | `3128` | Port each reader's broker listens on |
| readersDefaults.callbackBaseUrl | string | `""` | Public base URL of the webhook server (e.g. the Funnel ingress). When set, each reader gets READER_CALLBACK_URL=<base>/hooks/callback/<name> as the redirect URL for browser logins. |
| readersDefaults.memory | string | `"768Mi"` | Reader memory (request and limit, which is also the VM size) |
| readersDefaults.model | string | `"sonnet"` | Claude model for readers |
| readersDefaults.runtimeClassName | string | `"kata-qemu"` | RuntimeClass that runs each reader in its own VM |
| readersDefaults.stateSize | string | `"128Mi"` | Persistent state (IDs, expiry dates) for each reader |

### Authentication

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| secrets.discordToken | object | `{"key":"","name":""}` | Points to a Secret containing the Discord bot token |
| secrets.oauthToken | object | `{"key":"","name":""}` | Points to a Secret containing the Claude Code OAuth token Generated via `claude setup-token` |

### Webhooks

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| secrets.githubWebhookSecret | object | `{"key":"","name":""}` | Points to a Secret containing the GitHub webhook secret (X-Hub-Signature-256) When unset, GITHUB_WEBHOOK_SECRET is not injected and /hooks/github returns 404. |
| secrets.grafanaWebhookToken | object | `{"key":"","name":""}` | Points to a Secret containing the Grafana webhook bearer token When unset, GRAFANA_WEBHOOK_TOKEN is not injected and /hooks/grafana returns 404. |
| webhooks.ingress.enabled | bool | `false` | Create a Tailscale Ingress exposing only /hooks (requires webhooks.service.enabled) |
| webhooks.ingress.funnel | bool | `true` | Expose the ingress publicly via Tailscale Funnel (tailscale.com/funnel annotation) |
| webhooks.ingress.hostname | string | `"wicek-hooks"` | Tailnet hostname of the ingress (becomes <hostname>.<tailnet>.ts.net) |
| webhooks.port | int | `8080` | Port of the webhook HTTP server (/hooks/github, /hooks/grafana, /healthz) |
| webhooks.service.enabled | bool | `false` | Expose the webhook port as a ClusterIP Service and allow ingress to it from the Tailscale namespace |

### SSH

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| ssh.knownHosts | string | `""` | SSH known_hosts entries (`host keytype key`, one per line), mounted at /etc/ssh/ssh_known_hosts so host keys are trusted across redeploys. Get them with `ssh-keyscan <host>`. When empty, no file is mounted. |

### Other Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| broker.resources.limits.cpu | string | `"300m"` |  |
| broker.resources.limits.memory | string | `"256Mi"` |  |
| broker.resources.requests.cpu | string | `"10m"` |  |
| broker.resources.requests.memory | string | `"64Mi"` |  |
| chromium.image | string | `"chromedp/headless-shell:stable"` |  |
| chromium.resources.limits.cpu | string | `"500m"` |  |
| chromium.resources.limits.memory | string | `"512Mi"` |  |
| chromium.resources.requests.cpu | string | `"100m"` |  |
| chromium.resources.requests.memory | string | `"256Mi"` |  |
| image.pullPolicy | string | `"Always"` |  |
| image.repository | string | `"ghcr.io/xxczaki/wicek"` |  |
| image.tag | string | `"d9b52541c09e07417256e1590fbc5286cdb31db3"` |  |
| persistence.accessModes[0] | string | `"ReadWriteOnce"` |  |
| persistence.storageClass | string | `""` |  |
| resources.limits.cpu | string | `"500m"` |  |
| resources.limits.memory | string | `"1Gi"` |  |
| resources.requests.cpu | string | `"50m"` |  |
| resources.requests.memory | string | `"128Mi"` |  |
| securityContext.allowPrivilegeEscalation | bool | `false` |  |
| securityContext.capabilities.drop[0] | string | `"ALL"` |  |
| securityContext.runAsNonRoot | bool | `true` |  |
| securityContext.seccompProfile.type | string | `"RuntimeDefault"` |  |
| serviceAccount.annotations | object | `{}` |  |
| serviceAccount.create | bool | `true` |  |
| serviceAccount.name | string | `""` |  |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)
