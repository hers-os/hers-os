# Leak Detector — 三類洩漏檢查規則

> 生成腳本後、跑這份的檢查、把結果寫進 `.meta.yaml` 的 `leak_warnings` 欄。
> 不靜默放行任何洩漏（v0.2 第 3.3 節）。

---

## 類型 1：名字洩漏（name leak）— warning + metadata

### 規則
腳本字面**不可**出現任何被借的 reference 的名字（含本名、藝名、暱稱、別名）。

### 怎麼查
1. 從每個被借的 reference 檔讀「核心背景速覽」一節、抓出：
   - 主名（檔名、如「楊天真」）
   - 本名（如「楊思維」）
   - 媒體暱稱（如「濃人」「職場明燈」← 從 tier-B 抓）
2. 對腳本做字串包含檢查（不分大小寫、不分繁簡、空格忽略）

### 命中怎麼辦
- 寫進 metadata：
  ```yaml
  leak_warnings:
    - type: name_leak
      reference: 楊天真
      hit: "腳本第 N 行出現「楊天真」"
      action_required: 改寫
  ```
- 告訴使用者：「字面出現了 reference 的名字、違反鐵律 1『不掛名』、建議改寫成『有人說過』『有位前經紀人說過』或直接拿掉」
- 不自動改、等使用者決定

---

## 類型 2：tier-A 原句洩漏（string leak）— warning + metadata

### 規則
reference 檔內標 **tier-A** 的引號內整句、**用過即忘**（v0.2 第 5.2 節 #6 建議方向）。腳本字面**不可**整句出現。

### 怎麼查
1. 從每個被借的 reference 檔讀「引用語錄附錄 → tier-A」一節、抓出所有引號 `「⋯」` 內的句子
2. 對腳本逐句做兩種匹配：
   - **完全匹配**：腳本句 == tier-A 句（一字不差、含標點）
   - **高重合匹配**：腳本句與 tier-A 句的字串重合度 ≥ 70%（用 longest common substring 估、不需精準演算法、目測也行）

### 命中怎麼辦
- 完全匹配 = 強警告、必須改寫
- 高重合 = 一般警告、強烈建議改寫
- 寫進 metadata：
  ```yaml
  leak_warnings:
    - type: tier_a_leak
      reference: 楊天真
      tier_a_source: "真誠是溝通中最重要的、沒有之一"
      match_type: 完全 / 高重合
      hit: "腳本第 N 句"
      action_required: 改寫成意思相同但句法不同
  ```

### 改寫指引
tier-A 是「理解這個人怎麼想」用的、**不**是寫進腳本用的。如果腳本確實想表達「真誠最重要」的意思、改成緣緣的句法：
- ❌ 原樣：「真誠是溝通中最重要的、沒有之一」
- ✅ 改寫：「我們相信真誠走得最遠、技巧只是讓真誠更容易被收到」

---

## 類型 3：反胃詞（vomit terms）— 必須改、不放行

### 規則
`voice/style-guide.md` 1.7 + v0.2 第 3.2 節列的 **5 個鎖死反胃詞**、無論借誰、絕不能放。

### 鎖死清單
| 詞 | 變體 |
|---|---|
| 寶寶 | （含「寶貝」「baby」） |
| 乾貨 | — |
| 絕 | 絕子、絕了 |
| YYDS | yyds、永遠的神 |
| 泉 | （吹牛意） |

### 加碼建議檢查（非鎖死、但 warning）
style-guide 1.7 還有：
- 小紅書黑話：水水們、姐妹們、家人們
- 抖音梗：破防、emo、yyds（已含）
- 台灣網路用語：87、傻眼貓咪

這些命中時 warning、不阻擋、進 metadata。

### 命中怎麼辦
- 5 個鎖死詞 = **強制要求重生這段**、不寫進 metadata 就放行
- 加碼建議詞 = warning：
  ```yaml
  leak_warnings:
    - type: vomit_term_soft
      hit: "腳本第 N 行：『家人們』"
      action_required: 建議改寫
  ```

---

## 跑完後輸出格式

把所有 warning 集中成 `leak_warnings:` 陣列、塞進 `.meta.yaml`、見 `templates/meta.yaml`。

如果**完全沒命中**、欄位寫：
```yaml
leak_warnings: []
```

不要寫「無」或省略欄位、要明確空陣列、表示「跑過了、結果是乾淨」。「對輸出誠信」原則。
