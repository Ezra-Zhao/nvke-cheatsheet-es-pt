# NVKE 本地评估演练 | NVKE local evaluation drill: quickstart to a Ready cluster

没有 GPU 硬件也能把 NVKE 评估一遍，照原仓 quickstart 走：

1. **备机器**：准备一台 Linux 主机或虚拟机（amd64 或 arm64），确认至少 6 CPU、12 GiB 空闲内存、100 GiB 空闲磁盘；装好 Docker，并确认有虚拟 TPM 支持。只有 Mac 的话，按原仓本地站点指南用 Colima 虚拟机。
2. **跑 quickstart**：照原仓 docs 的 quickstart-local 一步步来，把开发版管理平面在模拟机上拉起来。这条路不需要 NVIDIA 账号。
3. **建工作负载集群**：用管理平面建一个工作负载集群，走完创建流程。
4. **验收**：确认工作负载集群的节点上报 `Ready`。到这一步，评估的主线就通了。
5. **回头看安全卡**：在模拟环境里先找到 API 鉴权与受保护访问的配置入口（原仓 operate 文档的 api-authentication 一节），心里记牢：预览版默认鉴权关闭、走明文 HTTP——真上自己基础设施时，这是第一个要改的默认值。
6. **记下限制再决定**：把诚实标注区那几条（预览版不建议生产、只有 NICo 一家、管理集群不自升级、FDE 镜像只供评估）对照自己的场景过一遍，再决定要不要走正式安装规划。
