# 原始碼補件與重建清單

## 已確認缺少的模組

`src/Group29_code.v` 第 68–76 行：

```verilog
KeyboardDecoder key_de (
    .key_down(key_down),
    .last_change(last_change),
    .key_valid(been_ready),
    .PS2_DATA(PS2_DATA),
    .PS2_CLK(PS2_CLK),
    .rst(rst),
    .clk(clk)
);
```

目前附上的 Verilog 沒有 `module KeyboardDecoder`。需要找回符合這組連接埠的實作。檔名不一定叫 `KeyboardDecoder.v`，應同時搜尋檔名及 `module KeyboardDecoder` 文字。

可優先檢查舊 Vivado 專案的 `.srcs/sources_1/`、其中的 `imports/` 或 `new/`，以及課堂提供的鍵盤範例。找到後再檢查它引用的子模組：可能需要額外來源檔，但現有材料不足以確定名稱。不要只補一個沒有功能的空殼模組。

## 重複的定義

- `speaker_control`：第 927–1017 行，以及第 1116–1206 行。
- `note_gen`：第 1019–1086 行，以及第 1208–1275 行。

兩組各自的內容在忽略註解和空白後相同。後續修復時各保留一份即可。本整理版保留來源檔原貌，尚未刪除。

`my_player_control` 的實例與定義都被註解，不是目前啟用設計的必要依賴。

## 現有來源中已找到的模組

| 模組 | 定義起始行 |
| --- | --- |
| `demo_1` | 1 |
| `pixel_gen` | 169 |
| `vga_controller` | 766 |
| `clock_divider` | 834 |
| `SevenSegment` | 850 |
| `speaker_control` | 927、1116 |
| `note_gen` | 1019、1208 |

單一 `.v` 可以包含多個模組，因此不需要因為沒有七個分開的檔案就判定它們遺失。

## 已做的檔案檢查

- 以文字檢查目前來源中的模組定義與啟用的模組實例，唯一未找到定義的模組類型為 `KeyboardDecoder`。
- 頂層有 36 個接腳位元，XDC 的 36 筆啟用接腳指定都有對應；沒有看到重複使用相同實體接腳。
- 未看到啟用的 `` `include `` 或 `$readmem*` 外部檔案讀取；音符資料直接寫在 `my_music` 陣列中。這不代表找回鍵盤模組後也不會有其他依賴。
- 沒有 Verilog/Vivado 編譯器檢查結果，不能據此保證修好以上兩類問題後就一定能通過編譯、時序或硬體測試。

## 補齊後的重建流程

1. 保留原始檔備份；移除重複的音訊模組定義。
2. 加入找回的鍵盤實作及其全部依賴。
3. 在 Vivado 建立 RTL 專案，依實際板卡選 Basys 3 的器件。官方 Basys 3 資料列出的器件為 `XC7A35T-1CPG236C`，Vivado part 名稱為 `xc7a35tcpg236-1`；仍應確認手上板卡一致。
4. 將 Verilog 加入 Design Sources，將 XDC 加入 Constraints，設定頂層為 `demo_1`。
5. 優先參考 bitstream 標頭的 2019.2 環境線索，並記錄實際驗證使用的版本。若工具對陣列初始化等語法報錯，再檢查原專案是否將來源設成 SystemVerilog；目前未驗證。
6. 執行 elaboration、synthesis、implementation；處理其他錯誤，檢查 DRC 與時序，再產生 bitstream。不要為了產生檔案而直接略過錯誤。
7. 確認實際外接音訊模組、接線、鍵盤和 VGA 螢幕，驗證五軌、速度、重設、音訊與七段顯示。
8. 在 README 記錄確實成功的工具版本、硬體、步驟與結果。

原 `.xpr`、專案 Tcl 或設定資料有助於重建，但 `.xpr` 本身不是硬體邏輯原始碼。測試平台（testbench）目前未提供，適合後續補上驗證用途；它與缺少遊戲執行所需的鍵盤實作是不同問題。
