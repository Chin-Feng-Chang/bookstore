# 頁碼書店 Folio — 網路書店

簡約現代風格的網路書店，Node.js + Express 後端、PostgreSQL 資料庫，
前端純 HTML/CSS/JS（由後端一併提供靜態檔案，單一服務即可部署）。

```
bookstore/
├── backend/            # Express API
│   ├── server.js
│   ├── db.js
│   ├── routes/
│   │   ├── books.js
│   │   └── orders.js
│   ├── package.json
│   └── .env.example
├── database/
│   ├── schema.sql       # 資料表結構
│   └── seed.sql         # 10 本書籍種子資料
└── public/              # 前端（靜態網頁）
    ├── index.html
    ├── css/style.css
    └── js/app.js
```

## 一、Supabase：建立資料庫

1. 到 [supabase.com](https://supabase.com) 建立新專案，記下資料庫密碼。
2. 進入專案 → **SQL Editor** → New query。
3. 貼上 `database/schema.sql` 全部內容並執行（建立 4 張資料表）。
4. 再貼上 `database/seed.sql` 並執行（寫入 10 本書籍）。
5. 到 **Project Settings → Database → Connection string**，複製 **URI**
   （建議使用 **Connection pooling** 的字串，port 通常是 `6543`，較適合 Render 這類無伺服器/多連線環境）。

   格式類似：
   ```
   postgresql://postgres.xxxxxxxx:[YOUR-PASSWORD]@aws-0-xxxx.pooler.supabase.com:6543/postgres
   ```

## 二、Render：部署後端（含前端靜態檔）

1. 把整個 `bookstore/` 專案推上 GitHub。
2. 到 [render.com](https://render.com) → **New → Web Service**，選擇該 repo。
3. 設定：
   - **Root Directory**：`backend`
   - **Build Command**：`npm install`
   - **Start Command**：`npm start`
   - **Instance Type**：Free 即可先行測試
4. 在 **Environment** 加入環境變數：
   - `DATABASE_URL` = 上一步從 Supabase 複製的連線字串
   - `PORT` 不用設，Render 會自動注入
5. 部署完成後，Render 會給一個網址，例如
   `https://folio-bookstore.onrender.com`，打開就能直接購書。

> 因為 `server.js` 同時提供 `public/` 靜態檔案並掛載 `/api/*`，
> 前後端只需要「一個」Render Web Service，不用額外部署前端。

## 三、本機開發測試

```bash
cd backend
npm install
cp .env.example .env      # 貼上你的 Supabase 連線字串
npm start
```

打開 http://localhost:3000 即可看到網站。

## 四、API 一覽

| Method | Path              | 說明                     |
|--------|-------------------|--------------------------|
| GET    | `/api/books`      | 取得所有書籍             |
| GET    | `/api/books/:id`  | 取得單一書籍             |
| POST   | `/api/orders`     | 建立訂單（含庫存檢查）   |
| GET    | `/api/orders/:id` | 查詢訂單明細             |

`POST /api/orders` 範例 body：
```json
{
  "customer": { "name": "王小明", "email": "test@example.com", "phone": "0912345678" },
  "items": [{ "book_id": 1, "qty": 2 }, { "book_id": 3, "qty": 1 }]
}
```

## 五、注意事項

- 訂單建立採用資料庫交易（transaction）+ `SELECT ... FOR UPDATE` 鎖定書籍列，
  避免多人同時搶購同一本書時發生超賣。
- `order_items.unit_price` 會記錄「下單當下」的售價，之後調整 `books.price`
  不會影響歷史訂單金額，符合你原始資料表設計的用意。
- 書籍封面使用 [Open Library Covers API](https://openlibrary.org/dev/docs/api/covers)
  依 ISBN 取得公開圖片；若某本書的圖片一時無法載入，前端會自動顯示以書名產生的
  備用書封，不會出現破圖。
- 若想把前端跟後端拆成兩個 Render 服務部署，只要在 `public/js/app.js` 把
  `fetch('/api/...')` 改成完整的後端網址，並確認 `server.js` 的 `cors()` 有開放即可。
