# SRBMiner Monitor

繁體中文 Windows 桌面監控工具，以自己的 BAT 啟動 SRBMiner-Multi，集中查看算力、GPU 狀態、分享結果與礦工輸出，並管理停止、重啟及異常退出復原。

**目前版本：0.2.0 · Windows x64 · C# / .NET 10 / WPF**

[下載 Windows 可攜版](https://github.com/DAS-WIN/SRBMiner-Monitor/releases/download/v0.2.0/SRBMinerMonitor-0.2.0-win-x64.zip) · [下載原始碼](https://github.com/DAS-WIN/SRBMiner-Monitor/releases/download/v0.2.0/SRBMinerMonitor-0.2.0-source.zip) · [版本說明](docs/RELEASE-NOTES-0.2.0.md)

## 功能總覽

| 功能 | 說明 |
| --- | --- |
| BAT 啟動 | 選擇現有 BAT／CMD，或直接貼上 BAT 內容 |
| 硬體監控 | 總算力、GPU 功耗、效率、最高溫度，以及每卡風扇與時脈 |
| 即時分享 | Accepted／Rejected 計數與礦工原始成功訊息 |
| 趨勢圖 | 最近 15 分鐘、1 小時、6 小時的算力／功耗／溫度 |
| 背景執行 | 礦工輸出整合於應用程式，不開啟獨立 CMD 視窗 |
| 重啟管理 | 手動重啟、定時重啟及非預期退出自動重啟 |
| 事件紀錄 | 分享成功、失敗及逾時，可篩選與匯出 |
| 系統匣 | 關閉視窗後持續監控；提供明確的結束操作 |

## 系統需求

- Windows x64 桌面環境。
- 自行安裝 [SRBMiner-Multi](https://github.com/doktor83/SRBMiner-Multi/releases)，並準備可正常運行的挖礦 BAT。
- SRBMiner 本機 JSON API，預設連接埠為 `21550`。
- NVIDIA 硬體感測資訊需要驅動提供的 `nvidia-smi`。

可攜版已包含 .NET 執行環境，不需另裝 SDK。本專案不包含 SRBMiner 執行檔，目前支援單機、多 GPU、單一演算法。AMD／Intel 的額外感測器介接尚未提供；可用資料依礦工 API 而定。

## 快速開始

1. 下載 `SRBMinerMonitor-0.2.0-win-x64.zip`，將整個 ZIP 解壓縮至可寫入的資料夾。
2. 執行 `SRBMiner Monitor.exe`。
3. 開啟「挖礦設定」，選擇 BAT 輸入方式並完成設定。
4. 按「儲存並啟動 BAT」。
5. 回到總覽，確認 API 狀態及礦工輸出。

啟動監控程式不會自行開始挖礦。礦池、錢包、演算法與挖礦參數由 BAT 決定，程式不調整超頻、風扇或功耗限制。

### 使用現有 BAT

1. 選擇「選擇現有 BAT」，指定 `.bat` 或 `.cmd`。
2. 若原 BAT 尚未啟用 API，按「建立監控副本」。
3. 將副本存於原 BAT 相同資料夾，再用副本啟動。

監控副本保留原 BAT，僅補上 API 參數。BAT 的工作目錄為其所在資料夾，讓 `%~dp0` 與相對路徑正常運作。包含多個礦工指令或無法辨識的複合命令時，需自行設定 API 參數。

### 貼上 BAT 內容

1. 選擇「貼上 BAT 內容」。
2. 選擇 SRBMiner 所在資料夾。
3. 貼上完整 BAT，按「儲存並啟動 BAT」。

程式會補上必要的 `--api-enable` 與 `--api-port` 參數，並在指定資料夾建立暫存 BAT。停止後清理暫存檔；貼上的內容以 Windows DPAPI 加密儲存。

## 資料更新與統計

### 硬體更新間隔

在挖礦設定輸入 **1–10000**，單位可選 **秒／分／時／天**，預設 **1 分鐘**。

- 算力、功耗、效率、溫度、風扇、時脈與趨勢使用同一採樣間隔。
- 預設在每分鐘邊界更新；首次資料、新礦工階段及修改間隔時立即採樣。
- 儲存設定後立即生效，不必重啟礦工。

### 即時分享

- 礦工原始成功訊息到達時立即顯示於「最近成功分享」。
- Accepted／Rejected 數字每秒核對 API，獨立於硬體採樣間隔。
- API 與輸出各自維護累積計數，再比對同一階段的最大已知值，不將兩份數字相加。
- 重啟建立新統計階段；趨勢保留階段分界。
- 未提供的數據顯示「—」；API 連線逾時不視為分享逾時。

GPU 功耗為顯卡感測值，並非整機耗電。效率為算力除以 GPU 功耗；功耗未知或為零時不計算效率。

### 事件與原始輸出

事件僅包含分享成功、失敗與分享逾時，保留最近 2000 筆，可顯示及匯出目前篩選的最近 500 筆。總覽另提供最近 500 行礦工即時輸出，方便檢查初始化、連線或 BAT 錯誤。

礦池區塊顯示 BAT 可解析的位址與來源；API 已連線不等同礦池已接受分享。

## 停止、重啟與自動復原

| 操作 | 行為 |
| --- | --- |
| 停止礦工 | 停止受管理程序，取消待重啟動作；監控視窗保持開啟 |
| 重啟礦工 | 完全停止舊礦工後重新啟動；監控視窗保持開啟 |
| 定時自動重啟 | 運行指定時間 → 完全停止 → 等待指定分鐘 → 啟動 |
| 非預期退出自動重啟 | 獨立啟用，礦工意外退出後等待 10 秒再啟動 |
| 結束應用程式 | 停止受管理礦工，並退出監控程式 |

### 定時排程

在總覽按「自動重啟礦工」，設定運行時間、停止後等待分鐘及是否持續循環，再套用排程。首次倒數從套用時開始，後續循環從新礦工啟動後開始。

例如：**運行 6 小時 → 停止 → 等待 5 分鐘 → 啟動**。

等待時間從確認礦工完全退出後開始。單次模式在成功重啟後完成；循環模式重新計算運行時間。

### 非預期退出復原

在挖礦設定勾選「礦工非預期退出後自動重啟」，並按「儲存設定」：

- 單獨啟用：退出後等待 10 秒重啟，不需要開啟定時排程。
- 定時排程已啟用：退出後依排程的等待分鐘倒數。
- 兩者同時啟用：以排程等待時間為準，只啟動一個礦工。
- 自動啟動失敗：重新倒數後再試。
- 手動停止：取消任何待重啟動作。
- 關閉獨立復原選項並儲存：取消該選項的待執行倒數。

自動重啟沿用目前運行的 BAT 設定。只儲存而尚未套用的 BAT 修改，需手動啟動或重啟才會生效。程式僅控制自身啟動的礦工，外部礦工僅供監控。

## 設定、升級與退出

| 項目 | 位置／方式 |
| --- | --- |
| 本機設定 | `%LOCALAPPDATA%\SRBMiner Monitor\settings.json` |
| 啟動診斷 | `%LOCALAPPDATA%\SRBMiner Monitor\Logs`；無法寫入時使用暫存資料夾 |
| 關閉視窗 | 縮至系統匣，監控與排程持續運行 |
| 完整退出 | 選擇「結束應用程式」 |
| 升級 | 結束舊版，將新版完整解壓縮至新資料夾 |

貼上 BAT 的加密內容需由相同 Windows 帳戶解密，不能直接搬至另一帳戶使用。重新開啟 GUI 會載入設定，但不自行啟動挖礦或啟用排程。

## 常見問題

### 雙擊後沒有畫面

執行同資料夾的 `Start-Diagnostics.cmd`，查看啟動紀錄。確認已完整解壓整個套件，不能只複製 EXE。

### API 尚未連線

確認 BAT 的 SRBMiner 指令含 `--api-enable`，且 `--api-port` 與應用程式設定一致。再展開「礦工即時輸出」，檢查礦工是否仍在初始化、指令錯誤或已退出。

### Accepted 已變更，但最近成功分享沒有新文字

數字可由 API 更新；文字列表需要礦工實際送出成功訊息。若礦工暫時緩衝輸出，文字會在收到後顯示。

### 部分硬體欄位顯示「—」

代表資料未知或來源未提供。確認 NVIDIA 驅動及 `nvidia-smi` 可用；缺少或無法明確對應的感測值不會推測補入。

## 開發與建置

需要 **.NET 10 SDK**。WPF 執行與 Windows 整合測試需要 Windows；核心測試可獨立執行。

取得原始碼並進入專案資料夾：

```powershell
git clone https://github.com/DAS-WIN/SRBMiner-Monitor.git
cd SRBMiner-Monitor
```

在原始碼根目錄執行：

```powershell
dotnet build SrbMinerMonitor.slnx -c Release
dotnet test tests/SrbMinerMonitor.Tests -c Release
pwsh ./scripts/Smoke-Windows.ps1
pwsh ./scripts/Verify-ReleaseContent.ps1
pwsh ./scripts/Publish-Windows.ps1
pwsh ./scripts/Verify-PublishedStartup.ps1
```

可攜版輸出至 `artifacts/win-x64`，ZIP 輸出至 `artifacts`。

### 專案結構

```text
src/
  SrbMinerMonitor.Core/          程序管理、排程、API、輸出解析與資料模型
  SrbMinerMonitor.App/           WPF 介面、ViewModel、設定與診斷
tests/
  SrbMinerMonitor.Tests/         核心自動測試
  SrbMinerMonitor.FakeMiner/     合成 API／輸出測試程序
  SrbMinerMonitor.WindowsSmoke/ Windows 視窗與生命週期整合測試
scripts/                        建置、發布與驗證腳本
docs/                           架構、資料來源及驗收文件
```

測試使用合成資料與不挖礦的程序，不連接外部礦池。自動驗證涵蓋資料解析、分享去重、BAT 兩種來源、硬體採樣、程序擁有權、手動與定時重啟、異常退出復原及發布後 EXE 啟動；實際硬體與礦池相容性仍取決於驅動和礦工版本。

## 延伸文件

- [程式架構](docs/IMPLEMENTATION.md)
- [資料來源與欄位](docs/DATA-SOURCES.md)
- [自動驗證](docs/VERIFICATION.md)
- [Windows 驗收](docs/WINDOWS-VALIDATION.md)
- [0.2.0 版本說明](docs/RELEASE-NOTES-0.2.0.md)

## 持續整合與發布

GitHub Actions 在推送與 Pull Request 時執行核心測試、Windows 建置、視窗整合驗證及封裝檢查。`main` 通過驗證後會發布尚未存在的專案版本；已發布的版本保留原有附件。發布新版前，請更新應用程式專案的 `Version`，並新增對應的 `docs/RELEASE-NOTES-版本號.md`。
