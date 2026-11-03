# ReactionTester2

\# 反應測試儀 (NUCLEO-G474RE 期中專題)

透過 STM32G474RE 控制紅黃綠三色 LED 燈與外部按鈕，精確記錄使用者在綠燈亮起時的反應時間（微秒級別），並將數據即時傳輸至 Python Jupyter Notebook 進行接收與顯示。



\## 1. 情境故事：誰、在哪、遇到什麼麻煩

\* \*\*對象與場合\*\*：駕駛員、運動員或實驗參加者，在需要精確測試反射神經與反應速度的場合。

\* \*\*面臨問題\*\*：傳統的人工開關或簡易測試器無法達到微秒級別（μs）的精確計時，且容易被使用者預測節奏進行「預判偷跑」，導致測試成績不公正。



\## 2. 替身對照：故事裡的模組 → 板上替身

\* \*\*預備/警告指示燈\*\* → 板外另接紅 LED 燈（PA5）與黃 LED 燈（PA6）

\* \*\*反應訊號燈\*\* → 板外另接綠 LED 燈（PA7）

\* \*\*測試者反應觸發器\*\* → 外部另接按鈕 Button (PC13, EXTI 中斷, Pull-up, Falling-edge)

\* \*\*上位機監控與紀錄\*\* → 電腦端 Python Jupyter Notebook (PySerial / UART2)



\## 3. 功能與驗收條件：動作、預期、實測結果

\* \*\*動作 1（流程啟動）\*\*：程式啟動時亮紅燈，1 秒後紅燈維持亮起並加亮黃燈。

&#x20; \* \*\*預期結果\*\*：燈號順序與時間間隔正確。

&#x20; \* \*\*實測結果\*\*：紅燈亮 1 秒後黃燈正常加入亮起。

\* \*\*動作 2（防偷跑機制）\*\*：黃燈亮起後，隨機延遲 1\~3 秒才會切換成綠燈（紅黃燈同時熄滅）。

&#x20; \* \*\*預期結果\*\*：無法透過預測固定節奏來按按鈕。若在紅燈或黃燈期間按下按鈕，立即判定偷跑並熄滅所有燈號，終止本次測試。

&#x20; \* \*\*實測結果\*\*：隨機延遲生效；若於紅/黃燈亮起時偷跑，Python 端成功收到 `FALSE\_START` 並終止測試。

\* \*\*動作 3（精確計時與按鍵反應）\*\*：綠燈亮起瞬間開始 TIM2 微秒計時，按下 PC13 外部按鈕時綠燈立刻熄滅，並傳送精確時間。

&#x20; \* \*\*預期結果\*\*：按下按鈕時精確計算綠燈亮起到按鈕按下的微秒數（μs），並透過 UART 傳至 Python。

&#x20; \* \*\*實測結果\*\*：Python 端成功收到微秒數並轉換為秒數與毫秒（ms）顯示。

\* \*\*動作 4（逾時機制）\*\*：綠燈亮起超過 3 秒未按下按鈕。

&#x20; \* \*\*預期結果\*\*：綠燈自動熄滅，終止本次測試。

&#x20; \* \*\*實測結果\*\*：超過 3 秒未動作，Python 端成功收到 `TIMEOUT` 訊息。



\## 4. 用到的技術：第 3 \~ 7 週勾選表

\- \[x] GPIO Output（控制 PA5/PA6/PA7 三色 LED 燈）

\- \[x] GPIO External Interrupt（PC13 EXTI 下降沿觸發按鈕中斷）

\- \[x] General Purpose Timer（TIM2 32-bit 定時器，Prescaler 設定達成微秒 $\\mu s$ 級計時）

\- \[x] USART / UART 通訊（USART2 115200 8N1 與 Python PySerial 通訊）

\- \[x] C 語言標準庫應用（`<stdlib.h>` 的 `rand()` 隨機延遲、`<stdio.h>` 的 `snprintf()` 格式化）

\- \[x] Python 串列埠通訊（PySerial 接收與資料處理）



\## 5. 腳位表：用到的每支腳與用途

| 腳位 | 標籤 / 功能 | 用途說明 |

| :--- | :--- | :--- |

| \*\*PA5\*\* | `GPIO\_Output` | 輸出高/低電位，控制外部紅色 LED 燈 |

| \*\*PA6\*\* | `GPIO\_Output` | 輸出高/低電位，控制外部黃色 LED 燈 |

| \*\*PA7\*\* | `GPIO\_Output` | 輸出高/低電位，控制外部綠色 LED 燈 |

| \*\*PC13\*\* | `GPIO\_EXTI13` | 外部按鈕中斷腳位（Pull-up、Falling-edge 觸發） |

| \*\*PA2\*\* | `USART2\_TX` | 連接板載 ST-LINK VCP，將資料傳送至電腦端 Python |

| \*\*PA3\*\* | `USART2\_RX` | 連接板載 ST-LINK VCP UART 接收腳位 |



\## 6. 建置與執行：開發環境、燒錄步驟、終端機設定、命令

\* \*\*開發環境\*\*：

&#x20; \* STM32CubeMX

&#x20; \* STM32CubeIDE (C Language)

&#x20; \* Python 3.x / Jupyter Notebook

\* \*\*硬體連接步驟\*\*：

&#x20; 1. 將紅、黃、綠 LED 正極（長腳）分別連接至 NUCLEO-G474RE 的 PA5、PA6、PA7，負極經限流電阻接地 (GND)。

&#x20; 2. 將外部按鈕的一端接至 PC13，另一端接地 (GND)。內部已啟用 Pull-up 電阻。

&#x20; 3. 使用 USB 電纜將 NUCLEO 板連接至電腦。

\* \*\*STM32 燒錄步驟\*\*：

&#x20; 1. 使用 STM32CubeIDE 開啟專案。

&#x20; 2. 點擊 \*\*Build\*\*（錘子圖示）編譯專案。

&#x20; 3. 點擊 \*\*Run\*\*（綠色播放按鈕）將程式碼燒錄進 NUCLEO-G474RE 板。

\* \*\*Python / Jupyter Notebook 設定與命令\*\*：

&#x20; 1. 安裝必要模組：在終端機或 Jupyter 中執行 `!pip install pyserial`。

&#x20; 2. 確認 NUCLEO 板子的 COM Port 號碼（例如 `COM3` 或 `/dev/ttyACM0`）。

&#x20; 3. 執行 Python 接收腳本，設定 Baud Rate 為 `115200`，開始進行反應時間測試。





\## 7. AI 使用說明：哪段請 AI、你怎樣查核

\* \*\*請 AI 協助部分\*\*：

&#x20; 1. 協助設計 TIM2 微秒（$\\mu s$）計時器的 Prescaler 與 Counter Period 數學計算。

&#x20; 2. 撰寫 `main.c` 中的狀態機架構（`STATE\_RED`, `STATE\_YELLOW`, `STATE\_GREEN`）與 EXTI 中斷服務函式（`HAL\_GPIO\_EXTI\_Callback`）。

&#x20; 3. 撰寫 Python PySerial 串列通訊接收邏輯與異常處理。

\* \*\*查核與驗證方式\*\*：

&#x20; 1. 查核 STM32G474RE 參考手冊（Reference Manual）確認 APB1 Timer Clock 為 170MHz，驗證 Prescaler 設為 `169` 可精確得到 1MHz (1us/tick)。

&#x20; 2. 實機燒錄程式，透過示波器/邏輯分析儀或 Python 端收到的時間資料進行比對，確認時間數值無誤且 EXTI 按鈕中斷能即時觸發。



\## 8. 已知限制：哪些用替身、換真貨要改哪裡

\* \*\*已知限制 1（按鈕彈跳問題）\*\*：目前使用純軟體中斷讀取按鈕，若外部機械按鈕品質較差，可能產生彈跳（Debounce）引發二次中斷。

&#x20; \* \*換真貨修改方式\*：硬體上可在按鈕兩端並聯 0.1μF 電容進行濾波，或在 `HAL\_GPIO\_EXTI\_Callback` 中加入時間門檻進行軟體防彈跳。

\* \*\*已知限制 2（UART 傳輸格式）\*\*：目前採整數微秒字串傳輸，若未啟用浮點數列印，需由上位機（Python）進行 `us / 10^6` 之秒數轉換。

&#x20; \* \*換真貨修改方式\*：若需在 STM32 端直接輸出浮點數字串，需在 STM32CubeIDE 專案屬性中啟用 `-u \_printf\_float` 連結旗標。



\---



\## 9. 程式結構：哪個檔做什麼、ISR 與主迴圈分工



\### (1) 解說與分工架構

\* \*\*哪個檔做什麼\*\*：

&#x20; \* `Core/Src/main.c`：包含全域變數、主狀態機控制迴圈、TIM2/USART2/GPIO 初始化以及中斷服務 Callback 函式。

&#x20; \* `Core/Inc/main.h`：定義專案相關標頭檔與腳位巨集定義。

&#x20; \* `Reaction\_Tester.ipynb`：Python 端程式，負責開啟串口、持續監聽來自 STM32 的字串、將微秒轉換為秒與毫秒並印出可讀性高的測試結果。

\* \*\*ISR 與主迴圈分工 (Interrupt vs Main Loop)\*\*：

&#x20; \* \*\*ISR（中斷服務程式 - `HAL\_GPIO\_EXTI\_Callback`）\*\*：

&#x20;   \* 專責處理\*\*高即時性\*\*事件。當 PC13 按鈕按下（下降沿觸發）時，立即在第一個 CPU 週期擷取當前 `TIM2` 的計數器數值（`\_\_HAL\_TIM\_GET\_COUNTER`），並將 `button\_pressed` 旗標設為 1。

&#x20;   \* 不在 ISR 內部執行耗時的 `printf` 或延遲，確保反應時間量測精確度達到微秒等級。

&#x20; \* \*\*主迴圈（Main Loop - `while(1)`）\*\*：

&#x20;   \* 專責處理\*\*流程邏輯與狀態轉換\*\*。控制紅燈 -> 黃燈（含 `rand()` 隨機延遲） -> 綠燈的狀態切換。

&#x20;   \* 輪詢監聽 ISR 設定的 `button\_pressed` 旗標，判斷是否偷跑或正常完成反應，並負責透過 UART 發送字串至 Python 端。



\### (2) 專案檔案結構說明

```text

ReactionTester2/

├── Core/

│   ├── Inc/

│   │   └── main.h             # 專案標頭檔與腳位定義

│   └── Src/

│       ├── main.c             # 主程式碼（包含狀態機、TIM2/UART/GPIO設定與 EXTI ISR）

│       ├── stm32g4xx\_hal\_msp.c# HAL 庫底層外設初始化

│       └── stm32g4xx\_it.c     # 中斷向量處理常式 (包含 EXTI15\_10\_IRQHandler)

├── Picturefortoday/

│   ├── Fullshot.jpg           # 系統全貌與擺設實作照片

│   ├── Line-setup.jpg         # 硬體電路與線路細節照片

│   ├── MCUhighlight.jpg       # STM32 開發板特寫照片

│   ├── State\_Only\_Red.jpg     # 紅燈狀態照片

│   ├── State\_Red\_and\_Yellow.jpg# 紅黃燈狀態照片

│   ├── State\_Green.jpg        # 綠燈狀態照片

│   └── State\_All\_Turnoff.jpg  # 燈全暗狀態照片

├── Reaction\_Tester.ioc        # STM32CubeMX 設定檔

├── Reaction\_Tester.ipynb      # Python Jupyter Notebook 串列接收程式

├── .gitignore                 # Git 忽略檔案設定檔 (含 C/STM32/Python 暫存檔過濾)

└── README.md                  # 專案說明文件

