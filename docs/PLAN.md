# 玄機籤閣 — 整體架構規劃（SSOT）

> 2026-07-07 定稿。之後加功能**先讀本檔**：位子在哪、接哪些共用件、排在哪一版。
> 改了架構決策要回來改本檔（superseded 標註，不刪）。

## 0. 產品定位（一句話）

**結合線上求籤、擲筊與塔羅的每日占問工具**——台灣求籤儀式感 × 塔羅 × 黃曆，差異化靠「儀式流程」不靠功能堆疊。

## 1. 資訊架構（終局版，NAV 註冊表容納全部）

```
🏮 求問    ── 求籤 ｜ 抽卦 ｜ 擲筊
🔮 塔羅    ── 單張 ｜ 三張 ｜ 混合陣(三牌+一籤)      ← 牌陣=資料不是程式
⭐ 星座    ── 今日運勢 ｜ 配對 ｜ 月相水逆
📜 曆法    ── 農民曆 ｜ 今日氣場
📖 紀錄簿  ── 右上角 icon（不佔主分頁），時間軸讀 xj_journal
```

判準：**主分頁永遠 ≤4 組**。新功能先問「屬於哪一組的子頁」，都不屬於才考慮動主分頁。

## 2. 分層架構（各層只往下依賴）

```
┌ SEO 靜態頁（v1.5，scripts/ 生成，深連結回 app）
├ 功能層：每功能 = NAV 一筆 + view div + <功能>-data.js
├ 共用核心（kernel，只寫一份）：
│   $ / rnd / buzz / toast / share / escapeHTML / wishHTML
│   LS（localStorage 包裝）/ dateSeed / pad / fmtI / parseI
│   lunarOf / almanacOf / T()簡→繁
│   ✅ seedRnd(seed) 有種子PRNG（v1.1a 已入）
│   ✅ logEntry(type,payload) 紀錄簿統一寫入口（v1.1a 已入）
│   〔待補〕motion layer：CSS tokens（--motion-fast/--motion-ritual）＋共用 keyframes
│           （pop/pulseGlow/flipCard/landRipple）＋ anim(el,cls) helper
│           （animationend 與 timeout 賽跑防卡死；只動 transform/opacity/filter）─ v1.1e 前置
├ 引擎/資料：lunar.js ｜ tarot-data.js ｜ QIAN/GUA(暫留主檔) ｜ zodiac-data.js…
└ 平台：Vercel 靜態 ＋（v2.0 起）唯一 serverless /api/interpret
```

規矩：
- **新資料一律獨立檔**（tarot-data.js 模式）。QIAN/GUA 留在主檔，等哪天為別的原因動到才順手搬（避免無意義 diff）。
- **共用件缺什麼，先補進 kernel 再寫功能**，不准在功能內複製一份。

## 3. 儲存 schema（localStorage，全部 `xj_` 前綴）

| key | 內容 | 狀態 |
|---|---|---|
| `xj_qian` | 今日籤 {date,lot} | ✅ 已有 |
| `xj_jiao` | 擲筊統計+設定 | ✅ 已有 |
| `xj_journal` | **append-only** 陣列 [{t,type,q,summary}]，所有功能經 logEntry() 寫入 | 待建 |
| `xj_profile` | 生日/星座（星座功能用，只存本機） | 待建 |
| `xj_settings` | 全域開關（擲筊確認流程 on/off 等） | 待建 |

判準：紀錄簿 UI 只**讀** `xj_journal`；任何功能不得自創紀錄格式。

## 4. AI 層合約（v2.0 才實作，介面現在定死）

- **唯一端點** `POST /api/interpret`：`{type:'qian'|'tarot'|…, question, payload, }` → 四段式固定輸出：`{summary, forQuestion, action, caution}`。
- 前端唯一入口 `aiInterpret()`，掛在結果卡「✨ 深度解讀」按鈕，**點了才呼叫**。
- 免費限次（每日 N 次，LS 計數）先收數據，付費牆之後才談。
- key 走 Vercel env，前端永不見 key。

## 5. SEO 層（v1.5）

- `scripts/build-seo.mjs` 從 tarot-data.js / QIAN 生成靜態頁（78 牌義＋60 籤詩＋擲筊說明），互鏈回 app 的 hash 深連結。
- 打長尾（「聖杯六逆位 感情」），避開「每日黃曆」紅海大詞。
- 同版加：OG meta、manifest.json+icon（PWA）、Vercel Analytics（先有數據才談變現）。

## 6. 版本排程（依賴順序，不是願望清單）

**v1.1a kernel 補件**：seedRnd + logEntry + xj_settings/profile 讀寫 → 現有四功能補 logEntry（各~3行）
　　*為什麼先做：紀錄簿的資料從今天開始累積，晚做=之前的占問全是空白。*

**v1.1b 星座**：`zodiac-data.js`（12星座+配對表+水逆日期表）＋ NAV 加 `astro` 組（今日運勢/配對/月相）
　　*第一個驗證「加功能=加一行」的地基收益。*

**v1.1c 曆法強化**：今日氣場（黃曆宜忌→今日適合/避免 ＋ seedRnd 日牌）

**v1.1d 求問強化**：問事分類 chips ＋ `advice-data.js` 四段式套版（吉凶級×分類）→ 掛進求籤/塔羅結果卡

**v1.1e 儀式與招牌** ✅ 已完成（2026-07-07）：
- ✅ motion layer（tokens/keyframes/anim/replay/countUp/reduceMotion）
- ✅ 擲筊 SVG 半月筊杯（jiaoSVG）＋拋落 settle＋金色 landRipple
- ✅ 塔羅翻牌：cardBackSVG 太極八卦牌背＋.flip-card 3D（棄 outerHTML）
- ✅ 收尾：配對 count-up＋scorebar fill、reel 停格金光、月相 glow、日牌 pop
- ✅ 東西合參混合牌陣：3 塔羅（現在/阻礙/未來）＋1 籤總結，logEntry `spread:dongxi`
- ✅ 擲筊確認流程：xj_settings.ritual 開關，求籤/抽牌前 requestBlessing 閘門，聖筊才開抽
- 純函式化：throwBlock/verdictOf 本就純；pickQian 已抽出；tarotRowsHTML 抽共用。
2. 擲筊確認流程（xj_settings 開關，包在抽籤/抽牌前）。
3. **儀式管線（SPREADS 註冊表）**——本產品的差異化核心。牌陣＝槽位序列，**每槽可來自不同系統**：
   ```js
   { id:"dongxi", name:"東西合參", steps:[
     {source:"jiao",  role:"請示", gate:true},      // 聖筊才開抽
     {source:"tarot", role:"現在"},{source:"tarot", role:"阻礙"},{source:"tarot", role:"未來"},
     {source:"qian",  role:"總結提醒"},
     {source:"almanac", role:"今日時運", context:true} ]}
   ```
   之後任何新陣（先筊後籤、卦+塔羅互參）＝ SPREADS 加一筆，不寫新程式。
   AI 合約不用改（payload 泛型天生吃混合結果）；logEntry type 用 `spread:<id>`。

**v1.1f 收口** ✅ 已完成（2026-07-07）：
- ✅ 紀錄簿 UI：header 📖 入口 → modal 時間軸（讀 xj_journal，按日分組、類型徽章、清空）。
- ✅ 分享圖卡：Canvas 共用繪卡器 drawShareCard(cfg)＋shareCard()（Web Share 傳檔＋下載＋文字三層退路）；求籤/塔羅/今日運勢已餵資料出圖，餘者仍走文字 share()。
- **v1.1 全部完成**。下一站 v1.5 SEO / v2.0 AI（見 §4、§5）。

**v1.5 SEO 資產**（見 §5）→ **v2.0 AI 深度解讀**（見 §4）→ 數據說話後才排變現。

## 7. 墓碑（否決過的，別重提）

| 否決 | 為什麼 |
|---|---|
| 遷框架（React/Next） | 單檔+註冊表已滿足擴充；框架只在「帳號+DB」時才回本 |
| v1 做完整星盤（上升/宮位） | 出生時間+時區+宮位系統=另一個產品的複雜度 |
| DB/帳號/推播/結果追蹤提醒 | 全是 Tier 2 門檻，流量沒證明前不碰；回訪先靠今日氣場+每日一牌 |
| 夢境解析、占星合盤 | 稀釋定位；合盤 v2.0 後看數據，且第一版只用塔羅牌陣不算星盤 |
| 星座直接塞第 6 顆主分頁 | 已被 v1.05 註冊表+分組取代 |
| 變現先行 | 沒 Analytics 數據就定價=自欺；先免費限次收數據 |
| 常態背景動畫（燈籠搖曳/飄粒子）、農民曆宜忌跳動 | 搶主功能焦點；資訊頁只准淡入（2026-07-07 動畫檢視定案） |
| 動畫先做、v1.1e 後做 | 擲筊/塔羅動畫與 v1.1e 重構碰同段程式，分開做=白工一次 |
