# HIS Tools - Purchase Target Monitor (貨品監控助手)

> 版本：v1.03 | 更新日期：2026-10-11

## 簡介

HIS Tools 貨品監控助手是一個手機優先的網頁工具，用於監控心儀貨品的價格變化。數據源自 Price.com.hk 及 Jumbo Computer，由 J05R 定時任務每日 20:00 自動爬取並更新至飛書多維表格，再導出為 JSON 供 HTML 動態讀取。

**線上體驗：** https://hkharrycheng.github.io/purchase-target/

---

## ADMIN 使用說明

### 日常運作

1. **J05R 定時任務**：每日 20:00 HKT 自動執行，爬取監控貨品的最新報價
2. **數據存儲**：報價數據存儲在飛書多維表格「詳細報價」表
3. **JSON 導出**：執行 `gen_products_json.py` 將多維表格數據導出為 `products.json`
4. **上傳 GitHub**：將 `products.json` 及 `index.html` 上傳至本 repository
5. **GitHub Pages**：頁面自動更新，用戶可實時查看最新價格

### 新增監控貨品

1. 在飛書多維表格「監控貨品」表新增一行
2. 填寫：貨品名稱、版本、官方售價、官方 Release 日期、Major Source、數目（爬幾多個報價）
3. 上傳 Product Shot 圖片
4. 填寫官方網頁 SPEC 鏈接
5. 設定貨品狀態為「正在監控」
6. 呼叫 J05R 執行一次爬取
7. 重新生成 products.json 並上傳

### 調整跌價目標

在飛書多維表格「常數設定」表的 J05R Parameter 中修改：
- `行貨跌價target`：行貨目標跌價百分比（預設 0.4 = 40%）
- `水貨跌價target`：水貨目標跌價百分比（預設 0.5 = 50%）

---

## Logic Flow

```
用戶打開 HTML
    ↓
fetch products.json (含 targets + products + quotes)
    ↓
渲染統計欄（監控貨品數量、報價總數）
    ↓
渲染產品圖庫（每張卡片：圖片、名稱、版本、來源、行/水貨價格、跌價%）
    ↓
跌價顏色邏輯：
  - 0% → 紅色「持平」
  - 0% ~ target → 紅→黃→綠漸變
  - ≥ target → 綠色（達標）
  - 行貨用 targets.hongDrop，水貨用 targets.waterDrop
    ↓
用戶點擊產品卡片 → 彈出 Modal 詳情
    ↓
顯示：官方售價、最新行/水貨價格、跌價%、推出時間、所有商家報價詳情
    ↓
每個商家：名稱、行/水貨價格、詳細介紹、地址（可點擊打開 Google Map）、來源鏈接
```

---

## Business Flow

```
J05R 定時任務 (每日20:00)
    ↓
讀取「監控貨品」表 → 取得所有「正在監控」的產品
    ↓
對每個產品，按「數目」字段爬取 Price.com.hk 報價
    ↓
解析：商家名稱、行貨價、水貨價、地址、詳細介紹、來源鏈接
    ↓
寫入「詳細報價」表（關聯對應產品）
    ↓
檢查並補充：Product Shot、官方售價、官方網頁SPEC（如為空）
    ↓
完成
    ↓
ADHOC 呼叫 J00 → 生成 products.json → 上傳 GitHub → HTML 自動更新
```

---

## JSON Structure

### products.json

```json
{
  "lastUpdate": "2026-10-09 14:20",
  "targets": {
    "hongDrop": 0.4,
    "waterDrop": 0.5
  },
  "products": [
    {
      "name": "Samsung Galaxy Tab S11 11吋 5G",
      "version": "12+256GB",
      "image": "assets/samsung_tab_s11.jpg",
      "officialPrice": 8988,
      "releaseDate": "2025-09-04",
      "months": "13 個月",
      "hongPrice": "$6150 - $7499",
      "waterPrice": "$8299 - $8299",
      "hongDrop": 0.241,
      "waterDrop": 0.077,
      "types": ["行貨", "水貨"],
      "source": "https://www.price.com.hk/product.php?p=672318",
      "spec": "https://www.samsung.com/hk/tablets/..."
    }
  ],
  "quotes": {
    "Samsung Galaxy Tab S11 11吋 5G": [
      {
        "merchant": "悅訊科技",
        "hongPrice": 6150,
        "waterPrice": null,
        "detail": "優質商戶 信心保證...",
        "address": "九龍旺角亞皆老街83號先達廣場地下G106號鋪",
        "source": "https://www.price.com.hk/starshop.php?s=XXX"
      }
    ]
  }
}
```

### 字段說明

| 字段 | 類型 | 說明 |
|------|------|------|
| lastUpdate | string | 最後更新時間 |
| targets.hongDrop | number | 行貨跌價目標（0.4 = 40%） |
| targets.waterDrop | number | 水貨跌價目標（0.5 = 50%） |
| products[].name | string | 貨品名稱 |
| products[].version | string | 版本/規格 |
| products[].image | string | 產品圖片路徑 |
| products[].officialPrice | number | 官方售價（HKD） |
| products[].releaseDate | string | 官方 Release 日期 |
| products[].months | string | 推出了多久 |
| products[].hongPrice | string | 最新行貨報價範圍 |
| products[].waterPrice | string | 最新水貨報價範圍 |
| products[].hongDrop | number | 行貨跌價百分比（0.241 = 24.1%） |
| products[].waterDrop | number | 水貨跌價百分比 |
| products[].types | array | 貨品類型（行貨/水貨） |
| products[].source | string | Major Source 鏈接 |
| products[].spec | string | 官方網頁 SPEC 鏈接 |
| quotes[productName] | array | 該產品的所有商家報價 |
| quotes[][].merchant | string | 商家名稱 |
| quotes[][].hongPrice | number | 行貨價格（null 表示無） |
| quotes[][].waterPrice | number | 水貨價格（null 表示無） |
| quotes[][].detail | string | 商家詳細介紹 |
| quotes[][].address | string | 商家地址 |
| quotes[][].source | string | 報價來源鏈接 |

---

## 飛書多維表格結構

### 監控貨品表 (tblsIyHuFWt0Wtx9)

| 字段 | 類型 | 說明 |
|------|------|------|
| 貨品名稱 | text | 主字段 |
| 貨品狀態 | select | 正在監控 / 已購買 / 已停止 |
| Product Shot | attachment | 產品圖片 |
| 數目 | text | 爬取報價數量 |
| 行貨跌價% | formula | 自動計算 |
| 最新水貨報價 | formula | 自動計算 |
| 官方網頁SPEC | text | 官方規格頁鏈接 |
| Major Source | text | 主要報價來源 |
| 水貨跌價% | formula | 自動計算 |
| 官方 Release 日期 | datetime | 發布日期 |
| 最新行貨報價 | formula | 自動計算 |
| 推出月份 | formula | 自動計算推出多久 |
| 官方售價 | number | 最初推出市場 HKD 價 |
| 監察類型 | select | 行貨 / 水貨 / 兩者 |
| 版本 | text | 產品版本/規格 |

### 詳細報價表 (tblBNpg1NKqtC4GL)

| 字段 | 類型 | 說明 |
|------|------|------|
| 監控時間 | datetime | 爬取時間 |
| 關聯記錄ID | formula | 匹配橋樑 |
| 來源鏈接 | text | 報價頁面鏈接 |
| 報價商家名 | text | 商家名稱 |
| 水貨價 | number | 水貨價格 |
| 詳細 | text | 商家介紹 |
| 行貨價 | number | 行貨價格 |
| 地址 | text | 商家地址 |
| 产品型号 | link | 關聯監控貨品 |

---

## Version Control

| 版本 | 日期 | 改動 |
|------|------|------|
| v1.00 | 2026-10-09 | 初始版本。產品圖庫展示、Modal 詳情、行/水貨跌價顏色（雙標準）、Major Source 鏈接、官方 SPEC 鏈接、動態 JSON 讀取 |
| v1.01 | 2026-10-11 | UI優化，跌價顏色邏輯調整 |
| v1.02 | 2026-10-11 | 增加日期篩選功能（下拉選單），每條報價顯示日期時間，因J05R改為追加模式保留歷史數據 |
| v1.03 | 2026-10-11 | 日期篩選改為Slider滑桿，用家可拉動選擇「從邊日開始」顯示歷史報價，含30日測試數據 |

---

## 相關連結

- **HIS Tools 主頁：** https://hkharrycheng.github.io/his-tools/
- **飛書多維表格：** https://www.doubao.com/base/An0abQoUBalSqJsG66ucDzT3ncg
- **GitHub Repository：** https://github.com/hkharrycheng/purchase-target
- **數據來源：** Price.com.hk / Jumbo Computer

---

## License

MIT License - HIS Tools © 2026
