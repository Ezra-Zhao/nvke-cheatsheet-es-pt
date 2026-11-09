# NVKE 速查表 | NVKE Cheatsheet | Guía rápida de NVKE

**在自己的 GPU 机器上开一家「托管 Kubernetes」 —— 但先看安全卡：预览版装完，API 鉴权默认是关着的**

> 一页看懂 NVIDIA 的 NVKE（NVIDIA Kubernetes Engine）：它是一个开源的管理平面。平台团队装上它，就能按需给每个客户开一个隔离的工作负载集群——各自独立的控制平面、etcd 和节点池；客户用标准 Kubernetes API 和 `kubectl` 操作，体验接近云厂商的托管 K8s，只是机器是你自己的 GPU 裸金属。节点入伙前先用 TPM 经 SPIFFE/SPIRE 证明身份，etcd 可以按集群加密。

- 上游项目：[dsx-ai-factory/nvke](https://github.com/dsx-ai-factory/nvke)（2026-11-09 实测 1,022 ★ / 119 forks，Apache-2.0；原仓明示为预览版）
- 平台：管理平面装在你自己的基础设施上；预览期唯一兼容的基础设施提供方是 NVIDIA Infra Controller（NICo）
- 本页结构：中文主文 → English → Español → Português

---

Apoyo y más: [GitHub Sponsors](https://github.com/sponsors/Ezra-Zhao) · [Gumroad — plantillas y cuadernos que uso](https://ezraflow28.gumroad.com) · [Ko-fi](https://ko-fi.com/ezra777zhao)

## 上线前必改的安全卡（先看这张，再看别的）

- 预览版装完之后，**API 鉴权默认关闭，API 走明文 HTTP**（原仓自述）。这是评估期的默认状态，不是让你带着它去接客户。
- 把任何调用方放进来之前，**必须先按原仓文档配好 API 鉴权与受保护访问**（原仓 docs 里的 operate/api-authentication 一节）。
- 一句话：本地 quickstart 可以先跑起来看，但对外开放之前，这张卡没做完就别开门。

## 三条起步路，先选哪条

| 你的情况 | 走哪条 | 要点 |
|---|---|---|
| 没有 GPU 硬件，先评估 | 本地 quickstart（Local quickstart） | 在模拟机上跑起一个开发版管理平面和一个工作负载集群，**不需要 NVIDIA 账号** |
| 要上自己的基础设施 | 正式安装规划（Plan your installation） | 机器、网络、节点镜像和镜像仓库都得自己备齐，按原仓安装规划的顺序来；机器供给走 NICo |
| 要改源码 / 参与开发 | 源码构建（Build from source） | 要 **Go 1.27.3**（go.work 里钉死的版本）和 **GNU Make 4 或更高**；`make build` 产出九个二进制，构建本身不会启动管理平面、也不会建集群 |

## 本地评估门槛卡

| 项 | 要求（原仓口径） |
|---|---|
| 主机 | Linux 主机或虚拟机，amd64 或 arm64 都行 |
| CPU / 内存 | 至少 6 个 CPU、12 GiB 空闲内存 |
| 磁盘 | 100 GiB 空闲空间 |
| 必备条件 | Docker、虚拟 TPM 支持，以及 quickstart 指南里列的工具 |
| macOS | 没有 Linux 机器时走原仓的本地站点指南，在 Colima 虚拟机里跑这套评估 |

跑通的标准：跟着 quickstart 走到建出工作负载集群，并确认集群的节点上报 `Ready`。

## 它不是什么卡

| 不是 | 是 |
|---|---|
| 不是一个 Kubernetes 发行版 | 是一座自托管的「集群工厂」 |
| 不是 Cluster API 的一种口味（它内部用到 Cluster API 等组件，但产品形态不是这个） | 先用一阵子的种子集群拉起管理集群，管理平面再按需造出并管住许多个隔离的工作负载集群 |
| 不是装在云厂商那边的托管服务 | 管理平面装在你自己的基础设施上，客户集群开在你的 GPU 机器上 |

两个概念别混：**管理集群**跑 NVKE 管理平面和基础设施控制器，由 `nvkectl` 借一个临时种子集群组成；**工作负载集群**才是跑客户应用的，它的控制平面、etcd、节点池都是独立的，由管理平面经 NICo 创建和运维。

## 诚实标注区（这一栏是这张速查的命，条条都是原仓自述）

- **预览版，官方不建议拿去跑生产**：原仓 README 顶部明示，API、配置与行为在版本之间都可能变。
- **唯一兼容的基础设施提供方是 NICo**（NVIDIA Infra Controller），预览期没有第二家。
- **管理集群不会自己动**：它不给自己升级、不自扩缩、不自替换节点；管理平面本身也没有升级流程，更没有备份/恢复流程。
- **FDE 节点镜像只供评估**：不提供静态数据保密（no data-at-rest confidentiality），别拿评估镜像去装真数据。
- **势头数据**：1,022 ★ / 119 forks（2026-11-09 原仓实测，证据 A），fork 率约 12%。体量是小的，但它出自 NVIDIA 官方组织、官方文档站已经同步上线（证据 B）。这类东西看的不是星数，是位置：GPU 裸金属上自建托管 K8s，正是 GPU 云厂商的核心活。

---

## English (summary)

NVKE — NVIDIA Kubernetes Engine (1,022 stars / 119 forks as of 2026-11-09, Apache-2.0, explicitly a preview) is NVIDIA's open-source management plane for running a managed Kubernetes service on your own GPU infrastructure. A platform team installs one management cluster and then creates isolated workload clusters on demand, each with its own control plane, etcd and node pools; customers use the standard Kubernetes API and kubectl. Nodes attest through SPIFFE/SPIRE using their TPM before receiving join credentials, and etcd encryption is available per cluster. Read the security card first: after a preview installation, API authentication is off and the API uses plain HTTP — configure API authentication and protected access (see the upstream operate docs) before admitting any callers. Three ways to start: the local quickstart evaluates NVKE on simulated machines with no GPU hardware and no NVIDIA account; a real installation follows the upstream planning guide on your own machines, networking, node images and registries, with NVIDIA Infra Controller (NICo) — the sole compatible infrastructure provider at preview — provisioning the machines; building from source needs Go 1.27.3 (pinned in go.work) and GNU Make 4+. The local quickstart needs a Linux host or VM (amd64 or arm64) with at least 6 CPUs, 12 GiB of free memory and 100 GiB of free disk, plus Docker and virtual TPM support; on macOS it runs inside a Colima VM. NVKE is not a Kubernetes distribution and not a Cluster API flavour: it is a self-hosting factory for clusters, where a throwaway seed cluster stands up the management cluster and the management plane then provisions workload clusters. Honest notes, all from the upstream README: it is a preview and not recommended for production; the management cluster does not upgrade, scale or replace itself, and there is no management-plane upgrade or backup/restore procedure; FDE node images are for evaluation and provide no data-at-rest confidentiality.

---

## Español (guía rápida — traducción de estudio, pendiente de revisión por un hablante nativo)

**Kubernetes gestionado en tus propias máquinas GPU. Pero mira primero la tarjeta de seguridad: en la vista previa, la autenticación de la API viene desactivada.**

NVKE, NVIDIA Kubernetes Engine (1.022 estrellas / 119 forks a 9 de noviembre de 2026, Apache-2.0, vista previa declarada por el propio repositorio), es el plano de gestión de código abierto de NVIDIA para ofrecer Kubernetes como servicio sobre tu propia infraestructura GPU. El equipo de plataforma instala un clúster de gestión y desde ahí crea, bajo demanda, clústeres de trabajo aislados, cada uno con su propio plano de control, su etcd y sus grupos de nodos; los clientes usan la API estándar de Kubernetes y `kubectl`. Los nodos demuestran su identidad con el TPM mediante SPIFFE/SPIRE antes de recibir credenciales, y el cifrado de etcd está disponible por clúster.

La tarjeta de seguridad va primero: tras instalar la vista previa, **la autenticación de la API está desactivada y la API viaja por HTTP plano**. Antes de admitir a cualquier cliente hay que configurar la autenticación y el acceso protegido siguiendo la documentación original. Tres caminos para empezar: el quickstart local, que evalúa NVKE en máquinas simuladas, sin hardware GPU y sin cuenta de NVIDIA; la instalación real sobre tu infraestructura, siguiendo la guía de planificación original, con NVIDIA Infra Controller (NICo) — el único proveedor de infraestructura compatible en la vista previa — aprovisionando las máquinas; y la compilación desde el código, que pide Go 1.27.3 (fijado en go.work) y GNU Make 4 o superior. El quickstart local necesita una máquina o VM Linux (amd64 o arm64) con al menos 6 CPU, 12 GiB de memoria libre y 100 GiB de disco libre, además de Docker y soporte de TPM virtual; en macOS se ejecuta dentro de una VM Colima.

NVKE no es una distribución de Kubernetes ni un sabor de Cluster API: es una fábrica de clústeres autohospedada, donde un clúster semilla temporal levanta el clúster de gestión y el plano de gestión crea después los clústeres de trabajo. Notas honestas, todas del README original: es una vista previa y no se recomienda para producción; el clúster de gestión no se actualiza, no escala ni se reemplaza a sí mismo, y no existe procedimiento de actualización ni de copia de seguridad del plano de gestión; las imágenes de nodo FDE son solo para evaluación y no ofrecen confidencialidad de los datos en reposo.

---

## Português (guia rápido — tradução de estudo, aguardando revisão de um falante nativo)

**Kubernetes gerenciado nas suas próprias máquinas GPU. Mas olhe primeiro o cartão de segurança: na prévia, a autenticação da API vem desligada.**

O NVKE, NVIDIA Kubernetes Engine (1.022 estrelas / 119 forks em 9 de novembro de 2026, Apache-2.0, prévia declarada pelo próprio repositório), é o plano de gerenciamento de código aberto da NVIDIA para oferecer Kubernetes como serviço sobre a sua própria infraestrutura de GPU. A equipe de plataforma instala um cluster de gerenciamento e, a partir dele, cria sob demanda clusters de trabalho isolados, cada um com seu próprio plano de controle, seu etcd e seus conjuntos de nós; os clientes usam a API padrão do Kubernetes e o `kubectl`. Os nós provam a identidade com o TPM via SPIFFE/SPIRE antes de receber as credenciais, e a criptografia do etcd está disponível por cluster.

O cartão de segurança vem primeiro: depois de instalar a prévia, **a autenticação da API fica desligada e a API trafega em HTTP puro**. Antes de admitir qualquer cliente, é preciso configurar a autenticação e o acesso protegido seguindo a documentação original. Três caminhos para começar: o quickstart local, que avalia o NVKE em máquinas simuladas, sem hardware de GPU e sem conta da NVIDIA; a instalação real na sua infraestrutura, seguindo o guia de planejamento original, com o NVIDIA Infra Controller (NICo) — o único provedor de infraestrutura compatível na prévia — provisionando as máquinas; e a compilação a partir do código, que pede Go 1.27.3 (fixado no go.work) e GNU Make 4 ou superior. O quickstart local precisa de uma máquina ou VM Linux (amd64 ou arm64) com pelo menos 6 CPUs, 12 GiB de memória livre e 100 GiB de disco livre, além de Docker e suporte a TPM virtual; no macOS ele roda dentro de uma VM Colima.

O NVKE não é uma distribuição de Kubernetes nem um sabor do Cluster API: é uma fábrica de clusters auto-hospedada, em que um cluster semente temporário levanta o cluster de gerenciamento e o plano de gerenciamento cria depois os clusters de trabalho. Notas honestas, todas do README original: é uma prévia e não é recomendado para produção; o cluster de gerenciamento não se atualiza, não escala nem se substitui sozinho, e não existe procedimento de atualização nem de backup/restauração do plano de gerenciamento; as imagens de nó FDE servem só para avaliação e não oferecem confidencialidade dos dados em repouso.

---

## 来源与声明

- 灵感来源 / Source / Fuente / Fonte: [dsx-ai-factory/nvke](https://github.com/dsx-ai-factory/nvke)（文档站已同步上线；安全默认值、评估门槛与限制均以 2026-11-09 的上游 README 为准，星数以同日 GitHub 实测为准）
- 本仓是个人学习整理：内容用我自己的话总结自上游公开文档，不搬运原文；上游是预览版、迭代快，动手前请回上游文档核对当时的版本口径。
- 西语、葡语部分为学习译稿，待母语校对。Las secciones en español y portugués son traducciones de estudio pendientes de revisión nativa.
- License: MIT © 2026 Guangyi Zhao（注：本速查仓自身的许可；上游 NVKE 本体为 Apache-2.0，与本仓无关）
