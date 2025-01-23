# Prerequisites

## cloudnative-pg/cloudnative-pg

cloudnative-pg is a `Level V - Auto Pilot` operator to handle PostgreSQL deployments in a reliable and HA fashion.

```
helm repo add cloudnative-pg https://cloudnative-pg.io/charts/
helm repo update cloudnative-pg
helm install cloudnative-pg cloudnative-pg/cloudnative-pg --create-namespace -n cloudnative-pg
```

## nvidia/gpu-operator

NVIDIA's gpu-operator handles injecting the nvidia runtime to all GPU-enabled cluster nodes, assigning appropriate labels to the nodes and in some host configurations may also handle installing appropriate drivers.

Driver installation requires the host to be running a compatible OS, and by default does not support vGPU due to those drivers not being available publicly. Additionally, nodes with drivers preinstalled should be labeled appropriately when they join the cluster.

The sample configuration below includes profiles for several NVIDIA Tesla cards, and tries to consider the "resource" as a unit of vRAM. The operator may/will consume 1 unit for node profiling, so every entry has an additional unit.

### Install
```
# values.gpu-operator.yaml
driver: { enabled: true }
toolkit: { enabled: true }
nfd: { enabled: true }
node-feature-discovery:
  worker:
    config:
      sources:
        custom:
          - name: gpu-timeslice
            labelsTemplate: "{{ range .pci.device }}nvidia.com/device-plugin.config=tesla-{{ .device }}{{ end }}"
            matchFeatures:
              - feature: "pci.device"
                matchExpressions:
                  class: { op: "InRegexp", value: ["^03"] }
                  vendor: ["10de"]
devicePlugin:
  config:
    create: true
    name: "device-plugin-configs"
    default: "any"
    data:
      # Tesla A16
      tesla-25b6: |
        version: v1
        flags: { migStrategy: "none" }
        sharing:
          timeSlicing:
            failRequestsGreaterThanOne: false
            resources:
              - name: "nvidia.com/gpu"
                replicas: 17
      # Telsa A40
      tesla-2235:
        version: v1
        flags: { migStrategy: "none" }
        sharing:
          timeSlicing:
            failRequestsGreaterThanOne: false
            resources:
              - name: "nvidia.com/gpu"
                replicas: 49
      # Telsa L4
      tesla-27b8:
        version: v1
        flags: { migStrategy: "none" }
        sharing:
          timeSlicing:
            failRequestsGreaterThanOne: false
            resources:
              - name: "nvidia.com/gpu"
                replicas: 25
      # Telsa L40S
      tesla-26b9:
        version: v1
        flags: { migStrategy: "none" }
        sharing:
          timeSlicing:
            failRequestsGreaterThanOne: false
            resources:
              - name: "nvidia.com/gpu"
                replicas: 49
```

```
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
helm repo update nvidia
helm install gpu-operator nvidia/gpu-operator --create-namespace -n gpu-operator -f values.gpu-operator.yaml
```

### GPU Node notes

The operator detects the presence of a GPU on a given node, and only attempts driver installation in that case. So no need to block the driver on non-GPU nodes.

The driver container images can be found at https://catalog.ngc.nvidia.com/orgs/nvidia/containers/driver/tags to determine compatible operating systems.

Automated driver installation can be prevented on a per-node basis by labeling the GPU node with the following:
```
kubectl label nodes $NODE nvidia.com/gpu.deploy.driver=false
```
[Reference](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html#preventing-installation-of-nvidia-gpu-driver-on-some-nodes)

GPU Nodes should be configured to come online with the following taint
```
nvidia.com/gpu: "true:NoSchedule"
```

Further labels/taints may be necessary for Cluster Autoscaler to manage related ASGs properly, this would include some tags on the ASG describing the available GPU resources.

## ollama-helm

Ollama is a locally hosted LLM engine. The model files can be expected to be quite large, so ensure that the root FS of your GPU nodes has sufficient space, or extend this configuration to store them on a RWX PVC.

### Install

```
# values.ollama-helm.yaml
ollama:
  gpu: { enabled: true, type: "nvidia" }
  models:
    pull:
      - llama3.1:8b-instruct-q8_0 # This model generally consumes ~8G of vRAM
    run:
      - llama3.1:8b-instruct-q8_0
  replicaCount: 1 # Increase for HA
  runtimeClassName: nvidia
  resources:
    requests: { cpu: "4", memory: "15Gi", "nvidia.com/gpu": "8" }
    requests: { cpu: "8", memory: "20Gi", "nvidia.com/gpu": "8" }
    # ^ nvidia.com/gpu should be adjusted according to the expected vRAM requirements of the model and queries
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: "nvidia.com/device-plugin.config"
                operator: "In"
                values:
                  - tesla-25b6 # Tesla A16
                  - tesla-2235 # Tesla A40
                  - tesla-27b8 # Tesla L4
                  - tesla-26b9 # Tesla L40S
                  # ^ This restricts Ollama to only run on nodes with the above GPUs, remove or add GPU IDs as necessary
```

```
helm repo add ollama-helm https://otwld.github.io/ollama-helm/
helm repo update ollama-helm
helm install ollama-helm ollama-helm/ollama-helm --create-namespace -n ollama -f values.ollama-helm.yaml
```

### Usage

The example above installs Ollama to the cluster and does not expose it publicly. It is available to other applications in the cluster by using the following configuration:
```
Base URL: http://ollama.ollama.svc.cluster.local:11434
API Key: <None>
Model: llama3.1:8b-instruct-q8_0
```
