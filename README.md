# onek3s

基于 k3s 快速搭建单体应用环境，包含数据库（mariadb）、对象存储（minio）、SSL证书颁发（letsencrypt）及可选的 Ingress 组件（ingress-nginx）。

## 安装 k3s

### 标准安装

```bash
curl -sfL https://get.k3s.io | sh -
```

### 禁用 `traefik` 组件安装

```bash
curl -sfL https://get.k3s.io | sh - --disable=traefik
```

## 配置 k3s

### 配置镜像节点

```bash
touch /etc/rancher/k3s/registries.yaml

cat > /etc/rancher/k3s/registries.yaml <<EOF
mirrors:
  docker.io:
    endpoint:
      - "http://mycustomreg.com:5000"
EOF
```

### 配置镜像节点及认证（不推荐）

```bash
touch /etc/rancher/k3s/registries.yaml

cat > /etc/rancher/k3s/registries.yaml <<EOF
mirrors:
  docker.io:
    endpoint:
      - "http://mycustomreg.com:5000"
configs:
  "mycustomreg:5000":
    auth:
      username: xxxxxx # this is the registry username
      password: xxxxxx # this is the registry password
EOF
```

### 创建镜像节点认证密钥

在 github 开发者设置中创建 `Personal Access Token`，选中 `read packages` 权限即可

```bash
kubectl create secret docker-registry ghcr --docker-server=ghcr.io --docker-username=yourname --docker-password=your_github_pat
```

使用时在 deployment 中配置

```yaml
imagePullSecrets:
    - name: ghcr
```

## 使用 helm 安装 onek3s

```bash
# 在 onek3s 目录中执行
helm install onek3s . \
    --set minio.apiIngress.hostname=oss.example.org \
    --set clusterIssuer.enabled=true \
    --set clusterIssuer.email=admin@example.org
```

或者直接编辑 `values.yaml` 文件后执行 install