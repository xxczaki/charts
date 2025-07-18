# discord-bot

![Version: 0.14.41](https://img.shields.io/badge/Version-0.14.41-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: eafb6a8ddc5c563084d78fedf578bf57f2c6b538](https://img.shields.io/badge/AppVersion-eafb6a8ddc5c563084d78fedf578bf57f2c6b538-informational?style=flat-square)

🎵 Music bot for my private Discord server, powered by discord-player

**Homepage:** <https://github.com/xxczaki/discord-bot>

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| Antoni Kępiński |  | <https://github.com/xxczaki> |

## Source Code

* <https://github.com/xxczaki/charts/tree/main/charts/discord-bot>

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://charts.bitnami.com/bitnami | redis | ~21.2.6 |

## Values

### Bot

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| env.botDebugChannelId | string | `""` | ID of the channel used for sending debug messages |
| env.clientId | string | `""` | Client ID used for Discord API authorization |
| env.ownerUserId | string | `""` | ID of the user that should be allowed to perform sensitive actions, like clearing the query cache |
| env.playlistsChannelId | string | `""` | ID of the channel used for storing the user playlists |
| env.spotifyClientId | string | `""` | Optional client ID for the Spotify API Used by https://github.com/iTsMaaT/discord-player-spotify |
| persistentVolumeClaim | object | `{"accessModes":["ReadWriteOnce"],"size":"5Gi","storageClass":""}` | Configuration of the PVC used for storing the cached songs |
| secrets.sentryDsn | object | `{"key":"","name":""}` | Points to a Secret containing the Sentry DSN Learn more: https://docs.sentry.io/concepts/key-terms/dsn-explainer/ |
| secrets.spotifyClientSecret | object | `{"key":"","name":""}` | Points to a Secret containing the Spotify API client secret Used by https://github.com/iTsMaaT/discord-player-spotify |
| secrets.token | object | `{"key":"","name":""}` | Points to a Secret containing the Discord bot token |

### RBAC

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| rbac | object | `{"create":true}` | RBAC configuration to access the Kubernetes API |

### Redis

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| redis | object | `{"architecture":"standalone","auth":{"enabled":false},"commonConfiguration":"# Enable AOF https://redis.io/topics/persistence#append-only-file\nappendonly yes\n# Disable RDB persistence, AOF persistence already enabled.\nsave \"\""}` | General configuration of the Redis instance used for query caching and storing usage statistics |

### Other Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| image | object | `{"pullPolicy":"Always","repository":"xxczaki/discord-bot","tag":"eafb6a8ddc5c563084d78fedf578bf57f2c6b538"}` | General configuration of the Redis instance used for query caching and storing usage statistics |
| livenessProbe | object | `{"initialDelaySeconds":10,"periodSeconds":10,"tcpSocket":{"port":8000}}` | Liveness probe used for the bot Pod |
| readinessProbe | object | `{"initialDelaySeconds":10,"periodSeconds":10,"tcpSocket":{"port":8000}}` | Readiness probe used for the bot Pod |
| resources | object | `{"limits":{"cpu":"400m","memory":"512Mi"},"requests":{"cpu":"100m","memory":"128Mi"}}` | Resource limits used for the bot Pod |
| securityContext | object | `{"allowPrivilegeEscalation":false,"capabilities":{"drop":["ALL"]},"runAsNonRoot":true,"seccompProfile":{"type":"RuntimeDefault"}}` | Security context used for the bot Pod |
| serviceAccount.annotations | object | `{}` |  |
| serviceAccount.create | bool | `true` |  |
| serviceAccount.name | string | `""` |  |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)
