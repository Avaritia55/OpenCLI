# 台灣站點清單

本文檔記錄台灣本地站點的用途、opencli 支援狀態與 fallback 路徑。

---

## 社交媒體 / 論壇

### Dcard

- **用途**：台灣年輕世代主流社群，涵蓋生活、職場、感情、消費、時事討論
- **Adapter 狀態**：⏳ 規劃中（Phase 1）
- **Fallback 路徑**：
  ```bash
  opencli google "site:dcard.tw [查詢詞]"
  opencli browser open "https://www.dcard.tw/search?query=[查詢詞]"
  ```
- **限制**：部分內容需登入；反爬中等（需 cookie/session）

### PTT

- **用途**：台灣最大 BBS，科技、財經、政治、生活資訊密度高
- **Adapter 狀態**：⏳ 規劃中（Phase 2）
- **Fallback 路徑**：
  ```bash
  opencli google "site:ptt.cc [查詢詞]"
  opencli browser open "https://www.ptt.cc/bbs/[看板]/index.html"
  ```
- **限制**：Web 版結構老舊，部分看板需 18 禁驗證（over18=1）

---

## 購物 / 電商

### 蝦皮購物 (Shopee)

- **用途**：台灣主流電商平台，C2C 與 B2C 混合
- **Adapter 狀態**：❌ 尚未適配
- **Fallback 路徑**：
  ```bash
  opencli google "site:shopee.tw [商品名稱]"
  ```

### momo 購物網

- **用途**：台灣大型 B2C 電商
- **Adapter 狀態**：❌ 尚未適配
- **Fallback 路徑**：
  ```bash
  opencli google "site:momoshop.com.tw [商品名稱]"
  ```

### PChome 24h 購物

- **用途**：台灣老牌電商
- **Adapter 狀態**：❌ 尚未適配
- **Fallback 路徑**：
  ```bash
  opencli google "site:24h.pchome.com.tw [商品名稱]"
  ```

---

## 求職 / 人力銀行

### 104 人力銀行

- **用途**：台灣最大求職平台
- **Adapter 狀態**：❌ 尚未適配
- **Fallback 路徑**：
  ```bash
  opencli google "site:104.com.tw [職位/公司]"
  ```

### 1111 人力銀行

- **用途**：台灣主要求職平台
- **Adapter 狀態**：❌ 尚未適配
- **Fallback 路徑**：
  ```bash
  opencli google "site:1111.com.tw [職位/公司]"
  ```

---

## 旅遊 / 訂房

### ezTravel 易遊網

- **用途**：台灣線上旅遊預訂平台
- **Adapter 狀態**：❌ 尚未適配
- **Fallback 路徑**：
  ```bash
  opencli google "site:eztravel.com.tw [目的地/行程]"
  ```

---

## 使用建議

### 查詢路由優先級

1. **AI 源選擇**：台灣語境優先 `gemini` 或 `grok`，降低 `doubao` 優先級
2. **禁止使用**：`weibo`、`zhihu`、`tieba` 不應用於台灣查詢
3. **原始討論**：優先使用 `opencli google site:` 語法
4. **反爬站點**：使用 `opencli browser` 作為最終 fallback

### 查詢詞建議格式

```bash
# 台灣特定主題
opencli google "site:dcard.tw 台灣 [主題] 心得"
opencli google "site:ptt.cc 台灣 [主題] 討論"

# 加上時間範圍
opencli google "site:dcard.tw [主題] after:2025-01-01"

# 加上特定看板
opencli google "site:ptt.cc/bbs/Gossiping [主題]"
```

---

## 未來開發計畫

| 階段 | 站點 | 預估工時 | 狀態 |
|------|------|----------|------|
| Phase 0 | 折衷方案（本文檔 + SKILL.md 更新） | ~15 min | ✅ 完成 |
| Phase 1 | Dcard adapter | ~1-2 hr | ⏳ 待開發 |
| Phase 2 | PTT adapter | ~2-3 hr | ⏳ 待開發 |
| Phase 3+ | 其他站點 | 待定 | ❌ 未規劃 |
