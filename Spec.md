# 21點（Blackjack）單檔應用（index.html）— 軟體開發規格

版本：1.0  
文件目的：定義「單檔（index.html）」的 21 點 PoC 功能、介面、資料、演算法與驗收標準。  
交付範圍：`Spec.md`（本文件）與 `index.html`（單檔實作，需符合本規格）。  
備註：以下「抽牌」指從牌堆隨機抽取撲克牌（發牌/要牌），以符合 21 點遊戲語境。

---

## 0. 名詞定義
- **牌堆（Deck）**：一副 52 張撲克牌（PoC 固定 1 副）。
- **發牌（Deal）**：新局開始時，玩家與莊家各拿 2 張。
- **要牌（Hit）**：玩家或莊家再抽 1 張牌。
- **停牌（Stand）**：玩家停止要牌，輪到莊家回合。
- **爆牌（Bust）**：點數 > 21。
- **Blackjack**：前兩張為 A + 10點牌（10/J/Q/K）。

---

## 1. 產品目標（Goals）
建立一個可離線開啟即玩的 21 點頁面，提供：
- 基本 UI（控制區 + 玩家/莊家手牌 + 狀態訊息）
- 隨機洗牌與發牌/要牌（無放回）
- 正確點數計算（A=1或11）
- 基本操作（New Game / Hit / Stand / Clear）
- 簡單敘述（回合提示、莊家行為、勝負原因）

---

## 2. 交付物（Deliverables）
1. `index.html`（單檔）
   - 必須包含：HTML + CSS + JavaScript（可內嵌於同檔）
   - 必須可離線執行：不可依賴外部 CDN 或外部資源
2. `Spec.md`（本文件）

---

## 3. 執行環境與限制（Constraints）
- 瀏覽器：Chrome / Edge / Firefox / Safari（近兩年版本）
- 無後端、無 API、無 DB
- PoC 固定規則：
  - 1 副牌（52張）
  - 莊家規則：**16 以下要牌、17 以上停牌**
  - Soft 17：預設 **S17（Soft 17 停牌）**
- 不含下注、加倍、分牌、保險、投降（可列為未來擴充）

---

## 4. 功能需求（Functional Requirements）

### FR-1：基本 UI
頁面必須包含：
1. Header
   - 標題（例如「21點 PoC」）
   - 規則摘要（例如「1副牌｜S17｜Dealer hits ≤16」）
2. 遊戲區
   - **莊家區（Dealer Panel）**
     - 顯示莊家手牌（玩家回合時需隱藏莊家第二張暗牌）
     - 顯示莊家點數（玩家回合可顯示部分或顯示「?」）
   - **玩家區（Player Panel）**
     - 顯示玩家手牌
     - 顯示玩家點數
3. 控制區（Control Panel）
   - 「New Game / 新局」按鈕
   - 「Hit / 要牌」按鈕
   - 「Stand / 停牌」按鈕
   - 「Clear / 清除」按鈕（清空畫面或回到 idle 狀態）
4. 訊息區（Message Area）
   - 顯示簡單敘述（例：輪到玩家、玩家爆牌、莊家補牌到 19、你獲勝/平手等）

### FR-2：隨機洗牌與抽牌（無放回）
- 必須建立 52 張牌的牌堆（唯一組合：4 花色 * 13 點數）。
- 洗牌需使用 Fisher–Yates（或等價無偏）演算法。
- 抽牌必須無放回：同一局同一副牌內不會抽到相同牌。

### FR-3：新局發牌流程
按下「New Game」後：
- 進入玩家回合（player_turn）
- 建立並洗牌新牌堆
- 發牌（固定順序即可，需一致）：
  - 玩家 2 張
  - 莊家 2 張（第二張為暗牌）
- 顯示玩家兩張牌與點數
- 顯示莊家第一張明牌，第二張顯示背面/🂠
- 訊息區顯示提示（例如「輪到你：要牌或停牌」）

### FR-4：玩家要牌（Hit）
在玩家回合按 Hit：
- 玩家抽 1 張牌加入手牌
- 更新玩家點數並重新渲染 UI
- 若玩家點數 > 21（爆牌）：
  - 立即結束（finished）
  - 揭露莊家暗牌
  - 訊息顯示：「你爆牌（X），莊家獲勝」

### FR-5：玩家停牌（Stand）與莊家回合
在玩家回合按 Stand：
- 揭露莊家暗牌
- 進入莊家回合（dealer_turn），莊家依規則自動要牌：
  - 莊家點數 < 17 → 必須 Hit
  - 莊家點數 > 17 → Stand
  - 莊家點數 == 17：
    - 若為 Soft 17 且採 S17 → Stand
- 莊家回合結束後判定勝負並顯示結果敘述

### FR-6：點數計算（Ace 1/11）
- 2~10：牌面點數
- J/Q/K：10 點
- A：1 或 11（取「不爆牌且最大」）
- 必須支援多張 A 的最佳化計分（例如 A+A+9 = 21；A+9+9 = 19（A=1））

### FR-7：勝負判定與簡單敘述
結算（finished）規則：
- 玩家爆牌 → 玩家輸
- 莊家爆牌 → 玩家勝
- 都未爆牌：
  - 玩家點數 > 莊家 → 玩家勝
  - 玩家點數 < 莊家 → 莊家勝
  - 相等 → 平手（Push）
訊息區至少需產出一條易懂敘述，例如：
- 「你獲勝（20 vs 18）」
- 「平手（19 vs 19）」
- 「莊家爆牌（22），你獲勝」

### FR-8：按鈕狀態（最小狀態機）
- idle（未開局）：
  - Hit/Stand disabled
  - New Game enabled
- player_turn：
  - Hit/Stand enabled
  - New Game 可 enabled（按下視為直接重開一局）
- dealer_turn：
  - Hit/Stand disabled
- finished：
  - Hit/Stand disabled
  - New Game enabled

---

## 5. 非功能性需求（Non-Functional Requirements）
- 離線可用：不得載入外部 JS/CSS/字型/圖片
- 響應式設計：手機可用、牌區自動換行
- 效能：每次操作（Hit/Stand）UI 更新 < 200ms（一般設備）
- 可維護性：牌堆生成、計分、莊家邏輯需以函式清楚分離

---

## 6. 資料規格（Data Specification）

### 6.1 Card
欄位：
- `suit`: '♠'|'♥'|'♦'|'♣'
- `rank`: 'A'|'2'..'10'|'J'|'Q'|'K'
- `value`: number（A 初始為 11；JQK=10；2..10=本身）
- `id`: string（例如 "♠A"、"♦10"）

### 6.2 Deck
- `cards: Card[]`
- `draw(): Card`（pop 或 shift）
- `shuffle(rng): void`

### 6.3 Hand
- `cards: Card[]`
- `score(): number`
- `isBust(): boolean`
- （選配）`isBlackjack(): boolean`

### 6.4 GameState
- `phase: 'idle'|'player_turn'|'dealer_turn'|'finished'`
- `deck: Deck`
- `playerHand: Hand`
- `dealerHand: Hand`
- `dealerHoleHidden: boolean`
- `message: string`
- `rules: { dealerStandOnSoft17: boolean }`

---

## 7. 演算法規格（Algorithm Specification）

### 7.1 洗牌（Fisher–Yates）
- 以 rng() 產生 [0,1)：
  - j = floor(rng()*(i+1))
  - swap(a[i], a[j])

### 7.2 計分（Ace 動態 1/11）
1. total = sum(values，A 先當 11)
2. aceCount = number of A
3. while total > 21 and aceCount > 0:
   - total -= 10（把一張 A 從 11 轉為 1）
   - aceCount--
4. return total

### 7.3 Soft 17 判定（莊家）
- 若手牌含 A 且存在一種計分方式使 A 仍為 11 且總分=17 → 視為 Soft 17
- PoC 可用 helper `isSoft(hand)` 或在 score 計算時回傳額外資訊（是否使用 A=11）

---

## 8. UI 呈現規格（UI Rendering）
- 每張牌顯示形式：
  - 例如「♠A」「♥10」等文字牌面（不需圖片）
- 莊家暗牌：
  - 玩家回合顯示「🂠」或「[Hidden]」
  - 結算/莊家回合揭露真牌面
- 訊息區顯示簡單敘述（必填）
- （選配）動效：淡入/滑入皆可，不影響功能

---

## 9. 錯誤處理（Error Handling）
- 在非 player_turn 按 Hit/Stand：忽略或提示（但不得破壞狀態）
- deck 不足（理論上 PoC 每局重建）：若發生，應自動重開新 deck 或提示錯誤

---

## 10. 無障礙（Accessibility）最低要求
- 按鈕具有可聚焦（Tab）與可鍵盤觸發（Enter/Space）
- 訊息區使用 `aria-live="polite"` 讓狀態更新可被讀取
- 主要區塊有清楚標題或 aria-label（Dealer/Player/Controls）

---

## 11. 驗收標準（Acceptance Criteria）
1. `index.html` 可離線開啟並正常運作。
2. New Game 後玩家與莊家各發 2 張，莊家第二張暗牌正確隱藏。
3. Hit 會讓玩家新增一張牌且無重複牌；爆牌即結束並顯示原因。
4. Stand 後莊家會依規則自動補牌並停牌，最後判定勝負並顯示（含點數對比）。
5. A 的計分符合 1/11 規則，不會出現錯誤計分。
6. UI 在手機視窗寬度下仍可閱讀與操作。

---

## 12. 測試案例（Test Cases）
- TC-01：New Game → 玩家=2張、莊家=2張且第二張隱藏
- TC-02：玩家連續 Hit 直到 bust → message 顯示「你爆牌」
- TC-03：玩家 Stand → 莊家自動補牌至 17+ → 結算 message
- TC-04：含 A 的計分：
  - A+9+9 應為 19（A=1）
  - A+A+9 應為 21
- TC-05：平手情境（可用 seed 或 mock）→ 顯示 Push
- TC-06：按鈕狀態：finished 時 Hit/Stand disabled

---

## 13. 未來擴充（Out of Scope）
- Double / Split / Insurance / Surrender
- 下注與籌碼
- 多副牌與牌鞋、洗牌點
- 基本策略提示與教學模式
- 歷史紀錄與統計（LocalStorage/後端）
