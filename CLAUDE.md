# CLAUDE.md — 這個 repo 的修改方式

AT13 Hair Design 基隆仁三店的網站。每次修改都照下面的方式做。細節與背景見
`README.md` 和 `source/designer-cards/README.md`。

## 先知道：`public/` 是編譯後的快照

- `public/` 是從正式站鏡像下來的 Vite 編譯產物，原始 `src/` 不在這裡。
- 改文字或版面只能直接改壓縮過的 bundle，用精確的字串取代，不要重新排版整個檔案。
- 檔名帶 hash（例如 `content-DRgEOFBw.js`），不要改檔名，`index.html` 和其他 chunk 都靠這些名字互相引用。

## 常見修改放在哪裡

| 要改什麼 | 改哪裡 |
| --- | --- |
| 設計師名單、名字、號碼、擅長項目 | `public/assets/content-DRgEOFBw.js` 的 roster |
| 擅長項目格式 | 一位設計師一個字串，項目之間用全形 `｜` 分隔（英文版會自動換成 ` / `） |
| 團隊區版面 | `public/assets/index-B7RSM3hh.css` 最後面、兩個註解標記之間的覆寫 |
| 圖片 id 解析與 `sizes` / `srcset` | `public/assets/index-BMUe7iZr.js` |
| 頁面 metadata、JSON-LD | `public/index.html` |

## 設計師照片

- 原稿是 `source/designer-cards/` 裡老闆給的設計師卡，這些是母檔，要保留。
- 從卡片重新裁切全部照片：`python3 tools/crop-designer-cards.py`（需要 Pillow）。
  要調整裁切中心，改腳本最上面表格的 `face_x`。
- 新的單張照片：`node tools/add-photos.mjs <photo> --id <id>`（需要 `npm install` 的 sharp）。
- 輸出固定在 `public/assets/gen/{,tex/,thumb/}<id>.webp`，三種尺寸：
  1200×1600、768×1024、400×533，全部 3:4。
- 卡片上的文字不要燒進圖片；文字放在 roster 裡。
- 名字刻意和卡片不同的地方（例如櫃台在網站上寫 瑪莉）記在
  `source/designer-cards/README.md`，照那裡的為準。

## 每個修補都要記錄

因為改的是編譯產物，下次從原始碼重新 build 就會被蓋掉。所以每個修改都要：

1. 在 `README.md` 的「Patches applied on top of the snapshot」加一段，
   寫清楚改了什麼、為什麼，以及原始 `src/` 應該怎麼改。
2. 名單或人數有變時，一併檢查其他地方的數字（例如 studio 的設計師人數統計）
   和 README 的名單描述。

## 驗證

- 本機預覽：`python3 -m http.server 8788 --directory public`，
  或 `npx wrangler dev`（和正式站一樣，未知路徑會回 index.html）。
- 改圖片或 `sizes` 時，要確認手機（≤700px）和桌機寬度的實際顯示。

## 不要動的東西

- `public/robots.txt` 的 `Disallow: /` 和 `index.html` 的 `noindex, nofollow`
  是刻意的；要等老闆同意，而且作品、店內、港口等圖片都換成真實照片之後才能拿掉。

## Git 與部署

- 在分支上改，開 PR 合併進 `main`；不要直接推 `main`。
- commit 訊息用英文、祈使句，說明做了什麼和為什麼
  （例如 `Remove the four designers who have left the salon`）。
- 合併到 `main` 後，`.github/workflows/deploy.yml` 會自動部署到
  <https://at13-hair-design.wahmey.workers.dev/>（需要 `CLOUDFLARE_API_TOKEN` secret）。
