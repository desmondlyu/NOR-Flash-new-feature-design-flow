# Log Format

## 測試前說明（在 y/n 之前印出）

```
<TEST NAME> AUTO TEST
Size = 32Mb (00400000h bytes), SPI clock = 25MHz

---------------------- HOW TO READ THIS LOG ----------------------
Step "2.4" = Test Flow 2, step 4.
Each step prints these lines:
  Do     : the instructions that the tester sends
  Expect : the result that the Datasheet gives
  Got    : the values that the tester reads
  Result : the step result
  Detail : more data for the step. A Detail line is not a step.

Step results:
  PASS        The device does what the Datasheet says.
  FAIL        The device does not do what the Datasheet says.
  NOT READY   The start condition is wrong. This is not a device fault.
  NOT RUN     The tester did not do this step.
  RETEST      The test condition did not occur (example: WEL=0). Do it again.
  INFO        The Datasheet gives no rule. The tester records the value only.
  ASK DESIGN  The Datasheet gives no rule, and the result is not safe.
              Design decides.
FAIL, NOT READY and RETEST make the final result FAIL. INFO and ASK DESIGN do not.
After a FAIL in Flow 0-5, the next steps do not run. Flow 6 always runs.

Instructions:   <opcode + Datasheet name, one per line>
Terms:          <each register bit or short name, one line each>
Test areas:     <address, size, data, use>
WARNING:        <steps that change the device for good>
Start the test? (y/n):
```

## 步驟

```
[1.1] Write V-IBP on block 070000h
      Do     : Write V-IBP (E1h) 070000h FFh, read V-IBP of all 94
      Expect : 070000h=FFh, all others 00h
      Got    : 0 of 94 differ
      Result : PASS
```

- 標題：動詞開頭或描述規則，≤60 字元。
- `Do` 太長時用續行，縮排 15 格對齊。
- `Got` 寫讀到的值，不寫 PASS/FAIL。
- 非 PASS/FAIL 的 `Result` 後面加括號說明原因。
- `Detail : ... -> CODE` 用於診斷或記錄，不計步。
- 停止時加一行：`Note   : the next steps do not run. The test goes to Flow 6.`
- 邊界 Flow 每步後加 `State  :`，寫下一個步驟的起始狀態。

## 結尾

```
========================= TEST SUMMARY =========================
Flow       What it checks                  Done  PASS FAIL INFO ASK NOT-RUN
0          Pre-check, test data            6/6     6    0    0   0    0
...
Done = steps run / steps planned. "Detail" lines are not counted as steps.
FAIL counts FAIL, NOT READY and RETEST.
================================================================
Restore status: Flow 6 all PASS. The device is back to default.
Total 63: PASS 54, FAIL 0, NOT READY 0, RETEST 0, NOT RUN 1, INFO 8, ASK DESIGN 0
FINAL RESULT = PASS
Follow-up: read the INFO, ASK DESIGN, NOT READY and NOT RUN steps: 1.2, 4.3, 5X.4
```

## 常見錯誤

| 不要寫 | 改寫成 | 原因 |
|---|---|---|
| `[WP-008] A-2 START positive control` | `[1.2] Erase with WPS=0` | 內部簡稱讀者看不懂 |
| `Expected=E0h Middle=FFh` | `Expect : 070000h=FFh` | 用位址，不用內部區域名 |
| `(5.3.14)` | 刪除 | 不同料號章節不同 |
| `Result=SETUP_FAIL` | `Result : NOT READY` | 讀者不懂內部代碼 |
| `has been erased` | `is erased` | 不用完成式 |
