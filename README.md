# Ansible Kubernetes 单节点二进制部署项目

这是一个使用 Ansible 自动化部署 Kubernetes 集群的项目，采用二进制方式安装 Kubernetes 组件，适用于 Ubuntu 24.04 和 Ubuntu 22.04 系统。仅支持单节点部署，适用于测试环境，可以快速搭建一个完整的 Kubernetes 集群环境。

## 📋 项目特性

- ✅ **二进制安装方式**：使用 Kubernetes 官方二进制文件进行安装
- ✅ **单节点部署**：支持在单台服务器上部署完整的 Kubernetes 集群（Master + Worker）
- ✅ **完整的组件支持**：包含 etcd、kube-apiserver、kube-controller-manager、kube-scheduler、kubelet、kube-proxy 等核心组件
- ✅ **容器运行时选择**：支持 containerd 或 Docker 作为容器运行时
- ✅ **自动化证书管理**：自动生成和管理 Kubernetes 所需的 SSL 证书
- ✅ **插件支持**：包含 CoreDNS、cilium 等常用插件
- ✅ **配置灵活**：通过变量文件轻松配置集群参数

## 🏗️ 项目结构

```
.
├── ansible.cfg              # Ansible 配置文件
├── inventory/               # 主机清单目录
│   └── hosts.yml           # 主机清单文件
├── playbooks/              # Playbook 目录
│   ├── generate-config.yml # 生成配置文件（证书和 kubeconfig）
│   ├── install-master.yml  # 安装 Master 节点
│   └── install-worker.yml  # 安装 Worker 节点
├── roles/                  # Ansible 角色目录
│   ├── addons/             # Kubernetes 插件（CoreDNS、Flannel 等）
│   ├── configure/          # 配置生成（证书、kubeconfig）
│   ├── container-runtime/  # 容器运行时（containerd/Docker）
│   ├── etcd-single/        # etcd 单节点部署
│   ├── init-system/        # 系统初始化
│   ├── kube-apiserver/     # kube-apiserver 组件
│   ├── kube-controller-manager/ # kube-controller-manager 组件
│   ├── kube-proxy/         # kube-proxy 组件
│   ├── kube-scheduler/     # kube-scheduler 组件
│   └── kubelet/            # kubelet 组件
├── group_vars/             # 组变量目录
│   ├── all.yml            # 全局变量
│   ├── masters.yml        # Master 节点变量
│   └── workers.yml        # Worker 节点变量
├── host_vars/             # 主机变量目录
│   ├── master1.yml        # Master1 主机变量
│   ├── master2.yml        # Master2 主机变量
│   └── master3.yml        # Master3 主机变量
├── templates/             # 模板文件目录
├── files/                 # 文件目录
└── README.md              # 项目说明文档
```

## 📦 前置要求

### 控制节点要求

- **Ansible 版本**：>= 2.9
- **Python 版本**：>= 3.6
- **操作系统**：Linux/macOS

### 目标节点要求

- **操作系统**：Ubuntu 24.04
- **用户权限**：root 或具有 sudo 权限的用户
- **SSH 访问**：控制节点能够通过 SSH 访问目标节点
- **网络连接**：目标节点需要能够访问互联网以下载 Kubernetes 二进制文件

## 🚀 快速开始

### 1. 配置主机清单

编辑 `inventory/hosts.yml` 文件，配置目标服务器的 IP 地址和主机名：

```yaml
all:
  children:
    etcd:
      hosts:
        etcd:
          ansible_host: 192.168.0.115
          etcd_hostname: 'etcd.k8s.local'
    
    masters:
      hosts:
        master:
          ansible_host: 192.168.0.115
          hostname: 'master.k8s.local'
          etcd_hostname: 'etcd.k8s.local'
    
    workers:
      hosts:
        worker1:
          ansible_host: 192.168.1.30
```

### 2. 配置全局变量

编辑 `group_vars/all.yml` 文件，根据实际环境修改以下关键变量：

- `COMMON_K8S_IPS`：Kubernetes 节点的 IP 地址
- `container_runtime`：容器运行时（`containerd` 或 `docker`）
- `pod_network`：Pod 网络 CIDR
- `service_network`：Service 网络 CIDR
- `dns_clusterip`：CoreDNS 的 ClusterIP
- `proxy_url`：镜像下载代理地址（如需要）

### 3. 配置 SSH 访问

确保控制节点可以通过 SSH 访问目标节点配置免秘钥。

### 4. 生成配置文件

首先运行配置生成 playbook，生成 SSL 证书和 kubeconfig 文件：

```bash
ansible-playbook playbooks/generate-config.yml
```

### 5. 安装 Master 节点

运行 Master 节点安装 playbook：

```bash
ansible-playbook playbooks/install-master.yml
```

### 6. 安装 Worker 节点（可选）

如果需要添加 Worker 节点，运行：

```bash
ansible-playbook playbooks/install-worker.yml
```

## ⚙️ 配置说明

### 主要配置变量

在 `group_vars/all.yml` 中可以配置以下变量：

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `container_runtime` | 容器运行时 | `containerd` |
| `pod_network` | Pod 网络 CIDR | `10.244.0.0/16` |
| `service_network` | Service 网络 CIDR | `172.16.0.0/16` |
| `dns_clusterip` | CoreDNS ClusterIP | `172.16.0.10` |
| `time_server_address` | NTP 服务器地址 | `ntp.aliyun.com` |
| `proxy_enabled` | 是否启用代理 | `true` |
| `proxy_url` | 代理地址 | `socks5://192.168.0.100:7897` |

### Ansible 配置

`ansible.cfg` 文件中的主要配置：

- `inventory`：主机清单文件路径
- `remote_user`：远程用户（默认 root）
- `private_key_file`：SSH 私钥文件路径，默认是 ~/.ssh/id_rsa 文件

## 🔧 使用说明

### 检查连接

在运行 playbook 之前，可以先测试与目标节点的连接：

```bash
ansible all -m ping
```

### 查看主机清单

```bash
ansible-inventory --list
```

### 运行特定任务

如果需要只运行某个角色的任务，可以使用标签：

```bash
ansible-playbook playbooks/install-master.yml --tags "init-system"
```

## 📝 部署后的操作

### 验证集群状态

部署完成后，在 Master 节点上执行以下命令验证集群状态：

```bash
# 查看节点状态
kubectl get nodes

# 查看所有 Pod 状态
kubectl get pods --all-namespaces

# 查看集群信息
kubectl cluster-info
```

## 🛠️ 故障排查

### 常见问题

1. **SSH 连接失败**
   - 检查网络连通性
   - 确认 SSH 密钥配置正确
   - 检查 `ansible.cfg` 中的 `private_key_file` 路径

## 📄 许可证

本项目采用 MIT 许可证。

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## ⚠️ 注意事项

1. 本项目适用于学习和测试环境

---

**版本信息**：Ansible 1.34.2 | Kubernetes 二进制部署 | Ubuntu 24.04
