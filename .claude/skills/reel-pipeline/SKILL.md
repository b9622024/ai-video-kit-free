---
name: reel-pipeline
description: 一支 IG Reels 從點子到可以發布的完整流程：寫中文口播腳本（借用 ig-reel 的開頭公式與結構）→ 使用者錄音（或無聲字幕版）→ 交給 broll-video 做動畫 → 用 ig-caption 寫說明文字 → 可選用 ig-repurpose／ig-carousel 延伸成輪播。當使用者說「從頭做一支影片」「幫我寫腳本再剪」「一條龍」「全流程」「有個主題想拍」，或給了一個點子還沒有逐字稿時使用。已經有錄好的錄音、只要做動畫時，直接用 broll-video。
---

# 從點子到發布（reel-pipeline）

這個技能是**總指揮**：它不自己做動畫，而是依序叫用其他技能，並負責把前一步的產出交接給下一步。

```
點子 ─▶ ① 腳本 ─▶ ② 無聲或自己錄 ─▶ ③ 動畫 ─▶ ③-2 換本人配音 ─▶ ④ 說明文字 ─▶ ⑤ 延伸（可選）─▶ 使用者自己發布
         ig-reel                    broll-video                  ig-caption     ig-repurpose
         ＋本技能                                                               ig-carousel
```

**每一步做完都停下來，等使用者確認才往下。** 這個技能永遠不會登入或發布到 Instagram（ig-* 技能本身就這樣設計，用自動化工具發文也違反 IG 使用條款）。

## 開工前必讀

| 檔案 | 什麼時候讀 |
|---|---|
| `brands/<品牌>/voice.md` | 每次開工，寫任何文字前 |
| `brands/<品牌>/brand.md`（有的話） | 步驟 ①、③ |
| `.claude/skills/broll-video/references/compliance.md` ＋ `brands/<品牌>/compliance.md` | **每次都讀**，步驟 ①、④ 都要對照 |
| `.claude/skills/ig-reel/SKILL.md`、`hooks.json` | 步驟 ① |
| `.claude/skills/ig-caption/SKILL.md` | 步驟 ④ |

品牌：`brands/` 底下每個資料夾是一個品牌（`_example-` 開頭的是範例，不能直接用）。只有一個正式品牌就用它；有好幾個就問；**一個正式品牌都沒有 → 先叫用 brand-setup 技能**。

## ig-* 技能在中文上的限制（一定要知道）

ig-* 這組技能（第三方，MIT，見 `licenses/instagram-agent-skill-LICENSE`）是為英文寫的：

- **`hookscore.py`、`beats.py` 不要用**：它們用空白切字，中文一律算成 0 字、0 秒，分數沒有意義。改用本技能的 `scripts/zh-beats.py`。
- **`caption.py` 只信「前 125 字預覽框」那一段**：中文 hashtag 偵測不到，hashtag 數量自己數（IG 上限 5 個）。
- **`ig-human` 的 `humanize.py`／`detect.py` 不要用在中文**：詞庫是英文。只採用它的原則：去掉破折號「——」、不要用「讓我們」「不僅……更……」「在這個……的時代」這類 AI 腔。
- **`hooks.json` 的 26 個公式可以用**，但要改寫成台灣口語，不要直譯。
- ig-* 技能讀寫的檔案都在 `~/.claude/instagram/`。雲端工作階段結束就會消失，所以**正本放 repo**：voice 在 `brands/<品牌>/voice.md`，紀錄寫進 `brands/<品牌>/reel-log.md`。

## 步驟 0：準備

```bash
mkdir -p ~/.claude/instagram
cp brands/<品牌>/voice.md ~/.claude/instagram/voice.md
```

建立專案資料夾 `projects/<影片名稱>/`（英數小寫加連字號），以及 `project.json`：

```json
{
  "title": "<影片標題>",
  "brand": "<brands/ 底下的資料夾名稱>",
  "mode": "A",
  "cta": { "question": "…", "action": "…", "keyword": "…" }
}
```

CTA 文字從 `brands/<品牌>/cta.json` 複製。

## 步驟 ①：寫腳本

1. **確認點子夠具體。** 照 ig-reel：一支影片只講一個重點。點子太空泛就一次問清楚：主題的「一個具體事實」是什麼（研究、數字、常見誤解）、要破解什麼說法。
2. **寫三個開頭。** 從 `hooks.json` 挑三個**不同**的公式，各寫：
   - 口播句（台灣口語，最好 4 秒內講完：字數上限＝voice.md 的語速 × 4）
   - 畫面大字（6～10 個中文字）
   列給使用者，說明你推薦哪一個、為什麼。
3. **寫全文**（用使用者選的開頭）。結構照 voice.md 的「結構習慣」，再套 ig-reel 的原則：
   - 一行一句，口語、短句；講者念出來不需要練習
   - 數字比形容詞好；沒有的數字留 `{{你的數字}}`，**不准編**
   - 研究、機構、數字都要有出處（之後會進 sources.md）
   - 結尾：講者自我介紹（若 voice.md 有）→ CTA 固定句；**整支只有一個行動呼籲**
   - 和 ig-reel 不同：開頭不用自我介紹，但結尾可以有（這是使用者的習慣）
4. **檢查：**
   ```bash
   python3 .claude/skills/reel-pipeline/scripts/zh-beats.py projects/<名稱>/script/script.txt \
     --cps <voice.md 的語速> --target <目標秒數> --keyword <CTA 關鍵字> --extra "<品牌 compliance.md 的禁用詞，用 | 分隔>"
   ```
   把「需要處理」的項目修到沒有，或向使用者說明為什麼保留。再依兩份 compliance.md 檢查一次（禁用詞、太絕對的研究說法、趣味入口只當 CTA）。
5. **存檔：**
   - `projects/<名稱>/script/script.txt`：口播全文，一行一句
   - `projects/<名稱>/script/on-screen.md`：開頭大字、每個數字／比較／清單建議配什麼畫面（給步驟 ③ 寫分鏡參考，不是分鏡表本身）
   - `projects/<名稱>/script/sources.md`：腳本裡每個數字、研究的出處，狀態「待使用者確認」
6. **停下來**，給使用者看：三個開頭、全文、預估秒數、出處清單。

## 步驟 ②：無聲或自己錄

使用者確認腳本後，先把 `script/script.txt` 複製成 `inputs/transcript.txt`（剪輯技能會拿它對齊時間點）。

問使用者要哪一種（使用者開頭已經說了就不用問）：

**選項 1：無聲模式（第一次用推薦）**
- 不用任何音檔，最快看到成品：剪輯技能步驟 1 改用 `silent-timing.py` 依閱讀速度排時間（見 broll-video SKILL.md）。
- 提醒使用者：無聲版節奏比較平均、比較像投影片；之後可以用「步驟 ③-2」換成自己的聲音，畫面不用重做。

**選項 2：自己錄音**
- 把腳本貼到手機備忘錄，用語音備忘錄錄音；句子之間停半拍，對齊會更準。
- 錄好後放 Google Drive（開「知道連結的人都能檢視」）把連結貼回來，或上傳到 GitHub 的 `projects/<名稱>/inputs/`（GitHub 網頁：Add file → Upload files）。
- 錄的時候講得和腳本不一樣沒關係，下一步會自動比對，再問你字幕要照哪個。
- **停下來等錄音。**

使用者想要 AI 用自己的聲音配音、或想把真人出鏡影片加上動畫：這個免費版沒有包含，請告訴使用者可以先用自己錄音的方式。

## 步驟 ③：動畫

照 `.claude/skills/broll-video/SKILL.md` 從它的步驟 1 開始（模式 A）。

交接重點：
- `script/sources.md` 的內容併入 `storyboard/sources.md`，不要重新查一次。
- 寫分鏡時參考 `script/on-screen.md`，但最後以剪輯技能的分鏡規則為準。
- 剪輯技能各步驟的停點照舊。

## 步驟 ③-2：換成本人配音（無聲版影片確認後）

使用者照 `script/script.txt` 自己錄好音（通常是手機語音備忘錄，放 Google Drive；也可能是 iMovie 匯出的 mp4，從裡面取出聲音即可）：

1. 下載到 `inputs/`（例：`inputs/voice-self.m4a`），**保留**原本的無聲版 `silent.m4a` 與它的 timing，不要覆蓋（timing 改放到 `storyboard/timing-silent/`）。
2. 用新錄音重跑 `transcribe.py` 產生新的 timing；`alignment.md` 有差異就照剪輯技能的規則問使用者字幕照哪個。
3. **分鏡不重做**：畫面、元件、順序都不變，只把每個畫面與元素的出現時間換成新 timing（同一句、同一個關鍵字對應的新秒數）。
4. `prepare-composition.py` → lint／check → 渲染 → 傳手機版 mp4。

## 步驟 ④：說明文字

影片確認後，照 `ig-caption` 寫說明文字，固定是它的 **Job A**（影片本身已經有開頭，說明文字不要再搶一次開頭）：

- 前 125 字：一句延伸影片主題的話＋CTA（留言關鍵字）。這段要能單獨看懂。
- 內文：影片重點 3～5 行、出處（機構＋年份）、一句提醒（例：「個人狀況請諮詢專業人員」）。
- 最後：搜尋關鍵字自然寫進句子；hashtag **最多 5 個**。
- 依 compliance.md 掃一次禁用詞。
- 存成 `projects/<名稱>/output/caption.txt`，同時把全文貼在對話裡方便複製。
- 可以跑 `python3 .claude/skills/ig-caption/caption.py output/caption.txt` 看預覽框，但它對中文 hashtag 的判讀不準。

## 步驟 ⑤：延伸（使用者要才做）

- 同一支影片拆成輪播：照 `ig-carousel`，圖檔 1080×1350，配色用品牌的 design-system／brand.md。
- 拆成其他短片題目：照 `ig-repurpose`，每個題目產出一份「點子＋開頭」，下次從步驟 ① 開始。
- 排一週的發文：`ig-plan`。

## 收尾

1. 在 `brands/<品牌>/reel-log.md` 加一行：日期、影片名稱、開頭公式、第一句。
2. 提醒使用者：**發布前逐條確認 sources.md**；發布由使用者自己來。
3. commit＋push（不含影片檔，照各剪輯技能的 .gitignore 規則）。
