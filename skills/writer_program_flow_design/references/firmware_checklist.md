# Firmware Checklist（LEAPER-56 實例）

本檔記錄 LEAPER-56 `LP_ICIDCheck.c` 的已知限制。換平台時，逐項重新確認，不要直接沿用。

## 檔案

| 項目 | LEAPER-56 值 | 怎麼確認 |
|---|---|---|
| 編碼 | CP950（Big5） | `open(p,'rb').read().decode('cp950')` 不出錯 |
| 換行 | CRLF | 單獨 LF 數 = `count(b'\n') - count(b'\r\n')` 必須為 0 |
| 修改方式 | Python 二進位讀寫，產生 `.new` | — |
| 備份 | `backup\LP_ICIDCheck.c_backup_YYYYMMDD_HHMM` | — |
| change_log | ASCII＋CRLF＋英文 | `all(b<128 for b in data)` |

## printf（at91lib）

| 限制 | 對策 |
|---|---|
| 不支援 `%-`（左對齊） | 用固定字串補空白，或 `%3u` 這類右對齊 |
| 寬度只支援數字與 0 填充 | `%02X`、`%08X` 可用 |
| 單次輸出 ≥100 字元會當機 | 拆成多次 printf，或用分行輸出函式（例如 `kk_wpx_text`，`|` 換行並縮排） |
| UART 輸出慢 | 時序窗口內不 printf |

## 編譯（DriverUpdate 目錄）

```powershell
$env:PATH="C:\Program Files (x86)\CodeSourcery\Sourcery G++ Lite\bin;$env:PATH"
arm-none-eabi-gcc -Wall -mlong-calls -ffunction-sections -g -Os -ILIB/ECPV -ILIB/ECPV/lp56_writer -I../external_libs/fat/fatfs/src -I../at91lib/boards/at91sam9rl-ek -I../at91lib/peripherals -I../at91lib/components -I../at91lib -I../external_libs/fat -I../at91lib/memories -Dat91sam9rl64 -DTRACE_LEVEL=0 -Dsdram -c -o $env:TEMP\x.o -x c <file>
```

正式燒錄檔用 `make.bat`，由使用者執行。

## 已知樣品／治具行為（2026-11 W25Q32RW `EF 80 16`）

- `06h→05h→D8h`：`D8h` 不執行。Erase／Program 不插 `05h`。
- WPS=0 時 V-IBP=FFh 仍擋 `D8h`（Datasheet 沒寫），判 INFO。
- 讀到 BUSY 後先 printf 再送指令，操作可能已結束，造成誤判。
- 舊程式曾因輸出太快讀不到 BUSY，固定輪詢至少 100ms。

## 函式分層（建議）

| 層 | 例子 | 規則 |
|---|---|---|
| 底層 SPI | 送指令、讀 SR、Power OFF/ON | 沿用既有函式，不重寫 |
| 測試動作 | rebuild 測試資料、掃 IBP、比對區域 | 回傳差異數或 ok，不 printf 結果 |
| 判定 | `begin`、`judge`、`pf`、`text` | 唯一會印 Do/Expect/Got/Result 的地方 |
| 流程 | `run()` 依 Flow 呼叫 | 用 goto 或旗標跳到復原 Flow |
