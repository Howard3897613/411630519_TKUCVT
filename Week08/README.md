# W08｜容器生產實踐

## Healthcheck 故障測試
- 停 db 後幾秒被標 unhealthy：`30.6`
- 對應的 log 訊息：
```
[+] stop 1/1
 ✔ Container w08-db-1 Stopped                                                                                                                                                      0.0s
NAME        IMAGE     COMMAND           SERVICE   CREATED          STATUS                     PORTS
w08-app-1   w08-app   "python app.py"   app       16 minutes ago   Up 3 minutes (unhealthy)   
unhealthy
app-1  | 127.0.0.1 - - [10/Jun/2026 04:19:54] "GET /healthz HTTP/1.1" 503 -
app-1  | 127.0.0.1 - - [10/Jun/2026 04:19:59] "GET /healthz HTTP/1.1" 503 -
app-1  | 127.0.0.1 - - [10/Jun/2026 04:20:04] "GET /healthz HTTP/1.1" 503 -
app-1  | 127.0.0.1 - - [10/Jun/2026 04:20:09] "GET /healthz HTTP/1.1" 503 -
app-1  | 127.0.0.1 - - [10/Jun/2026 04:20:14] "GET /healthz HTTP/1.1" 503 -
```


## Log 失控估算
- noisy 容器 30s log 大小：`3933937176`
- 預估 24h 大小：`10551.6 GB`
- 套 rotation 後穩定上限：`5.3M`

## 資源限制實驗
| 實驗 | 命令 | 觀察結果 | 對應 cgroup 檔 | 值 |
|---|---|---|---|---|
| OOM | `python -c "x = bytearray(256 * 1024 * 1024)"` | exit code = 137 | memory.max | 134217728 |
| CPU throttle | `stress-ng --cpu 4 --timeout 30s` | docker stats CPU% ≈ 50% | cpu.max | 50000 100000 |

## 權限四階對照
| 階梯 | id | CapEff | NoNewPrivs | curl /healthz |
|---|---|---|---|---|
| 0 (預設) | uid=0(root) | `00000000a80425fb` | 0 | 200 |
| 1 (非 root) | uid=1000 | `0000000000000000` | 0 | 200 |
| 2 (唯讀 rootfs) | uid=1000 | `0000000000000000` | 0 | 200 (需搭配 tmpfs) |
| 3 (cap_drop) | uid=1000 | `0000000000000000` | 0 | 200 |
| 4 (no-new-priv) | uid=1000 | `0000000000000000` | 1 | 200 |

## 排錯紀錄
- **症狀**：在執行 Part B 的 `noisy` 測試容器時，虛擬機的硬碟空間突然飆升至 100% 爆滿，導致系統卡頓，後續指令無法正常執行。
- **診斷**：Docker 預設的 `json-file` logging driver 並沒有大小限制。`noisy` 容器內部的程序使用 `yes` 指令無節制地狂吐日誌，導致在極短時間內產生了數 GB 的 `*-json.log` 檔案，徹底吃光了 Host 虛擬機 `/var/lib/docker/containers/` 底下的所有儲存空間。
- **修正**：
  1. 用快照強制修復(然後重弄作業)，快死掉了

## 設計決策
**你選的 mem_limit / cpus 數值理由是什麼？**
設定值必須基於實際觀察。通常會先透過 `docker stats` 觀察服務在正常負載下運作數天的峰值，然後取該峰值的 1.2 到 1.5 倍作為上限。如果抓太緊（例如直接貼齊平均值），突發的流量很容易觸發 OOM 被系統直接砍死；若抓太鬆則失去保護 Host 避免資源被單一壞死容器吃光的意義。

**read_only 之後你補了哪些 tmpfs，為什麼？**
補了 `/tmp` 與 `/home/appuser/.cache`。因為即使程式本身的邏輯沒有主動寫檔，許多語言框架（如 Python 的 `tempfile` 模組或套件快取）仍會預設向這些系統暫存目錄寫入資料。加上 `tmpfs` 可以在保持系統核心目錄唯讀以防止惡意程式植入的同時，維持應用程式正常的暫存運作需求。