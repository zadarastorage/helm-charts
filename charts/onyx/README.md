

# onyx

![Version: 0.0.10](https://img.shields.io/badge/Version-0.0.10-informational?style=flat-square) ![AppVersion: v0.21.1](https://img.shields.io/badge/AppVersion-v0.21.1-informational?style=flat-square)

Gen-AI Chat for Teams - Think ChatGPT if it had access to your team's unique knowledge.

**Homepage:** <https://www.onyx.app/>

## Cluster Prerequisites

| State | Chart | Repository | Description |
| ----- | ----- | ---------- | ----------- |
| Required by default | [cloudnative-pg](https://github.com/cloudnative-pg/charts?tab=readme-ov-file#operator-chart) | https://cloudnative-pg.io/charts/ | This operator is used to manage the Postgresql deployment and ensure redundancy to the database. <br>It can be disabled, but another solution would need to be configured. |
| Optional/Recommended | ollama | https://otwld.github.io/ollama-helm/ | Onyx supports multiple LLM solutions, we're recommending Ollama by default. |
| Optional/Recommended | [gpu-operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html#procedure) | https://helm.ngc.nvidia.com/nvidia | This is necessary prior to installing ollama to the cluster. |

[Prerequisite Installation Reference](docs/prerequisites.md)

## Install

```
helm repo add zadara https://zadarastorage.github.io/helm-charts
helm repo update zadara
helm install onyx zadara/onyx --create-namespace -n onyx
```

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| eric-zadara |  | <https://github.com/eric-zadara> |

## Source Code

* <https://github.com/zadarastorage/helm-charts/blob/main/charts/onyx>
* <https://github.com/onyx-dot-app/onyx>

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://charts.bitnami.com/bitnami | redis | 20.1.7 |
| https://cloudnative-pg.github.io/charts | cnpg(cluster) | 0.0.11 |
| https://zadarastorage.github.io/helm-charts | vespa(vespa-ai) | 0.0.2 |

## Values tl;dr

Values examples can be found in the [examples/](https://github.com/zadarastorage/helm-charts/tree/main/charts/onyx/examples) folder.<br>
This chart favors single-replicas for all configurations except for the Postgresql Database by default, the [values.ha.yaml](https://github.com/zadarastorage/helm-charts/tree/main/charts/onyx/examples/values.ha.yaml) example shows the minimum configuration needed to scale up all compatible pods.<br>
Some services do not play nice when scaled past 1 replica, they are noted in the full values documentation below.

## Values

### Backend api

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| api.replicaCount | int | `1` | Set number of api replicas |

### Resources

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| api.resources.limits | object | `{}` | Resource limits for API pods |
| api.resources.requests | object | `{"cpu":"0.1","memory":"1Gi"}` | Resource requests for API pods |
| index.resources.limits | object | `{"nvidia.com/gpu":"4"}` | Resource limits for index pods |
| index.resources.requests | object | `{"cpu":"0.1","memory":"2Gi","nvidia.com/gpu":"4"}` | Resource requests for index pods |
| inference.resources.limits | object | `{"nvidia.com/gpu":"4"}` | Resource limits for inference pods |
| inference.resources.requests | object | `{"cpu":"0.1","memory":"2Gi","nvidia.com/gpu":"4"}` | Resource requests for inference pods |
| web.resources.limits | object | `{}` | Resource limits for web pods |
| web.resources.requests | object | `{"cpu":"0.1","memory":"512Mi"}` | Resource requests for web pods |
| worker.resources.limits | object | `{}` | Resource limits for worker pods |
| worker.resources.requests | object | `{"cpu":"2","memory":"4Gi"}` | Resource requests for worker pods |

### Backend api/worker

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| configMap.api | object | `{}` | ConfigMap for env vars specific to "api" pod(s) |
| configMap.worker | object | `{"CELERY_WORKER_LIGHT_CONCURRENCY":"32","CELERY_WORKER_LIGHT_PREFETCH_MULTIPLIER":"10"}` | ConfigMap for env vars specific to "worker" pod(s) |

### Auth

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| configMap.auth.AUTH_TYPE | string | `"basic"` | Auth mode to operate in [disabled | google_oauth | basic]. https://docs.onyx.app/configuration_guide#auth-type |
| configMap.auth.GOOGLE_OAUTH_CLIENT_ID | string | `""` | Client ID for Google OAuth authentication. https://docs.onyx.app/configuration_guide#google-oauth-client-id |
| configMap.auth.GOOGLE_OAUTH_CLIENT_SECRET | string | `""` | Client Secret for Google OAuth authentication. https://docs.onyx.app/configuration_guide#google-oauth-client-secret |

### Embedding and Indexing

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| configMap.index | object | `{"NUM_INDEXING_WORKERS":"2","VESPA_SEARCHER_THREADS":"4"}` | ConfigMap for env vars specific to "index" pod(s) |
| configMap.inference | object | `{}` | ConfigMap for env vars specific to "inference" pod(s) |
| configMap.ml | object | `{"LOG_LEVEL":"info","MIN_THREADS_ML_MODELS":"10"}` | Common ConfigMap to both index and inference pods |
| images.model | object | `{"repository":"onyxdotapp/onyx-model-server"}` | Docker image used for the indexing and embedding model server |
| index.replicaCount | int | `1` | Set number of inference pods. Untested >1 |
| inference.replicaCount | int | `1` | Set number of inference pods. Untested >1 |

### Vespa-ai Embedding Database

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| configMap.vespa.VESPA_CONFIG_SERVER_HOST | string | `"vespa-single"` | Set to name of vespa "config" statefulset |
| configMap.vespa.VESPA_HOST | string | `"vespa-single"` | Set to name of vespa "content" statefulset |
| vespa.configMap.env.VESPA_CONFIGPROXY_JVMARGS | string | `"-Xms128M -Xmx1G"` | JVMARGS env var for Vespa config servers |
| vespa.configMap.env.VESPA_CONFIGSERVER_JVMARGS | string | `"-Xms128M -Xmx1G"` | JVMARGS env var for Vespa config servers |
| vespa.nameOverride | string | `"vespa"` | Override the resource prefix used |
| vespa.statefulSets.cfg.enabled | bool | `false` | Enable preconfigured statefulSet "cfg" |
| vespa.statefulSets.cfg.resources | object | `{"limits":{"cpu":"5","memory":"12Gi"},"requests":{"cpu":"3","memory":"8Gi"}}` | Resource configuration for "cfg" statefulset |
| vespa.statefulSets.content.enabled | bool | `false` | Enable preconfigured statefulSet "content" |
| vespa.statefulSets.content.resources | object | `{"limits":{"cpu":"10","memory":"18Gi"},"requests":{"cpu":"5","memory":"10Gi"}}` | Resource configuration for "cfg" statefulset |
| vespa.statefulSets.single.enabled | bool | `true` | Enable preconfigured statefulSet "single" |
| vespa.statefulSets.single.resources | object | `{"limits":{"cpu":"5","memory":"14Gi"},"requests":{"cpu":"3","memory":"10Gi"}}` | Resource configuration for a single-replica statefulset |

### Frontend web

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| configMap.web | object | `{"DISABLE_LLM_CHUNK_FILTER":"True","DISABLE_LLM_QUERY_REPHRASE":"True","ENCRYPTION_KEY_SECRET":"TODO-ENCRYPTION_KEY_SECRET","QA_TIMEOUT":"120","SESSION_EXPIRE_TIME_SECONDS":"86400","WEB_DOMAIN":"http://localhost:3000"}` | ConfigMap for env vars specific to "web" pod(s) |
| images.web | object | `{"repository":"onyxdotapp/onyx-web-server"}` | Docker image used for frontend web pod |
| web.replicaCount | int | `1` | Set number of web replicas |

### Onyx hotpatches

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| hotpatch.enabled | bool | `true` | Enables the hotpatching logic. Primarily used to augment vespa configurations and adjust some connector configurations |
| hotpatch.vespaRedundancy | int | `1` | Configure how many copies of each embedding should be stored within Vespa |

### Ingress configuration

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| ingress.annotations | object | `{}` | Optional addition annotations for the ingress configuration |
| ingress.enabled | bool | `false` | Enable ingress |
| ingress.hostname | string | `""` | Set ingress domain name. Optional(If not specified, creates catch all with https disabled) |
| ingress.ingressClassName | string | `"traefik"` | Set name of ingress controller |
| ingress.tls | bool | `false` | Enable TLS |

### Backend worker

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| worker.replicaCount | int | `1` | Set number of background worker pods. Currently breaks when > 1. |

### Other Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| api.updateStrategy.type | string | `"RollingUpdate"` |  |
| cnpg.backups.enabled | bool | `false` |  |
| cnpg.cluster.affinity.topologyKey | string | `"kubernetes.io/hostname"` |  |
| cnpg.cluster.instances | int | `3` | Number of psql replicas. 1 is master, N-1 are replica |
| cnpg.cluster.monitoring.additionalLabels.release | string | `"victoria-metrics-k8s-stack"` |  |
| cnpg.cluster.monitoring.enabled | bool | `false` |  |
| cnpg.cluster.postgresql.max_connections | string | `"500"` | Max psql connections. Default was 100 |
| cnpg.cluster.postgresql.max_locks_per_transaction | string | `"128"` | Max locks per transaction. Default was 64 |
| cnpg.enabled | bool | `true` | Enable preconfigured cloudnative-pg psql configuration |
| cnpg.mode | string | `"standalone"` |  |
| cnpg.pooler.enabled | bool | `true` |  |
| cnpg.pooler.instances | int | `2` | Number of psql poolers |
| cnpg.pooler.monitoring.additionalLabels.release | string | `"victoria-metrics-k8s-stack"` |  |
| cnpg.pooler.monitoring.enabled | bool | `false` |  |
| cnpg.pooler.parameters.default_pool_size | string | `"50"` | Pool size. Default was 25 |
| cnpg.pooler.parameters.max_client_conn | string | `"1000"` | Max client connections, default was 1000 |
| cnpg.pooler.parameters.reserve_pool_size | string | `"25"` | Reservice pool size, default was 0/disabled |
| cnpg.type | string | `"postgresql"` |  |
| configMap.global | object | `{}` | ConfigMap for env vars for all onyx pods |
| configMap.vespa.VESPA_PORT | string | `"8081"` | Default vespa content port |
| configMap.vespa.VESPA_TENANT_PORT | string | `"19071"` | Default vespa configuration port |
| images.backend | object | `{"repository":"onyxdotapp/onyx-backend"}` | Docker image used for both backend API and worker pods |
| index.affinity | object | `{}` |  |
| index.extraVolumeMounts | list | `[]` |  |
| index.extraVolumes | list | `[]` |  |
| index.runtimeClassName | string | `"nvidia"` |  |
| index.tolerations[0].effect | string | `"NoSchedule"` |  |
| index.tolerations[0].key | string | `"nvidia.com/gpu"` |  |
| index.tolerations[0].operator | string | `"Exists"` |  |
| index.updateStrategy.type | string | `"Recreate"` |  |
| inference.affinity | object | `{}` |  |
| inference.extraVolumeMounts | list | `[]` |  |
| inference.extraVolumes | list | `[]` |  |
| inference.runtimeClassName | string | `"nvidia"` |  |
| inference.tolerations[0].effect | string | `"NoSchedule"` |  |
| inference.tolerations[0].key | string | `"nvidia.com/gpu"` |  |
| inference.tolerations[0].operator | string | `"Exists"` |  |
| inference.updateStrategy.type | string | `"Recreate"` |  |
| nameOverride | string | `""` | Override Release Name |
| redis.architecture | string | `"standalone"` |  |
| redis.auth.password | string | `"correct-horse-battery-staple"` |  |
| redis.auth.sentinel | bool | `false` |  |
| redis.enabled | bool | `true` | Enable preconfigured redis configuration |
| redis.master.resourcesPreset | string | `"medium"` |  |
| redis.master.revisionHistoryLimit | int | `2` |  |
| vespa.statefulSets.cfg.replicaCount | int | `3` |  |
| vespa.statefulSets.cfg.volumes.var.accessModes[0] | string | `"ReadWriteOnce"` |  |
| vespa.statefulSets.cfg.volumes.var.className | string | `"gp3"` |  |
| vespa.statefulSets.cfg.volumes.var.path | string | `"/opt/vespa/var"` |  |
| vespa.statefulSets.cfg.volumes.var.size | string | `"20Gi"` |  |
| vespa.statefulSets.content.ports[0] | string | `"8081"` |  |
| vespa.statefulSets.content.replicaCount | int | `3` |  |
| vespa.statefulSets.content.volumes.var.className | string | `"gp3"` |  |
| vespa.statefulSets.content.volumes.var.path | string | `"/opt/vespa/var"` |  |
| vespa.statefulSets.content.volumes.var.size | string | `"60Gi"` |  |
| vespa.statefulSets.single.volumes.var.accessModes[0] | string | `"ReadWriteOnce"` |  |
| vespa.statefulSets.single.volumes.var.className | string | `"gp3"` |  |
| vespa.statefulSets.single.volumes.var.path | string | `"/opt/vespa/var"` |  |
| vespa.statefulSets.single.volumes.var.size | string | `"50Gi"` |  |
| web.updateStrategy.type | string | `"RollingUpdate"` |  |
| worker.updateStrategy.type | string | `"Recreate"` |  |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)
