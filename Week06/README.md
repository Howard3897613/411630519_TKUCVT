# W06｜Docker Image 與 Dockerfile

## 映像組成
- **Layers 是什麼**：一堆唯讀的檔案系統差異層（tarball），對應到 overlay2 的 lower dirs。不同 Image 若有相同的底層架構，這些 layer 可以共用以節省硬碟空間。
- **Config 是什麼**：一份 JSON 格式的中介資料（Metadata），裡面記錄了容器啟動時需要的資訊，例如執行命令 (CMD/ENTRYPOINT)、工作目錄 (WORKDIR)、環境變數 (ENV) 等。
- **Manifest 是什麼**：一份清單檔案，負責將上述的 Layers 與 Config 綁定在一起，並記錄每一層的摘要值 (digest) 與檔案大小。

## python:3.12-slim inspect 摘錄
- Config.Cmd：`["python3"]`
- Config.Env：`["PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin", "LANG=C.UTF-8", "GPG_KEY=7169605F62C751356D054A26A821E680E5FA6305"]`
- Config.WorkingDir：未指定（預設為 `/`）
- RootFS.Layers 數量：4 層

## Layer 快取實驗
*(請執行 Part B & C 步驟 8, 11, 13 並填入)*
| 情境 | build 時間 |
|---|---|
| v1 首次 build | 56.295s |
| v1 改 app.py 後 rebuild | 47.765s |
| v2 首次 build | 45.075s |
| v2 改 app.py 後 rebuild | 4.894s |

**觀察：為什麼 v2 的 rebuild 這麼快？**
因為 Dockerfile.v2 將變動頻率低的 `COPY requirements.txt` 與耗時的 `RUN pip install` 往上移。當我只修改 `app.py` 時，Docker 的快取鍵值 (cache key) 計算發現前面的步驟沒有改變，因此直接重用快取 (CACHED)。由於避開了重新下載與編譯套件的過程，rebuild 只需要重算最後幾層檔案複製與 metadata 的層級，速度自然從幾十秒縮短到幾秒內。

## CMD vs ENTRYPOINT 實驗
| 寫法 | `docker run <img>` 輸出 | `docker run <img> extra1 extra2` 輸出 |
|---|---|---|
| CMD shell form | `argv = ['show_args.py', 'default1', 'default2']` (PID = 7) | 報錯：`exec: "extra1": executable file not found in $PATH` |
| CMD exec form | `argv = ['show_args.py', 'default1', 'default2']` (PID = 1) | 報錯：`exec: "extra1": executable file not found in $PATH` |
| ENTRYPOINT + CMD | `argv = ['show_args.py', 'default1', 'default2']` (PID = 1) | `argv = ['show_args.py', 'extra1', 'extra2']` (PID = 1) |

**結論：**
`CMD` 的設計是提供「預設值」，因此當 `docker run` 後方帶有附加參數時，會將 Dockerfile 中的 `CMD` 整串無情覆蓋。而 `ENTRYPOINT` (Exec form) 則是固定了容器的主程式，`docker run` 的參數只會當作附加參數傳遞給它。這就是為什麼將應用程式主體放在 `ENTRYPOINT`，並將預設參數放在 `CMD`，是實務上最穩定且保有彈性的寫法。另外，Exec 寫法能確保應用程式直接成為 PID 1，正確接收系統信號 (如 SIGTERM)，這點也是 Shell 寫法做不到的。

## Multi-stage 大小對照
| Image | SIZE |
|---|---|
| python:3.12（builder base） | 428MB |
| python:3.12-slim（runtime base）| 45.4MB |
| myapp:v2（單階段） | 48.1MB |
| myapp:multi（多階段） | 44.8MB |

**解釋：builder stage 的 layer 去哪了？**
Multi-stage build 的核心在於最終產出的 Image 只會打包「最後一個 Stage」的內容。Builder stage 所產生的龐大編譯工具層與暫存檔，依然保留在本機的 Docker cache 中（使用 `docker images -a` 可以看到標籤為 `<none>` 的影像），但它們不會被包裹進最終的 `myapp:multi` Image 中，這使得最終映像檔大幅瘦身且更安全。

## .dockerignore 故障注入
| 項目 | 故障前 | 故障中 | 回復後 |
|---|---|---|---|
| du -sh . | 52K | 151M | 151M (垃圾檔仍在硬碟，但被 Docker 忽略) |
| build context 傳輸大小 | (極小) | 129B | 130B |
| build 時間 | - | 3.502s | 2.430s |

## 排錯紀錄
- 症狀：`docker.entrypoint執行出現錯誤`
- 診斷：`回推指令，確認是哪一部出錯`
- 修正：`重新nano一份`
- 驗證：`再度執行，確認執行成功`

## 設計決策
**為什麼 runtime stage 選擇 `python:3.12-slim` 而不是 `alpine`？**
對於 Python 專案，`alpine` 使用的是 musl libc，而多數預編譯的 Python 套件 (wheels) 都是基於 glibc 編譯的。若使用 alpine，經常需要手動安裝編譯器 (gcc, musl-dev) 並從原始碼重新編譯套件，這不僅耗時，還可能引發相容性問題並失去容量優勢。因此選擇 `slim` 是兼顧容量與 Python 生態系相容性的最佳決策。