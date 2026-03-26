### VM vs Container 對照

VM 和 Container 都提供「隔離的執行環境」，但隔離的層級完全不同：

- **VM 虛擬的是「硬體」：** 每台 VM 包含自己的 Guest OS（完整核心 + 系統工具），隔離很完整，但啟動慢、資源佔用大。
- **Container 虛擬的是「OS 資源」：** 所有容器共用 Host 的 kernel，只隔離程序看到的檔案系統、網路、PID 等，啟動快、資源省，但隔離邊界取決於 kernel。

| 維度     | 虛擬機器（VM）                       | 容器（Container）                       |
| -------- | ------------------------------------ | --------------------------------------- |
| 隔離層   | 完整 Guest OS + Hypervisor           | 共用 Host OS kernel                     |
| 啟動速度 | 數分鐘（要開整個 OS）                | 數秒（只啟動程序）                      |
| 資源佔用 | 重（每台需獨立 OS，通常 1+ GB）      | 輕（單機可跑數百個，MB 等級）           |
| 封裝內容 | 完整 OS + 應用程式 + 設定            | 應用程式 + 函式庫 + 相依項              |
| 映像大小 | 數 GB                                | 數十 MB ~ 數百 MB                       |
| 核心技術 | Hypervisor（VMware / KVM / Hyper-V） | Container Engine（Docker / containerd） |
| 回復方式 | Snapshot 還原                        | 重新拉取映像 / 重新部署                 |


### 本課選擇「VM 裡跑 Docker」的理由
利用VMware作為環境，統一底層作業系統為Ubuntu，消除不同 Host OS 之間的差異，而且出問題時修復容易。

VMWare：利用虛擬機可以統一所有人的底層作業系統（Ubuntu 24.04），消除個人電腦作業系統之間的差異。萬一環境被改壞，還能透過snapshot將整台虛擬機輕鬆回復。

Docker：在統一的 Ubuntu 底座上，透過 Docker 來交付可重現的應用程式環境。如果映像檔損壞，只需刪除並重新拉取，完全不會影響到基底。


### Hypervisor Type 1 vs Type 2 的差異與本課的選擇


Hypervisor 是負責建立和管理虛擬機的軟體，依照安裝位置分成兩類：

**Type 1（Bare-metal Hypervisor）**

- 直接安裝在實體硬體上，不需要先有作業系統。
- 範例：VMware ESXi、Microsoft Hyper-V Server、Xen。
- 特性：效能高、延遲低，VM 靠 Hypervisor 直接跟硬體溝通。
- 適用場景：企業資料中心、雲端基礎設施（AWS EC2 底層就是 Type 1）。

**Type 2（Hosted Hypervisor）**

- 安裝在現有作業系統之上，像一般應用程式一樣執行。
- 範例：VMware Workstation、VirtualBox、Parallels、VMware Fusion。
- 特性：方便安裝在個人電腦，但多一層 Host OS，效能略低。
- 適用場景：個人開發、教學、本機測試。

**本課選擇 Type 2（VMware Workstation）的理由：** 學生可以在自己的筆電上安裝，不需要專用伺服器硬體。教學環境只需要「一致」和「可回復」，不需要資料中心等級的效能。