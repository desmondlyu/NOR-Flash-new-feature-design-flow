---
name: writer_program_flow_design
version: 20261110_02
description: 把 /nor_flash_dv_design 產出的測試計畫書（.html 或 .md 的測項表）轉成 Writer／Programmer 韌體 C code（例如 LEAPER-56 的 LP_ICIDCheck.c）的自動測試流程。範圍包含流程骨架、指令序列、判定引擎、停止規則、測試前說明與測試中 printf log 文字、結尾總表、編譯與實測回填。只要使用者提到「把測試流程寫成 C code」「Writer 測試程式」「Auto Test 選單」「printf log 看不懂」「log 格式」「測試總表」「實測 log FAIL 要查」「計畫書轉程式」，或在 /nor_flash_dv_design 之後要實作，就用這個 skill，即使使用者沒有說出 skill 名稱。
---

# Writer Program Flow Design

本 skill 是 `/nor_flash_dv_design` 的下一棒：

| 階段 | Skill | 產出 |
|---|---|---|
| 計畫 | `/nor_flash_dv_design` | 計畫書（.html／.md）：狀態圖、規則 `R*/F*/U*`、測項表、判定代碼 |
| 實作 | `writer_program_flow_design`（本 skill） | Writer C code、log 格式、結尾總表、change_log、參考 log |
| 回填 | 本 skill 第 8 步，再回到 `/nor_flash_dv_design` 第 5 步 | 實測判讀、計畫書與 C code 同步 |

計畫書是依據來源。C code 跟隨計畫書。兩者不一致時，先停下來，問使用者以哪一份為準。

## 沿用計畫的 Writer 測試範圍

以計畫書選定的代表性應用流程實作，不自行展開所有組合。固定沿用符合指令規格的低頻設定，不掃頻或驗證 SPEED；保留完成等待、狀態輪詢、timeout 與 Reset recovery time。一般初始化／復原採 Datasheet 支援的 Software Reset，不預設 Power OFF/ON、/HOLD、硬體 Reset 腳位或特殊電源控制，也不要求使用者先提供這些能力。

Software Reset 不能替代真正的上下電驗證。無法執行 Reset 或復原時停止，不自行加入電源操作。需上下電才能驗證的功能記錄為範圍外或未涵蓋；不以 Reset 結果宣稱通過。只有使用者明確擴充範圍才增加特殊控制。

接收 HTML 的條件、步驟與判讀方式即可，不要求計畫預先具備固定 R*/F*/U* 編號或程式判定代碼。所需程式對應由本 skill 建立，保留原測項意義與範圍。

## 寫作規則（兩種讀者，兩套規則）

| 文字 | 讀者 | 規則 |
|---|---|---|
| printf log（英文） | 產線與 DV 工程師，看得懂 Datasheet，不一定懂測試程式的簡稱 | `asd-ste100` Strict 模式 |
| 計畫書、說明文件、回覆（繁中） | 使用者、主管 | `speak-human-tw`（辦公文書情境）＋ `asd-ste100` 結構規則 |

1. 動筆前載入 `asd-ste100` 與 `speak-human-tw`。載入失敗時，在回覆開頭標「未套用」，不可假裝已套用。
2. 修飾對象是本 skill 自己產生的文字。不執行 `speak-human-tw` 的兩輪確認流程。
3. log 用詞以 Datasheet 的名稱為準（例如 `Write Enable (06h)`、`Read Status Register-1 (05h)`），第一次出現時寫全名加 opcode。
4. log 不寫 Datasheet 章節號或頁碼。不同料號的 Datasheet 章節不同。
5. 不在 log 用測試程式內部的簡稱（例如 `Bottom`、`A-2r`、`WP-BASELINE`）。改用位址或 Datasheet 用詞。
6. 保護清單：opcode、暫存器與位元名稱、位址、數值、測項編號、判定代碼。改寫時不可改動。

## 0. 收集輸入

一次問完缺少的項目。使用者已給的不再問。

- 計畫書路徑（`/nor_flash_dv_design` 產出）與版本。
- 目標 C 檔路徑、選單入口、要新增或改寫的函式範圍。
- DUT 資訊：JEDEC ID、容量、位址模式（3-byte／4-byte，見 skill `4-Byte-Addressing-Mode`）。
- 可用的底層函式（SPI 傳送、Read Status、延遲／timeout、Software Reset 所需指令傳送）。用 `leaper56-spi-test-sequence-generator` 的對應規則查找，不自己發明底層函式。
- 刻意保留的規則（固定等待時間、固定 pattern、固定位址）。

完成條件：計畫書每個測項都能對到一段待寫的 C code，或標「NOT RUN（原因）」。

## 1. 盤點韌體環境（動手前必做）

詳細清單見 `references/firmware_checklist.md`。至少檢查：

1. 檔案編碼與換行（例如 CP950＋CRLF）。一律用 Python 以二進位讀寫，不用會改編碼的編輯方式。
2. printf 限制（例如 at91lib：不支援 `%-`、單次輸出 ≥100 字元會當機）。長文字用專用輸出函式分行。
3. 舊編譯器可能因原始檔過大而失敗。先確認實際檔案大小限制，再控制 C code 大小。
4. 編譯指令。先對原檔編譯一次，記下既有 warning，之後只看新增的 warning。
5. 備份：`backup\<檔名>_backup_YYYYMMDD_HHMM`。
6. change_log 的編碼與語言（ASCII＋CRLF＋英文，避免亂碼）。

## 2. 流程骨架

每個 Auto Test 都用同一個骨架：

| Flow | 用途 | 規則 |
|---|---|---|
| Flow 0 | 前置檢查與測試資料 | 讀 ID、讀殘留保護與非揮發性設定、建立測試資料。失敗判 NOT READY |
| Flow 1～N | 計畫書的正驗、反驗、邊界 | 依計畫書順序。每個 Flow 開頭檢查前置狀態 |
| Flow NX | 邊界用法與 Datasheet 未定義用法 | 放在最後。`[Spec]` 判正式 PASS/FAIL，`[No spec]` 不符時判 ASK DESIGN |
| 最後一個 Flow | 復原 | 僅執行目前狀態下規格允許且設備可行的復原；無法復原時停止，請工程師處理 |

1. 步驟編號用數字：`2.4` 表示 Flow 2 第 4 步。子項用字母（`4.5a`～`4.5f`）。Detail 用 `r` 結尾（`1.2r`），不算一步。
2. 編號依實際執行順序排列。移動步驟時，重新編號，並同步計畫書。
3. 每個 Flow 記下 plan 數（應跑步驟數），供結尾總表計算 `Done = 已跑/應跑`。
4. 會永久改變樣品的步驟（OTP、Lock bit），前面加一道閘門：前置步驟全部 PASS 才執行，否則後面判 NOT READY，跳到復原。

## 3. 指令序列規則

1. 每個寫入類指令前送 `Write Enable (06h)`。
2. 保護類指令（寫暫存器、密碼、IBP 等）：`06h` 後讀 `05h`，WEL=1 才送下一個指令。WEL=0 判 RETEST。
3. Erase／Program：`06h` 後直接送指令，不插入其他指令，除非計畫書要測的就是插入行為。曾經實測：`06h→05h→D8h` 時 `D8h` 不執行。
4. 用讀回資料判 PASS/FAIL。BUSY 只作記錄。沒讀到 BUSY 時，仍要至少輪詢到規定時間（例如 100ms）才讀回。
5. 時序敏感的窗口（讀到 BUSY=1 到送出下一個指令之間）不可有 printf。UART 輸出會讓操作先結束。
6. 中斷類測項：送中斷前讀到 BUSY=1，送完後 BUSY 仍為 1，才算有效。否則判 RETEST。
7. 先做正向對照（無保護時 Erase／Program 成功），再做反驗。正向對照失敗時，反驗的 PASS 不可信。
8. 前置步驟不用待測指令（例如待測 `D8h` 時，建立資料改用 `20h`）。
9. 正向對照失敗時，加一個診斷子程序（Detail），依序縮小原因，例如：同指令重送、換鄰近區、移除保護後重送。診斷結果符合已知的「待確認」行為時判 INFO，否則判 FAIL。

## 4. 判定引擎

計畫書代碼對到 log 顯示名稱：

| 計畫書代碼 | log 顯示 | 算進 FINAL RESULT=FAIL |
|---|---|---|
| PASS | PASS | 否 |
| FAIL | FAIL | 是 |
| SETUP_FAIL | NOT READY | 是 |
| BLOCKED | NOT READY | 是 |
| INVALID | RETEST | 是 |
| RECORD | INFO | 否（待確認） |
| CHECK | ASK DESIGN | 否（待確認） |
| NOT_RUN | NOT RUN | 否 |

停止規則：

1. Flow 0～N 遇到 FAIL、NOT READY 或 RETEST：印停止說明，後面步驟不執行，跳到復原 Flow。
2. Flow NX 遇到 FAIL 或 ASK DESIGN：繼續執行。
3. 復原 Flow 第一步失敗：直接停止。
4. INFO 與 ASK DESIGN 是待確認，不算進正式 PASS/FAIL，列在 Follow-up。

實作上用一個 `judge(id, code, note)` 統一計數與輸出，再用 `pf(id, ok, note)` 包裝 PASS/FAIL。不要在各步驟自己 printf 結果。不要重複輸出相同資訊。

## 5. log 文字

格式與範例見 `references/log_format.md`。骨架如下：

```
WRITE PROTECTION AUTO TEST              <- 標題、容量、SPI clock
Instructions / Terms / Test areas       <- 只列必要的 Datasheet 名稱與 opcode
WARNING ...                             <- 只列會永久改變樣品的步驟
Start the test? (y/n)

TEST FLOW 1: ...
[1.2] Erase with WPS=0
      Do     : Sector Erase (20h) 000000h ...
      Expect : the 3 targets = FFh, neighbors do not change
      Got    : 070000h is not erased (first byte 5Ah, expected FFh)
      Detail : try 1: Block Erase 070000h again -> no BUSY, not erased
      Result : INFO (no Datasheet rule: ...)
      Note   : the next steps do not run. The test goes to Flow 6.

TEST SUMMARY / Restore status / Total / FINAL RESULT / Follow-up
```

英文規則（asd-ste100 Strict）：

- 一行一個意思。指令句 ≤20 字，說明句 ≤25 字。
- 主動語態、現在式。不用分號。不用片語動詞。
- 同一件事只用一個詞（例如固定用 "test data"，不混用 baseline/pattern）。
- `Expect` 寫 Datasheet 給的結果，不寫程式條件式。
- 不產生固定的 `HOW TO READ THIS LOG` 區塊。
- 只輸出判定所需的 `Do`、`Expect`、`Got`、`Result` 與必要的 `Detail`。
- 不重複列出已在測項或總表出現的資訊。
- 若編譯器有檔案大小限制，優先縮短固定文字與移除重複說明，不刪除判定所需的位址、數值、opcode 或讀回值。
- 寫完後用 `asd-ste100/scripts/ste-lint.py` 檢查 legend 與說明句。

## 6. 結尾總表

依序輸出：

1. `TEST SUMMARY`：每個 Flow 一列，欄位 `Done PASS FAIL INFO ASK NOT-RUN`。FAIL 欄含 FAIL、NOT READY、RETEST。
2. `Restore status:` 一句話說明樣品是否回到預設。
3. `Total n: PASS, FAIL, NOT READY, RETEST, NOT RUN, INFO, ASK DESIGN`。
4. `FINAL RESULT = PASS/FAIL`。
5. `Follow-up:` 列出要人看的步驟編號（INFO、ASK DESIGN、NOT READY、NOT RUN）。

## 7. 實作與驗證

1. 用 Python 產生 `.new` 檔，不直接覆蓋原檔。
2. 編譯 `.new`：0 error，且沒有新增 warning。
3. 機械檢查（缺一不可）：
   - 編碼可正確解碼，沒有單獨的 LF。
   - 沒有超過 printf 上限的字串，沒有不支援的格式字元。
   - 檢查 `.new` 原始檔大小沒有超過目標編譯器限制。
   - 檢查固定 log 文字沒有重複輸出。
   - 沒有殘留舊編號、舊代碼或舊標籤（用 grep 列出 0 筆）。
   - 每個計畫書測項在 C code 都有 begin 與 judge（列出清單比對）。
   - plan 數 = 實際 begin 數。
4. 通過後才用 `.new` 覆蓋原檔。
5. 產出一份參考 log（依程式輸出順序手工排出預期結果），給使用者看格式。
6. change_log 追加英文 entry：改了什麼、編號與代碼變更、編譯結果、是否已上 DUT、備份路徑。
7. 回覆時明確寫「已編譯，未上 DUT 實測」或「已實測」。

## 8. 實測回填與除錯

使用者貼實測 log 時：

1. 先回答「現在是什麼問題」：哪一步、預期什麼、讀到什麼。不先改程式。
2. 區分三種原因，各寫證據：
   - 樣品行為（例如 Datasheet 沒寫的阻擋）→ INFO／ASK DESIGN，回報設計端。
   - 測試程式（指令序列、時序窗口、printf 太長、競態）→ 修程式。
   - 治具條件（電源、時脈）→ 標治具問題。
3. 舊流程沒有 FAIL、新流程 FAIL 時，先比對兩者指令序列的差異，再判斷。
4. 不可為了讓結果變 PASS 而放寬判定。改判定代碼時，寫出依據，並同步計畫書的「待確認」清單。
5. 修正後回到第 7 步驗證，並回 `/nor_flash_dv_design` 第 5 步更新計畫書。

## 9. 文件同步

1. 計畫書、`*.md` 流程文件、C code 的步驟編號、指令、預期值、判定代碼必須一致。
2. 有多份相同 md 時，修改後比對 hash 相同。
3. 說明文件（.html）只改相關章節，改完檢查標籤成對，並實際開瀏覽器確認。

## 最終回覆

寫出：修改的檔案、備份路徑、編譯結果、驗證結果、尚未實測的部分、待確認清單。
