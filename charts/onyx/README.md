# onyx

![Version: 0.0.3](https://img.shields.io/badge/Version-0.0.3-informational?style=flat-square)

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://charts.bitnami.com/bitnami | redis | 20.1.7 |
| https://cloudnative-pg.github.io/charts | cnpg(cluster) | 0.0.11 |
| https://zadarastorage.github.io/helm-charts | vespa(vespa-ai) | 0.0.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| api.replicaCount | int | `1` | Set number of api replicas |
| api.resources.limits | object | `{}` |  |
| api.resources.requests.cpu | string | `"0.1"` |  |
| api.resources.requests.memory | string | `"1Gi"` |  |
| cnpg.backups.enabled | bool | `false` |  |
| cnpg.cluster.affinity.topologyKey | string | `"kubernetes.io/hostname"` |  |
| cnpg.cluster.instances | int | `3` |  |
| cnpg.cluster.monitoring.additionalLabels.release | string | `"victoria-metrics-k8s-stack"` |  |
| cnpg.cluster.monitoring.enabled | bool | `true` |  |
| cnpg.cluster.postgresql.max_connections | string | `"500"` | Max psql connections. Default was 100 |
| cnpg.cluster.postgresql.max_locks_per_transaction | string | `"128"` | Max locks per transaction. Default was 64 |
| cnpg.enabled | bool | `true` |  |
| cnpg.mode | string | `"standalone"` |  |
| cnpg.pooler.enabled | bool | `true` |  |
| cnpg.pooler.instances | int | `2` |  |
| cnpg.pooler.monitoring.additionalLabels.release | string | `"victoria-metrics-k8s-stack"` |  |
| cnpg.pooler.monitoring.enabled | bool | `true` |  |
| cnpg.pooler.parameters.default_pool_size | string | `"50"` | Pool size. Default was 25 |
| cnpg.pooler.parameters.max_client_conn | string | `"1000"` | Max client connections, default was 1000 |
| cnpg.pooler.parameters.reserve_pool_size | string | `"25"` | Reservice pool size, default was 0/disabled |
| cnpg.type | string | `"postgresql"` |  |
| configMap.api | object | `{}` | ConfigMap for env vars specific to "api" pod(s) |
| configMap.auth | object | `{"AUTH_TYPE":"disabled","DANSWER_BOT_SLACK_APP_TOKEN":"","DANSWER_BOT_SLACK_BOT_TOKEN":"","GOOGLE_OAUTH_CLIENT_ID":"","GOOGLE_OAUTH_CLIENT_SECRET":""}` | ConfigMap for auth env vars |
| configMap.global | object | `{}` | ConfigMap for env vars for all onyx pods |
| configMap.index | object | `{"NUM_INDEXING_WORKERS":"2","VESPA_SEARCHER_THREADS":"4"}` | ConfigMap for env vars specific to "index" pod(s) |
| configMap.inference | object | `{}` | ConfigMap for env vars specific to "inference" pod(s) |
| configMap.ml | object | `{"LOG_LEVEL":"info","MIN_THREADS_ML_MODELS":"10"}` | Common ConfigMap to both index and inference pods |
| configMap.vespa | object | `{"VESPA_CONFIG_SERVER_HOST":"vespa-single","VESPA_HOST":"vespa-single","VESPA_PORT":"8081","VESPA_TENANT_PORT":"19071"}` | ConfigMap for vespa configuration env vars |
| configMap.vespa.VESPA_CONFIG_SERVER_HOST | string | `"vespa-single"` | Set to name of vespa "config" statefulset |
| configMap.vespa.VESPA_HOST | string | `"vespa-single"` | Set to name of vespa "content" statefulset |
| configMap.web | object | `{"DISABLE_LLM_CHUNK_FILTER":"True","DISABLE_LLM_QUERY_REPHRASE":"True","ENCRYPTION_KEY_SECRET":"TODO-ENCRYPTION_KEY_SECRET","QA_TIMEOUT":"120","SESSION_EXPIRE_TIME_SECONDS":"86400","WEB_DOMAIN":"http://localhost:3000"}` | ConfigMap for env vars specific to "web" pod(s) |
| configMap.worker | object | `{"CELERY_WORKER_LIGHT_CONCURRENCY":"32","CELERY_WORKER_LIGHT_PREFETCH_MULTIPLIER":"10"}` | ConfigMap for env vars specific to "worker" pod(s) |
| hotpatch.enabled | bool | `true` |  |
| hotpatch.vespaRedundancy | int | `1` |  |
| images.backend.repository | string | `"onyxdotapp/onyx-backend"` |  |
| images.model.repository | string | `"onyxdotapp/onyx-model-server"` |  |
| images.web.repository | string | `"onyxdotapp/onyx-web-server"` |  |
| index.affinity | object | `{}` |  |
| index.extraVolumeMounts | list | `[]` |  |
| index.extraVolumes | list | `[]` |  |
| index.replicaCount | int | `1` | Set number of inference pods. Untested >1 |
| index.resources.limits."nvidia.com/gpu" | string | `"4"` |  |
| index.resources.requests."nvidia.com/gpu" | string | `"4"` |  |
| index.resources.requests.cpu | string | `"0.1"` |  |
| index.resources.requests.memory | string | `"2Gi"` |  |
| index.runtimeClassName | string | `"nvidia"` |  |
| index.tolerations[0].effect | string | `"NoSchedule"` |  |
| index.tolerations[0].key | string | `"nvidia.com/gpu"` |  |
| index.tolerations[0].operator | string | `"Exists"` |  |
| inference.affinity | object | `{}` |  |
| inference.extraVolumeMounts | list | `[]` |  |
| inference.extraVolumes | list | `[]` |  |
| inference.replicaCount | int | `1` | Set number of inference pods. Untested >1 |
| inference.resources.limits."nvidia.com/gpu" | string | `"4"` |  |
| inference.resources.requests."nvidia.com/gpu" | string | `"4"` |  |
| inference.resources.requests.cpu | string | `"0.1"` |  |
| inference.resources.requests.memory | string | `"2Gi"` |  |
| inference.runtimeClassName | string | `"nvidia"` |  |
| inference.tolerations[0].effect | string | `"NoSchedule"` |  |
| inference.tolerations[0].key | string | `"nvidia.com/gpu"` |  |
| inference.tolerations[0].operator | string | `"Exists"` |  |
| ingress.annotations | object | `{}` |  |
| ingress.enabled | bool | `false` | Enable ingress |
| ingress.hostname | string | `""` | Set ingress domain name. Optional(If not specified, creates catch all with https disabled) |
| ingress.ingressClassName | string | `"traefik"` |  |
| ingress.tls | bool | `false` | Enable TLS |
| nameOverride | string | `""` | Override Release Name |
| redis.architecture | string | `"standalone"` |  |
| redis.auth.password | string | `"correct-horse-battery-staple"` |  |
| redis.auth.sentinel | bool | `false` |  |
| redis.enabled | bool | `true` |  |
| redis.master.resourcesPreset | string | `"medium"` |  |
| redis.master.revisionHistoryLimit | int | `2` |  |
| version_tag | string | `"v0.18.0-beta.2"` | Default version tag for container images |
| vespa.configMap.env.VESPA_CONFIGPROXY_JVMARGS | string | `"-Xms128M -Xmx1G"` |  |
| vespa.configMap.env.VESPA_CONFIGSERVER_JVMARGS | string | `"-Xms128M -Xmx1G"` |  |
| vespa.nameOverride | string | `"vespa"` |  |
| vespa.statefulSets.cfg.enabled | bool | `false` |  |
| vespa.statefulSets.cfg.replicaCount | int | `3` |  |
| vespa.statefulSets.cfg.resources.limits.cpu | string | `"5"` |  |
| vespa.statefulSets.cfg.resources.limits.memory | string | `"12Gi"` |  |
| vespa.statefulSets.cfg.resources.requests.cpu | string | `"3"` |  |
| vespa.statefulSets.cfg.resources.requests.memory | string | `"8Gi"` |  |
| vespa.statefulSets.cfg.volumes.var.accessModes[0] | string | `"ReadWriteOnce"` |  |
| vespa.statefulSets.cfg.volumes.var.className | string | `"gp3"` |  |
| vespa.statefulSets.cfg.volumes.var.path | string | `"/opt/vespa/var"` |  |
| vespa.statefulSets.cfg.volumes.var.size | string | `"20Gi"` |  |
| vespa.statefulSets.content.enabled | bool | `false` |  |
| vespa.statefulSets.content.ports[0] | string | `"8081"` |  |
| vespa.statefulSets.content.replicaCount | int | `3` |  |
| vespa.statefulSets.content.resources.limits.cpu | string | `"10"` |  |
| vespa.statefulSets.content.resources.limits.memory | string | `"18Gi"` |  |
| vespa.statefulSets.content.resources.requests.cpu | string | `"5"` |  |
| vespa.statefulSets.content.resources.requests.memory | string | `"10Gi"` |  |
| vespa.statefulSets.content.volumes.var.className | string | `"gp3"` |  |
| vespa.statefulSets.content.volumes.var.path | string | `"/opt/vespa/var"` |  |
| vespa.statefulSets.content.volumes.var.size | string | `"60Gi"` |  |
| vespa.statefulSets.single.enabled | bool | `true` |  |
| vespa.statefulSets.single.resources.limits.cpu | string | `"5"` |  |
| vespa.statefulSets.single.resources.limits.memory | string | `"14Gi"` |  |
| vespa.statefulSets.single.resources.requests.cpu | string | `"3"` |  |
| vespa.statefulSets.single.resources.requests.memory | string | `"10Gi"` |  |
| vespa.statefulSets.single.volumes.var.accessModes[0] | string | `"ReadWriteOnce"` |  |
| vespa.statefulSets.single.volumes.var.className | string | `"gp3"` |  |
| vespa.statefulSets.single.volumes.var.path | string | `"/opt/vespa/var"` |  |
| vespa.statefulSets.single.volumes.var.size | string | `"50Gi"` |  |
| web.replicaCount | int | `1` | Set number of web replicas |
| web.resources.limits | object | `{}` |  |
| web.resources.requests.cpu | string | `"0.1"` |  |
| web.resources.requests.memory | string | `"512Mi"` |  |
| worker.replicaCount | int | `1` | Set number of background worker pods. Currently breaks when > 1. |
| worker.resources.limits | object | `{}` |  |
| worker.resources.requests.cpu | string | `"2"` |  |
| worker.resources.requests.memory | string | `"4Gi"` |  |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)
