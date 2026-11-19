# AI 全自動短影音技能包（免費版）

把「一個點子」做成 IG／Reels 直式動畫短影音：腳本 → 動畫＋字幕 → IG 說明文字。
不用會寫程式，全程用中文跟 AI 對話，**用手機就能操作**。

**第一次用，請看 [使用說明.md](使用說明.md)。**

## 怎麼開始

1. 按右上角綠色的 **Use this template** → **Create a new repository**，複製一份到你自己的 GitHub。
2. 照 [使用說明.md](使用說明.md) 設定 Claude Code 的環境（只要做一次）。
3. 跟 AI 說「幫我設定品牌」，接著說「用 reel-pipeline，主題是＿＿」，就開始做第一支影片。

## 技能一覽

| 技能 | 做什麼 | 什麼時候說 |
|---|---|---|
| brand-setup | 一題一題問你，建立自己的品牌設定（說話風格、配色、CTA） | 「幫我設定品牌」 |
| reel-pipeline | 總指揮：點子 → 腳本 → 動畫 → 說明文字 | 「用 reel-pipeline，主題是＿＿」 |
| broll-video | 錄音（或無聲）＋逐字稿 → 動畫影片 | 已經有錄音時 |
| ig-reel、ig-caption、ig-carousel、ig-repurpose、ig-plan、ig-profile、ig-human | IG 腳本、說明文字、輪播、延伸題目、排程、個人檔案健檢（第三方，MIT，見 `licenses/`） | 由 reel-pipeline 叫用，也可以單獨用 |

## 資料夾

```
brands/
  _example-sleep/        範例：熬夜族保健品品牌
  _example-wellness/     範例：身心靈品牌
  <你的品牌>/            brand-setup 會幫你建立
projects/
  example-aspartame/     範例：一支完整的動畫影片專案
  <影片名稱>/            每支影片一個資料夾
reference/               畫面風格範例
fonts/                   思源黑體（Noto Sans TC，OFL 授權）
setup.sh                 雲端環境的安裝腳本
```

## 規則

- 素材（錄音）和成品 mp4 不進 Git（見 `.gitignore`）。
- 畫面上的每個數字都要有出處，發布前逐條確認 `storyboard/sources.md`。
- 每一步 AI 都會停下來等你確認。

## 想要更完整的版本？

免費版可以做出有動畫、有字幕的完整影片。
進階版另外包含：**AI 用你自己的聲音配音**、**真人出鏡影片自動加動畫**、**剖面與 3D 動畫模組**，以及一對一的設定協助。
有興趣的話，到原本領取技能包的官方帳號詢問。
