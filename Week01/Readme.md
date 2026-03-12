# W01｜虛擬化概論、環境建置與 Snapshot 機制

## 環境資訊
- Host OS：Windows 11 專業版
- VM 名稱：Ubuntu 64-bit
- Ubuntu 版本：Ubuntu 24.04.4 LTS Release:24.0 Codename:noble
- Docker 版本：29.3.0, build 5927d80
- Docker Compose 版本：v5.1.0

## VM 資源配置驗證

| 項目 | VMware 設定值 | VM 內命令 | VM 內輸出 |
|---|---|---|---|
| CPU | 2 vCPU | `lscpu \| grep "^CPU(s)"` | lscpu \| grep "^CPU(s)" |
| 記憶體 | 4 GB | `free -h \| grep Mem` | free [options] (詳見圖)|
| 磁碟 | 40 GB | `df -h /` | （填入） |
| Hypervisor | VMware | `lscpu \| grep Hypervisor` | lscpu: bad usage |

![MEm](MEm.png)

## 四層驗收證據
- [ ] ① Repository：`deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg]   https://download.docker.com/linux/ubuntu   noble stable`
- [ ] ② Engine：
```
ii  docker-ce                                      5:29.3.0-1~ubuntu.24.04~noble            amd64        Docker: the open-source application container engine
ii  docker-ce-cli                                  5:29.3.0-1~ubuntu.24.04~noble            amd64        Docker CLI: the open-source application container engine
ii  docker-ce-rootless-extras                      5:29.3.0-1~ubuntu.24.04~noble            amd64        Rootless support for Docker.
```
- [ ] ③ Daemon：`sudo systemctl status docker` 顯示 active
![Daemon](Daemon.png)
- [ ] ④ 端到端：`sudo docker run hello-world` 成功輸出
![hello](helloworld.png)
- [ ] Compose：`Docker Compose version v5.1.0`

## 容器操作紀錄
- [ ] nginx：
```
howard@howard-VMware-Virtual-Platform:~$ sudo docker run -d -p 8080:80 nginx
5e1bb49346dc952fec4cf7cbee53925219fac37783008ac8da2bc05633b6a753
howard@howard-VMware-Virtual-Platform:~$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```
- [ ] alpine：`sudo docker run -it --rm alpine /bin/sh` 內部命令與輸出

| 指令 | 輸出 |
| --- | --- |
| `hostname` | `07ff009d7dd4` |
| `whoami` | `root` |
| `cat /etc/os-release` | `NAME="Alpine Linux" ID=alpine VERSION_ID=3.23.3 PRETTY_NAME="Alpine Linux v3.23" HOME_URL="https://alpinelinux.org/" BUG_REPORT_URL="https://gitlab.alpinelinux.org/alpine/aports/-/issues"` |
| `ls /` | `bin dev etc home lib media mnt opt proc root run sbin srv sys tmp usr var` |

- [ ] 映像列表：`sudo docker images` 輸出

## Snapshot 清單

| 名稱 | 建立時機 | 用途說明 | 建立前驗證 |
|---|---|---|---|
| clean-baseline | （時間點） | （此節點代表的狀態） | （列出建點前做了哪些驗證） |
| docker-ready | （時間點） | （此節點代表的狀態） | （列出建點前做了哪些驗證） |

## 故障演練三階段對照

| 項目 | 故障前（基線） | 故障中（注入後） | 回復後 |
|---|---|---|---|
| docker.list 存在 | 是 | 否 | （填入） |
| apt-cache policy 有候選版本 | 是 | 否 | （填入） |
| docker 重裝可行 | 是 | 否 | （填入） |
| hello-world 成功 | 是 | N/A | （填入） |
| nginx curl 成功 | 是 | N/A | （填入） |

## 手動修復 vs Snapshot 回復

| 面向 | 手動修復 | Snapshot 回復 |
|---|---|---|
| 所需時間 | （你的實測） | （你的實測） |
| 適用情境 | （你的判斷） | （你的判斷） |
| 風險 | （你的判斷） | （你的判斷） |

## Snapshot 保留策略
- 新增條件：
- 保留上限：
- 刪除條件：

## 最小可重現命令鏈
（列出讓他人能重現故障注入與回復驗證的命令序列）

## 排錯紀錄
- 症狀：
- 診斷：（你首先查了什麼？）
- 修正：（做了什麼改動？）
- 驗證：（如何確認修正有效？）

## 設計決策
（說明本週至少 1 個技術選擇與取捨）