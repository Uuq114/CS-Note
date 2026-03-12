# Helm

## Helm Chart 简介
Helm 被称为 Kubernetes 的 “包管理器”（类似于 Linux 的 apt 或 yum）。

- Chart：是 Helm 的打包格式，包含了一个应用运行所需的所有资源定义（Deployment, Service, ConfigMap 等）。
- Template（模板）：Chart 中的 YAML 文件不是静态的，而是包含变量（如 `{{.Values.replicaCount}}`）的模板。
- Values（值）：一个 `values.yaml` 文件，用于定义模板中变量的具体值。
- Release（发布）：每次执行 `helm install` 就在集群中运行了一个 Chart 的实例。

核心优势：
- 参数化：一套模板，通过修改 values 即可部署不同配置的应用。
- 版本管理：可以回滚、升级、查看历史版本。
- 依赖管理：可以引用其他 Chart（如依赖 MySQL、Redis）。

## 自定义 Helm Chart：双机推理服务

以双机推理服务为例，需要使用 DP、主从模式启动，因此使用 StatefulSet + Headless Service 表示

将现有的 yaml manifest 改造成 helm chart 模板

** 总结 **

| 文件                              | 作用                                                                             |
| --------------------------------- | -------------------------------------------------------------------------------- |
| `Chart.yaml`                      | Chart 元数据，定义名称、版本、描述等                                             |
| `values.yaml`                     | 默认配置值，所有可参数化的变量集中管理，部署时可通过 `--set` 覆盖                |
| `templates/_helpers.tpl`          | 辅助模板，定义可复用的命名规则，供其他模板 `include` 调用                        |
| `templates/statefulset.yaml`      | 核心工作负载定义，包含容器、启动命令、资源限制和主从启动逻辑                     |
| `templates/headless-service.yaml` | Headless Service（`clusterIP: None`），为 Pod 提供稳定 DNS，实现主从节点互相发现 |
| `templates/nodeport-service.yaml` | NodePort Service，将推理服务暴露到集群外部                                       |

数据流：`values.yaml` 提供参数 → `_helpers.tpl` 生成统一命名 → `templates/*.yaml` 渲染出最终的 K8s 资源清单

整个服务的流量是 `client -> ingress -> loadbalancer service -> (many) nodeport service -> pod`，为了让 lb svc 能起到负载均衡作用，将 nodeport 的 label 拆成了 `app` 和 `instance_id`，同种服务的 `app`label 是相同的，lb svc 引用这个 label

** 整理 chart 目录结构 **

创建一个 chart 目录

```
.
├── deploy-instance.sh
├── minimax-inference
│   ├── Chart.yaml                    # Chart 的元数据（名称、版本）
│   ├── templates
│   │   ├── headless-service.yaml
│   │   ├── _helpers.tpl              # 辅助模板（定义命名规则）
│   │   ├── nodeport-service.yaml
│   │   └── statefulset.yaml
│   └── values.yaml                   # 默认配置值
└── remove-all-instance.sh
```

** 编写模板 **

`Chart.yaml`

```yml
apiVersion: v2
name: minimax-inference
description: Helm chart for deploying MiniMax M2.5 inference service with vLLM on Ascend NPU
type: application
version: 1.0.0
appVersion: "v0.14.0rc1"
keywords:
  - minimax
  - vllm
  - inference
  - ascend
  - npu
maintainers:
  - name: LLM Team
```

** 模板具体内容 **

`templates/headless-service.yaml`

```yml
---
# Headless Service for stable network identity
apiVersion: v1
kind: Service
metadata:
  name: {{include "minimax.headlessServiceName" .}}
  namespace: {{.Values.namespace}}
spec:
  clusterIP: None
  selector:
    app: minimax-m25-infer
    instance-id: "{{.Values.instanceId}}"
  ports:
    - port: {{.Values.ports.vllm}}
      name: http
    - port: {{.Values.ports.rpc}}
      name: rpc
```

`templates/statefulset.yaml`

```yml
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: {{include "minimax.name" .}}
  namespace: {{.Values.namespace}}
spec:
  serviceName: {{include "minimax.headlessServiceName" .}}
  replicas: {{.Values.replicas}}
  selector:
    matchLabels:
      app: minimax-m25-infer
      instance-id: "{{.Values.instanceId}}"
  template:
    metadata:
      labels:
        app: minimax-m25-infer
        instance-id: "{{.Values.instanceId}}"
        app.kubernetes.io/name: helm-minimax
    spec:
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      nodeSelector:
        accelerator/huawei-npu: ascend-1980
      imagePullSecrets:
        - name: default-secret
      containers:
        - name: vllm
          image: "{{.Values.image.repository}}:{{ .Values.image.tag }}"
          imagePullPolicy: {{.Values.image.pullPolicy}}
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
          command: ["/bin/sh", "-c"]
          args:
            - |
              #!/bin/sh
              set -e

              # add patch for minimax 2.5
              cd /vllm-workspace/vllm/
              cp {{.Values.vllm.modelPath}}/patch/0001-MiniMax-M2-adapt-Ascend-fp8-loading-and-qk-norm-path.patch .
              git apply --check ./0001-MiniMax-M2-adapt-Ascend-fp8-loading-and-qk-norm-path.patch && \
              git apply ./0001-MiniMax-M2-adapt-Ascend-fp8-loading-and-qk-norm-path.patch

              NIC_NAME="{{.Values.network.nicName}}"
              MODEL_PATH="{{.Values.vllm.modelPath}}"

              # Get local node IP (hostNetwork => same as physical IP)
              LOCAL_IP=$(hostname -I | awk '{print $1}')
              if [-z "$LOCAL_IP"]; then
                echo "ERROR: Failed to get local IP"
                exit 1
              fi
              echo "Local IP: $LOCAL_IP"

              # Resolve node0 IP dynamically
              NODE0_HOST="{{include"minimax.name".}}-0.{{ include"minimax.headlessServiceName". }}.{{ .Values.namespace }}.svc.cluster.local"
              echo "Resolving $NODE0_HOST ..."
              for i in $(seq 1 30); do
                if NODE0_IP=$(getent hosts "$NODE0_HOST" 2>/dev/null | awk '{print $1}' | head -n1); then
                  if [-n "$NODE0_IP"]; then
                    echo "Resolved NODE0_IP: $NODE0_IP"
                    break
                  fi
                fi
                echo "Waiting for $NODE0_HOST... ($i/30)"
                sleep 2
              done

              if [-z "$NODE0_IP"]; then
                echo "ERROR: Failed to resolve NODE0_IP after 60 seconds"
                exit 1
              fi

              # envs
              export HCCL_IF_IP="$LOCAL_IP"
              export GLOO_SOCKET_IFNAME="$NIC_NAME"
              export TP_SOCKET_IFNAME="$NIC_NAME"
              export HCCL_SOCKET_IFNAME="$NIC_NAME"
              export HCCL_BUFFSIZE=1024
              export ASCEND_RT_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
              export HCCL_OP_EXPANSION_MODE="AIV"
              export PYTORCH_NPU_ALLOC_CONF=expandable_segments:True
              export OMP_PROC_BIND=false
              export OMP_NUM_THREADS=100
              export VLLM_USE_V1=1
              export VLLM_ASCEND_ENABLE_FLASHCOMM1=1
              export HCCL_INTRA_PCIE_ENABLE=1
              export HCCL_INTRA_ROCE_ENABLE=0
              export VLLM_TORCH_PROFILER_WITH_STACK=0
              export VLLM_TORCH_PROFILER_DIR="/data/profiling/minimax"

              # rank 0-1
              case "$POD_NAME" in
                {{include "minimax.name" .}}-0)
                  echo "Running as MASTER (rank 0)..."
                  exec vllm serve "$MODEL_PATH" \
                    --served-model-name "{{.Values.vllm.servedModelName}}" \
                    --host 0.0.0.0 \
                    --port {{.Values.ports.vllm}} \
                    --tensor-parallel-size {{.Values.vllm.tensorParallelSize}} \
                    --data-parallel-size {{.Values.vllm.dataParallelSize}} \
                    --data-parallel-size-local 1 \
                    --data-parallel-start-rank 0 \
                    --data-parallel-address "$NODE0_IP" \
                    --data-parallel-rpc-port {{.Values.ports.rpc}} \
                    --max-num-seqs {{.Values.vllm.maxNumSeqs}} \
                    --max-num-batched-tokens {{.Values.vllm.maxNumBatchedTokens}} \
                    --gpu-memory-utilization {{.Values.vllm.gpuMemoryUtilization}} \
                    --enable-expert-parallel \
                    --enable-chunked-prefill \
                    --enable-prefix-caching \
                    --trust-remote-code \
                    --compilation-config '{"cudagraph_mode":"FULL_DECODE_ONLY"}' \
                    --mm_processor_cache_type="shm" \
                    --async-scheduling \
                    --enable-auto-tool-choice \
                    --tool-call-parser minimax_m2 \
                    --reasoning-parser minimax_m2_append_think \
                    --additional-config '{"enable_cpu_binding":true}'
                  ;;

                {{include "minimax.name" .}}-1)
                  echo "Running as WORKER (rank 1)..."
                  exec vllm serve "$MODEL_PATH" \
                    --served-model-name "{{.Values.vllm.servedModelName}}" \
                    --host 0.0.0.0 \
                    --port {{.Values.ports.vllm}} \
                    --headless \
                    --tensor-parallel-size {{.Values.vllm.tensorParallelSize}} \
                    --data-parallel-size {{.Values.vllm.dataParallelSize}} \
                    --data-parallel-size-local 1 \
                    --data-parallel-start-rank 1 \
                    --data-parallel-address "$NODE0_IP" \
                    --data-parallel-rpc-port {{.Values.ports.rpc}} \
                    --max-num-seqs {{.Values.vllm.maxNumSeqs}} \
                    --max-num-batched-tokens {{.Values.vllm.maxNumBatchedTokens}} \
                    --gpu-memory-utilization {{.Values.vllm.gpuMemoryUtilization}} \
                    --enable-expert-parallel \
                    --enable-chunked-prefill \
                    --enable-prefix-caching \
                    --trust-remote-code \
                    --compilation-config '{"cudagraph_mode":"FULL_DECODE_ONLY"}' \
                    --mm_processor_cache_type="shm" \
                    --async-scheduling \
                    --enable-auto-tool-choice \
                    --tool-call-parser minimax_m2 \
                    --reasoning-parser minimax_m2_append_think \
                    --additional-config '{"enable_cpu_binding":true}'
                  ;;

                *)
                  echo "ERROR: Unexpected hostname: $POD_NAME"
                  exit 1
                  ;;
              esac
          resources:
            limits:
              cpu: "172"
              huawei.com/ascend-1980: "8"
              memory: "1350Gi"
            requests:
              cpu: "172"
              huawei.com/ascend-1980: "8"
              memory: "1350Gi"
          volumeMounts:
            - name: llm-service-data
              mountPath: /data
            - name: shm
              mountPath: /dev/shm
      volumes:
        - name: llm-service-data
          persistentVolumeClaim:
            claimName: {{.Values.volumes.pvcName}}
        - name: shm
          emptyDir:
            medium: Memory
            sizeLimit: {{.Values.volumes.shmSize}}
```

`templates/nodeport-service.yaml`

```yml
---
# NodePort Service for external access
apiVersion: v1
kind: Service
metadata:
  name: {{include "minimax.nodeportServiceName" .}}
  namespace: {{.Values.namespace}}
  labels:
    app: minimax-m25-infer-{{.Values.instanceId}}
    instance: "{{.Values.instanceId}}"
spec:
  type: NodePort
  selector:
    app: minimax-m25-infer-{{.Values.instanceId}}
  ports:
    - port: {{.Values.ports.vllm}}
      targetPort: {{.Values.ports.vllm}}
      {{- if .Values.ports.nodePort}}
      nodePort: {{.Values.ports.nodePort}}
      {{- end}}
      protocol: TCP
      name: vllm
```

** 定义可复用模板片段 **

通过 `{{- define "mychart.name" -}}` 定义命名模板，然后在其他模板文件中用 `{{ include "mychart.name" . }}` 调用

`templates/_helpers.tpl`

```tpl
{{/*
Expand the name of the chart.
*/}}
{{- define "minimax.name" -}}
minimax-m25-infer-{{.Values.instanceId}}
{{- end}}

{{/*
Headless service name
*/}}
{{- define "minimax.headlessServiceName" -}}
minimax-m25-headless-{{.Values.instanceId}}
{{- end}}

{{/*
NodePort service name
*/}}
{{- define "minimax.nodeportServiceName" -}}
minimax-m25-nodeport-{{.Values.instanceId}}
{{- end}}

{{/*
Common labels
*/}}
{{- define "minimax.labels" -}}
app: minimax-m25-infer-{{.Values.instanceId}}
instance: {{.Values.instanceId}}
{{- end}}

{{/*
Selector labels
*/}}
{{- define "minimax.selectorLabels" -}}
app: minimax-m25-infer-{{.Values.instanceId}}
{{- end}}
```

** 定义默认值 **

`values.yml`

```yml
# Instance identifier (01-10)
instanceId: "01"

# Namespace
namespace: llm

# Number of replicas (always 2 for data-parallel)
replicas: 2

# Container image
image:
  repository: <repo>/<user>/vllm-ascend
  tag: v0.14.0rc1
  pullPolicy: IfNotPresent

# Image pull secrets
imagePullSecrets:
  - name: default-secret

# Node selector
nodeSelector:
  accelerator/huawei-npu: ascend-1980

# Resources
resources:
  limits:
    cpu: "172"
    huawei.com/ascend-1980: "8"
    memory: "1350Gi"
  requests:
    cpu: "172"
    huawei.com/ascend-1980: "8"
    memory: "1350Gi"

# Ports
ports:
  vllm: 20004
  rpc: 2347
  nodePort: null  # null = auto-assign, or specify like 30001

# Volumes
volumes:
  pvcName: pvc-llm-service-data
  shmSize: 1200Gi

# vLLM configuration
vllm:
  modelPath: /data/weights/model_scope/MiniMax/MiniMax-M2.5
  servedModelName: minimax2_5
  tensorParallelSize: 8
  dataParallelSize: 2
  maxNumSeqs: 128
  maxNumBatchedTokens: 65536
  gpuMemoryUtilization: 0.92

# Network interface for HCCL
network:
  nicName: enp67s0f5
```

** 使用 helm install 部署 **

```bash
helm install minimax-m25-01 ./minimax-inference --set instanceId=01 --namespace llm
helm uninstall minimax-m25-01 -n llm
```

** 使用 helmfile 部署（待验证）**

helmfile.yaml

```yml
# helmfile.yaml
releases:
  {{- $sets := list "01" "02" "03" "04" "05" "06" "07" "08" "09" "10"}}
  {{- range $idx, $name := $sets}}
  - name: inference-set-{{$name}}
    namespace: llm
    chart: local/inference-ascend
    values:
      - values/common.yaml
      - values/set-{{$name}}.yaml
    # 端口偏移：每套 +100
    set:
      - name: service.portOffset
        value: {{mul $idx 100}}
      - name: fullnameOverride
        value: inference-set-{{$name}}
  {{- end}}
```

部署步骤

```bash
# 1. 预览所有变更（确认端口 / 资源无冲突）
helmfile diff

# 2. 分批部署（避免存储 / 网络瞬时压力）
# 先部署 3 套，观察节点资源
helmfile -l name=inference-set-0{1,2,3} apply

# 3. 确认资源充足后，部署剩余 7 套
helmfile -l name=inference-set-0{4,5,6,7,8,9,10} apply

# 4. 验证调度结果（确认每套双机在不同节点）
kubectl get pods -n llm -l app.kubernetes.io/part-of=inference -o wide
# 输出示例：
# inference-set-01-0   ...   node-gpu-01
# inference-set-01-1   ...   node-gpu-02  ✅ 不同节点
# inference-set-02-0   ...   node-gpu-03
# inference-set-02-1   ...   node-gpu-04  ✅ 不同节点
```

优化 TODO:

- nodePort 端口范围
- 全量重启
- 预热保护，`initialDelaySeconds: 600`