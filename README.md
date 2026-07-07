# 玄機籤閣

生活小 app：每日求籤 · 易經抽卦 · 擲筊請示 · **塔羅占卜** · **農民曆**。單一自包含 HTML，手機優先，離線可用（可加到主畫面）。

## 檔案
- `index.html` — 全部畫面與邏輯（無框架、無 build）。直接用瀏覽器開即可。
- `lunar.js` — 農民曆引擎（[lunar-javascript](https://github.com/6tail/lunar-javascript)，6tail，MIT）。離線計算農曆／干支／節氣／建除／宜忌／沖煞方位。
- `tarot-data.js` — 塔羅 78 張繁中牌義（正/逆位）。
- `zodiac-data.js` — 星座資料：12 星座/元素配對規則/每日運勢文案池/水逆日期表（2026–2027，已查證：Old Farmer's Almanac、Britannica、Astro-Seek 三源一致）。運勢＝seedRnd(日期^星座) 套版組合，同日同座固定；月相用 lunar.js `getYueXiang`＋農曆日近似。
- `cards/` — 塔羅牌面圖（Rider-Waite-Smith 1909，公有領域；圖檔取自 [metabismuth/tarot-json](https://github.com/metabismuth/tarot-json)，350×600）。
- 本機預覽：`python3 -m http.server 8848` 後開 http://localhost:8848

## 塔羅設計決策（why）
- **圖用公有領域**：RWS 繪者 Pamela Colman Smith 1951 逝，台灣（卒後 50 年）2002 起、多數國家 2022 起進入公有領域，可免費合法用；避開 US Games 重新上色的現代版。
- **牌義自寫繁中**：現成資料集只有英文且需重譯潤稿，故由 5 個 sub-agent 並行寫 RWS 經典義的繁中版（台灣口吻），再組裝進 `tarot-data.js`。
- **離線免 API**：抽牌邏輯與抽卦同模式（洗牌→抽 N 張不重複→隨機正/逆位），單張＋三張（過去/現在/未來）牌陣，願望帶入＋分享沿用共用元件。

## 農民曆設計決策（why）
- **深度＝中等**：農曆日期＋生肖＋年月日干支＋節氣＋建除十二神＋宜忌＋沖煞／喜財福神方位。宜忌為傳統通書**簡化參考**（各家版本本就有出入），UI 明白標註。
- **不自刻農曆表**：農曆換算需 1900–2100 資料表，手刻等於憑記憶敲 200 行 hex，錯一格靜默歪掉 → 改用經三個春節錨點（2024/2025/2026 皆準確落在正月初一）驗過的 lunar-javascript。
- **簡→繁**：lunar-javascript 只出簡體。掃了 42 年（2008–2050）所有可能輸出字，建了**涵蓋全部輸出**的 S→T 對照表（`S2T` / `T()`），非逐字亂猜；歧義字按語境定（谷→穀 為穀雨、涂→塗；采／舍保留）。
- **日期可選**：求籤有「今日／自選」切換（自選＝抽那天的籤＋顯示該日農曆／日干支）；農民曆分頁可用日期選擇器與前/後一天鈕回溯或看未來。

## v1.05 地基約定（加新功能照此辦，不動架構）
- **導覽**：3 組主分頁（求問｜塔羅｜曆法）＋組內膠囊子頁，由 `NAV` 註冊表驅動。**加功能＝ NAV 加一筆 + 一個 `<div class="view" id="v-xxx">`**，渲染/路由/深連結自動生效；懶載入掛 `onEnter`。單子頁的組自動隱藏膠囊列。之後星座＝新增一組 `{id:"astro",…}`。
- **hash 路由**：`#qian/#gua/#jiao/#tarot/#almanac` 可直連分享；初始路由放檔尾（避 TDZ）。
- **檔案切分**：資料一律獨立檔（`tarot-data.js` 模式，之後 `zodiac-data.js`）；共用工具（`share/toast/LS/wishHTML/escapeHTML/dateSeed/fmtI`）只寫一份，新功能引用不重寫。
- **localStorage 命名**：`xj_<功能>`（現有 `xj_qian`/`xj_jiao`）；未來紀錄簿直接掃 `xj_*`。

## 由來
2026-07-07 由 `~/Downloads/玄機籤閣.html`（原本只有求籤／抽卦／擲筊，頭部干支・節氣為近似值）搬進 `~/code/xuanji` 立為專案，新增農民曆分頁並把頭部干支／節氣改用 lunar.js 精算。
