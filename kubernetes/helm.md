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

## 自定义 Chart：双机推理服务

以双机推理服务为例，需要使用 DP、主从模式启动，因此使用 StatefulSet + Headless Service 表示

将现有的 yaml manifest 改造成 helm chart 模板

** 整理 chart 目录结构 **

创建一个 chart 目录

```
inference-chart/
├── Chart.yaml          # Chart 的元数据（名称、版本）
├── values.yaml         # 默认配置值
└── templates/          # 存放模板文件
    ├── _helpers.tpl    # 辅助模板（定义命名规则）
    ├── statefulset.yaml
    └── service.yaml
```

** 编写模板 **

`templates/service.yaml`

```yml
apiVersion: v1
kind: Service
metadata:
  # 使用 release 名称 + 服务名，避免多套部署时名字冲突
  name: {{include "inference.fullname" .}}
  labels:
    app.kubernetes.io/name: {{include "inference.name" .}}
    app.kubernetes.io/instance: {{.Release.Name}}
spec:
  type: ClusterIP
  clusterIP: None # 关键：Headless Service
  ports:
    - port: {{.Values.service.port}}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: {{include "inference.name" .}}
    app.kubernetes.io/instance: {{.Release.Name}}
```

`templates/statefulset.yaml`

```yml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: {{include "inference.fullname" .}}
spec:
  serviceName: {{include "inference.fullname" .}} # 关联 Headless Service
  replicas: {{.Values.replicaCount}} # 双机通常设为 2
  selector:
    matchLabels:
      app.kubernetes.io/name: {{include "inference.name" .}}
      app.kubernetes.io/instance: {{.Release.Name}}
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{include "inference.name" .}}
        app.kubernetes.io/instance: {{.Release.Name}}
    spec:
      containers:
        - name: inference
          image: "{{.Values.image.repository}}:{{ .Values.image.tag }}"
          ports:
            - name: http
              containerPort: {{.Values.service.port}}
          resources:
            {{- toYaml .Values.resources | nindent 12}}
          # 双机推理通常需要区分 Pod 身份 (0 或 1)
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
```

** 定义默认值 **

`values.yaml`

```yml
replicaCount: 2 # 双机

image:
  repository: my-registry/inference-engine
  tag: "v1.0.0"

service:
  port: 8080

resources:
  limits:
    cpu: 4
    memory: 8Gi
    nvidia.com/gpu: 1 # 假设每机需要 1 张卡
  requests:
    cpu: 2
    memory: 4Gi
```

** 使用 chart 部署多套服务 **

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

优化 todo:

- nodePort 端口范围
- 全量重启
- 预热保护，`initialDelaySeconds: 600`