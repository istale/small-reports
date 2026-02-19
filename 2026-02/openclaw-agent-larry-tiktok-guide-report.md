# 報告：Larry（OpenClaw agent）一週拿到大量 TikTok 觀看的自動化系統（逐步拆解）

> 目的：整理並萃取〈How my OpenClaw agent, Larry, got millions of TikTok views in one week. (Full step-by-step guide)〉這篇長文的可複用做法，並轉成能在 OpenClaw 專案裡落地的流程/規格。

## 原文連結（必放）
- 目前可公開引用到的版本（archive.ph）：https://archive.ph/zefae  
  - 註：此站可能需要 CAPTCHA，工具化抓取不穩定。
- 可能的原始貼文出處（X / Twitter）：原文看起來是 X 上的長貼文/串文；由於 X 對抓取限制，這裡先以 archive 連結作為穩定引用點。

## 一句話摘要
把 TikTok 行銷拆成「可批次化的投放素材產線」：用 agent 自動化產生 *6 張固定格式* 的 photo carousel（含 hook 文案 + caption + hashtag），透過排程工具 API 上傳成 **draft**，人類只負責最後一步（挑音樂 + 發佈）。核心不是模型，而是 **skill files + 持續修正規則的記憶系統**。

---

## 系統全貌（元件清單）
文中描述的 Larry 系統大致由以下構成：

1) **OpenClaw agent（Larry）**
- 常駐在本地機器（Ubuntu / 低規格也可）。
- 有可持久化的身份、檔案型記憶（skill / memory）。
- 可呼叫外部 API、寫程式、排程。

2) **圖片生成（OpenAI gpt-image-1.5 / 或同級）**
- 用於產出「像 iPhone 實拍」的室內改造圖。
- 透過 prompt 規則確保 6 張圖一致性。

3) **後處理（文字 overlay）**
- Larry 自己寫 code 把 hook 文字疊上第一張（或每張）。
- 這是很關鍵的「格式一致」環節。

4) **投放/排程（Postiz API → TikTok drafts）**
- 透過 API 上傳 carousel 作為 **草稿**。
- 設 `privacy_level: "SELF_ONLY"`：草稿只自己可見。
- 人類最後手動加上 trending sound + 貼 caption 後發布（因為 API 無法指定音樂）。

5) **知識累積（skill files + memory files）**
- skill file：500+ 行、規格化每個細節（尺寸、字體比例、hashtags 上限、hook 模板…）。
- memory：追蹤每篇貼文表現、失敗原因 → 回寫規則 → 形成複利。

---

## 內容配方（最可複用的部分）

### A. TikTok photo carousel 固定規格
- **每支 6 張**（作者聲稱是甜蜜點）
- Slide 1：一定要有 **hook** 的文字 overlay
- Caption：故事式，且自然帶到 app
- Hashtags：最多 5 個（作者聲稱 TikTok 當前限制）

### B. 圖片一致性的關鍵：鎖定「建築結構（architecture）」
問題：室內改造類內容最怕「每張都變成不同房間」，觀看者一眼覺得假。

解法：
- 把「房間的不可變資訊」寫成一段超長、超具體的描述，**每張都 copy-paste**：
  - 尺寸、窗戶數量/位置、門的位置、鏡頭角度、家具尺寸、天花板高度、地板材質…
- 6 張之間只改變：**風格（style）**：牆色、裝潢、燈具、軟裝、氛圍光。

附帶洞見（文中強調）：
- “before” 房間要「現代但疲乏」而不是破爛；加入生活痕跡（杯子、遙控器）更像真實租屋。

### C. Hook 的關鍵公式：不是功能，是「人」與「衝突」
作者的高表現 hook 類型：
- `另一個人 + 衝突/懷疑 → 我把 AI 結果給他看 → 他改變想法`

例：
- 「房東說不能改，我就給她看 AI 覺得可以長怎樣」

這個部分可視為可移植的「hook 生成規則」：
- 問自己：*Who is the other person? What is the conflict?*
- 沒有人/沒有衝突 → hook 很可能 flop。

---

## 迭代與失敗記錄（為何一週內能變好）
文中最有價值的是「失敗→規則化」的速度：

1) **尺寸/比例錯誤**
- 一開始用 landscape 導致黑邊、降低完讀/互動。
- 規則化：固定用 1024×1536 portrait。

2) **文字 overlay 不可讀**
- 字太小、位置被 UI 擋、換行導致壓縮變形。
- 規則化：字體比例、位置、安全區、每行字數上限等。

3) **hook 太自我中心**
- 談功能/價格/自己感受，數據很差。
- 規則化：用「他人 + 衝突」敘事。

這其實就是一個「內容版 TDD」：
- 每次失敗都變成 skill file 的新規則，下一次永遠不再犯。

---

## 可落地的 OpenClaw 實作建議（把文章變成可執行 SOP）

### 1) 建議的資料結構（落盤）
- `projects/tiktok-growth/`
  - `SKILL_TIKTOK_CAROUSEL.md`（格式規格 + 產線流程）
  - `HOOKS.md`（hook 模板庫 + 反例）
  - `PROMPTS.md`（architecture lock 模板 + style 變量）
  - `POST_LOG.jsonl`（每次發文：hook、caption、hash、生成成本、views/likes/comments、結論）

### 2) 建議的工作流（每天/每週）
- 週期性：
  1. 研究：找當週 carousel 表現好的帳號/模板
  2. 產生：一次產 10–15 個 hooks（批次）
  3. 排程：用 cron 讓 agent 在離峰批次生成素材
  4. 上傳：透過 Postiz API 產 draft（SELF_ONLY）
  5. 人工：挑 trending sound + 貼 caption + 發佈（1 分鐘/篇）
  6. 回寫：把每篇成效與失敗原因寫入 skill/memory

### 3) 風險與注意事項
- 平台政策：TikTok 對自動化/批量投放有風險，務必用 drafts + 人工 publish 降低風險。
- 數據歸因：觀看數≠轉換；要同步記錄下載/試用/付費（RevenueCat 之類）才能避免誤判。
- 內容倫理：AI 生成的「看似真實照片」要避免誤導（尤其醫美類/臉部類）。

---

## 我們可以直接抄的「最小版本」
如果你要在 OpenClaw 上最快複刻（MVP）：
1) 固定 6 張 carousel 規格 + 1024×1536
2) architecture lock prompt（copy/paste 不可變描述）
3) hook 公式：他人 + 衝突 → 展示 AI → 反轉
4) Postiz API 上傳 drafts（SELF_ONLY）
5) 每篇發完記錄成效 → 失敗寫成規則

---

## 附註（資料來源說明）
本報告以你提供的原文內容為主，並附上可公開引用的 archive 連結作為來源定位點。若你之後提供原始 X 貼文 URL（或作者同步到 blog/newsletter 的 URL），可再補上更直接的來源連結。
