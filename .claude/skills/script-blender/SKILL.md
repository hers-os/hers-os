---
name: script-blender
description: 為赫絲（緣緣）生成 60 秒內的 IG Reels / TikTok 腳本、把 voice/references/ 裡的角色「邏輯滲透」進緣緣的嘴。當使用者說「我要寫一支聊 X、借楊天真」「來一支 Reels 腳本」「幫我寫個 60 秒 IG 短影音」「blend 一支關於 Y 的」「短影音腳本」或任何用 voice/references/ 借人寫腳本的需求、都要觸發這個 skill。即使使用者沒講 /blend、只要意圖是「赫絲腳本 + 借誰」、就用這個 skill。
---

# script-blender — 赫絲腳本融合師

> **這個 skill 的核心**：用緣緣的嘴、借 reference 的判斷邏輯、生 60 秒以內的短影音腳本、輸出進 `03-scripts/`。
>
> **絕不違反**：`CLAUDE.md` 最高原則「誠信為本」、`voice/style-guide.md` 紅線 1-4、`00-design/blend-architecture-v0.2.md` 第 3.2 節「絕對鎖死」清單。

---

## 0. 啟動前自檢（缺料就停、不要硬寫）

進入對話前先確認以下檔案存在、缺任一個就告訴使用者「校準包不齊、無法生成」、不要假裝有資料：

- `voice/style-guide.md` ← 緣緣語氣規則（必讀）
- `brand-facts.md` ← 不在這裡的數字 / 事件不能寫進腳本（誠信為本）
- `CLAUDE.md` ← 紅線清單
- `voice/references/` ← 至少 1 個非 `_template.md` 的 reference 檔（如果使用者要借）
- `voice/samples/` ← 嘴感校準（有就讀、沒有不阻擋、但 metadata 標記 `voice_sample_count: 0`）

「啟動前校準包」精神來自 `00-design/blend-architecture-v0.2.md` 第 4 節。

---

## 1. 對話流程（這個 skill 是對話入口、不是指令）

使用者只要表達「想寫腳本」、就用以下節奏問完 7 個關鍵欄位、不要一次轟炸全問、**一輪 1-2 題**、讓使用者有節奏感。

每問完一輪都簡短覆述「我目前理解是 ___、對嗎？」、再問下一輪。

### Round 1：主題 + 借不借

1. **主題**：這支腳本要聊什麼？（一句話、不限長度）
2. **借不借 reference**：
   - 不借 → 跳到 Round 4（純緣緣模式、嘴 100%、邏輯也 100% 緣緣）
   - 借 → 進 Round 2

### Round 2：借誰 + 借哪一層

3. **借誰**：列出 `voice/references/` 內所有**非 `_template.md`** 檔的人名、給使用者選。
   - 首版上限 **2 人**（呼應 v0.3 對 v0.2 Q3 的覆寫、見 `00-design/blend-architecture-v0.3.md`）
   - 借 2 人時、必須在輸出 metadata 註記「雙借實驗、待累積觀察」
4. **借哪一層**：對每個選中的 reference、問借哪一層、限以下三選：
   - 反對對象（他/她反對 ___）
   - 判斷標準（他/她判斷好壞看 ___）
   - 使命（他/她長期想推 ___ / 避免 ___）

   ⚠️ **絕不可借**的層（鎖死、即使使用者要求也要拒絕並解釋）：
   - 角色身份（誰是誰）
   - 服務對象（替誰說話）
   - 內容邊界（不講什麼、不像誰）

   依據：`00-design/blend-architecture-v0.2.md` 第 2 節 Q2 拍板。

### Round 3：強度

5. **強度**：強 / 中 / 弱、預設「中」。
   - **弱**：reference 的邏輯滲透感佔判斷層比例 ~15%、整體只有一兩處能隱約感覺到
   - **中**：~30%、能感覺到味道、但不會搶緣緣的主體
   - **強**：~45%、味道明顯、有偏移風險、metadata 必須明確標記「強滲透、待 20 支審查時抽看」

   **不可超過 50%**、超過就違反 v0.2「主大腦唯一」核心。

### Round 4：平台 + 確認

6. **平台**：IG Reels / TikTok（兩者腳本結構接近、節奏微差）
7. **確認**：把 1-6 整理成一段、複誦給使用者、等「好」再生成。

---

## 2. 載入校準包（確認後做、不要每輪都讀）

確認後、一次性讀以下檔案（**只在這時讀、不要重複讀**）：

| 檔 | 讀什麼 |
|---|---|
| `voice/style-guide.md` | 通用規則 1.1-1.8、紅線、模式 A/B 結構（IG Reels 通常是 A 為主、混進 B 的金句斷言） |
| `brand-facts.md` | 確認腳本要用到的任何具體數字 / 事件 / 客戶名都在這裡、不在就不能寫 |
| `CLAUDE.md` | 紅線 1-6 |
| `voice/references/<選中的人>.md`（每人讀完整檔） | 抓 ✅ 可借的 3 欄、加「為什麼緣緣參考她」、加「絕對不要學的點」（這節用來反向避雷、不是借的內容） |
| `voice/samples/*`（如果有） | 抓緣緣真實口頭禪 / 句法當嘴感校準 |

詳細的「reference 6 欄怎麼讀、哪些可用」見 `loaders/reference-loader.md`。

---

## 3. 生成腳本

調用 `prompts/generate.md` 的 prompt 模板、套入：
- 主題、平台、強度
- 緣緣可用的判斷邏輯（從 style-guide + brand-facts）
- 借的層（reference 的 ✅ 3 欄）
- 嘴感樣本（samples）

**60 秒上限的估算**：用「中文每秒約 4-5 字」估、整支腳本字數控在 **240-300 字**。超過就請使用者選砍哪段、不要自己亂砍。

腳本結構（IG Reels / TikTok 通用）：
- **鈎子**（0-3 秒、第一行）：金句斷言、不寒暄、不自問句
- **展開**（3-30 秒）：1-2 段、每段 2-4 句、長句斷言為主
- **轉折**（30-50 秒）：對比結構或畫面（style-guide 3.2 句型）
- **收**（50-60 秒）：留白、最多一句感受收、**絕不寫 CTA**

---

## 4. Leak 檢查（生完後跑、warning + metadata、不 reject）

依 `checks/leak-detector.md` 走完三類檢查：

1. **名字洩漏**：腳本字面是否出現任何 reference 的名字（楊天真、鄧紫棋⋯⋯）
2. **tier-A 原句洩漏**：腳本是否含 reference 檔內 tier-A 引號內整句（即使一字不差）
3. **反胃詞**：是否含 style-guide 1.7 的 5 個鎖死反胃詞（寶寶、乾貨、絕、YYDS、泉）

**處理方式**：
- 反胃詞鎖死 5 個 = **必須改、不能放行**、原地請使用者重生這段
- 名字洩漏 = warning、進 metadata、問使用者要不要改寫
- tier-A 洩漏 = warning、進 metadata、強烈建議改寫成意思相同但句子不同

絕不靜默放行任何洩漏。「以正合、以奇勝、奇兵也要有紀律」（v0.2 第 3.3 節）。

---

## 5. 輸出（雙檔、進 03-scripts/）

如果 `03-scripts/` 目錄不存在、先建。命名：

```
03-scripts/YYYY-MM-DD-<平台>-<題目精簡>-v<N>.md
03-scripts/YYYY-MM-DD-<平台>-<題目精簡>-v<N>.meta.yaml
```

- 日期用 `currentDate`（系統提供）、不要猜
- 平台：`igreels` 或 `tiktok`
- 題目精簡：5-8 字、空格換 `-`
- 版本：先掃同名是否存在、不存在從 v1、存在就 v(N+1)

腳本檔（`.md`）用 `templates/script.md`。
Metadata 檔（`.meta.yaml`）用 `templates/meta.yaml`、必填：
- borrow（誰、哪層、強度）
- loosened_rules（這支放寬了哪些 v0.2 第 3.1 節的鐵律）
- leak_warnings（leak 檢查結果）
- review_status: `待緣緣事後審查`
- voice_sample_count（生成時讀到幾個 samples、揭露樣本厚度）

---

## 6. 回報（最後一步）

生完雙檔後、用一段話告訴使用者：
- 寫到哪 / 字數多少 / 估秒數
- 哪些規則放寬了（loosened_rules 條目）
- 哪些 leak warning（如有）
- 建議：第幾支腳本了（從 `03-scripts/` 數）、離 v0.2 第 2.3 節「20 支累積審查」還差幾支

回報語氣要平、不要炫。「對自己誠信」= 不灌水。

---

## 7. 不可以做的事（紅線）

- ❌ 把 AI 產的腳本回寫進 `voice/samples/`（自我污染、CLAUDE.md 紅線）
- ❌ 借「角色身份 / 服務對象 / 內容邊界」三鎖死層、就算使用者拍桌也不行（v0.2 Q2 拍板）
- ❌ 引用 tier-C 句子（reference 內標 tier-C 的、查無出處、絕不寫進腳本）
- ❌ 在腳本內掛 reference 的名字（鐵律 1「不掛名」、v0.2 第 3.2 節）
- ❌ 寫「快來預約 / 立即下單 / DM 我」（style-guide 1.6）
- ❌ 在 brand-facts.md 之外發明任何具體數字 / 客戶名 / 事件
- ❌ 把模式 B 的哲學語言（指數型增長、賦能、賽道）塞進模式 A 的美甲日常文
- ❌ 第一版同時借超過 2 人（v0.3 對 v0.2 Q3 的放寬上限）

---

## 8. 出處與依據

- `CLAUDE.md` 最高原則「誠信為本」
- `voice/style-guide.md` v0.2（語氣真實狀態）
- `00-design/blend-architecture-v0.2.md`（拍板的 Q1-Q4、鐵律分區）
- `00-design/blend-architecture-v0.3.md`（對 v0.2 Q3「一次一個」放寬到「首版 2 人」、Q4「指令格式」改為「對話入口」的覆寫紀錄）
- `voice/references/_template.md`（6 欄結構）
- `brand-facts.md`（可引用的事實邊界）
