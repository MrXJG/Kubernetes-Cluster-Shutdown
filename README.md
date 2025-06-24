# Kubernetes 集群优雅关机脚本

## 📌 项目简介

本脚本提供了一套完整的 Kubernetes 集群安全关机方案，特别适用于维护、升级或计划性断电场景。通过有序的节点关闭流程和 Pod 驱逐机制，最大限度减少对业务的影响。

## ✨ 核心功能

- **有序关机流程**：先 Worker 节点，再非主 Master 节点，最后当前 Master 节点
- **智能 Pod 驱逐**：自动处理 Pod 中断预算(PDB)约束
- **重试机制**：可配置的驱逐重试次数和间隔
- **详细日志记录**：完整的操作过程跟踪
- **安全检查**：操作前验证节点可达性
- **重启指引**：提供清晰的集群恢复指南

## 🛠 适用场景

1. **计划性维护窗口**
2. **数据中心电力管理**
3. **集群升级/迁移**
4. **需要重启的安全补丁安装**
5. **硬件更换流程**

## 🚀 使用指南

### 前提条件
- Bash 环境
- 配置好 kubectl 且具有集群管理员权限
- 所有节点的 SSH 访问权限
- 所有节点的 sudo 权限

### 基本使用
```bash
chmod +x k8s-graceful-shutdown.sh
./k8s-graceful-shutdown.sh
```

### 配置说明
根据实际情况修改脚本中的变量：
```bash
WORKER_NODES=("work01" "work02" "work03")      # Worker 节点主机名
MASTER_NODES=("master01" "master02" "master03") # Master 节点主机名
MAX_EVICT_RETRY=3                              # 驱逐重试次数
FORCE_SHUTDOWN_TIMEOUT=300                     # 强制关机前等待时间(秒)
DRAIN_TIMEOUT=120                              # 驱逐操作超时时间(秒)
```

### 重启后恢复
集群重启后执行：
1. 恢复 Worker 节点调度：
   ```bash
   for node in work01 work02 work03; do kubectl uncordon $node; done
   ```
2. 检查集群状态：
   ```bash
   kubectl get nodes -o wide
   kubectl get pods -A -o wide | grep -Ev "Running|Completed"
   ```

## ⚠️ 重要注意事项

1. **预先测试**：务必在非生产环境测试
2. **数据备份**：确保 etcd 和关键数据已备份
3. **业务影响**：部分工作负载可能出现短暂中断
4. **执行时机**：建议在业务低峰期执行
5. **监控观察**：操作过程中密切关注集群指标


