# 金鍏電子 - 官網 AI 網頁助理整合專案 (Goldenway AI Web Assistant)

本專案是一個**獨立、無干擾、零門檻**的 AI 客服網頁助理管理庫。採用嵌入式 Iframe 方案，能完美繞過官網後台阻擋 JavaScript 腳本的安全限制，快速在各產品頁面部署專屬的「文章摘要與互動問答」助理。

---

## 📂 檔案目錄說明

-   **`燈具以租代買/燈具以租代買AI助理.html`**：針對「**燈具以租代買**」頁面設計的專用 AI 問答網頁。
    -   預載台糖照明汰換案例等核心摘要（Chrome Gemini 風格）。
    -   內建免金鑰問答庫與 Gemini 實時 API 連接模組。
-   **`README.md`**：本專案的使用、開發、部署與安全維護指南。

---

## 🚀 官網部署三步驟 (100% 成功)

### 第一步：上傳檔案至您的 GitHub Pages
1. 將本專案中的所有檔案（包含 `燈具以租代買` 資料夾）上傳到您在 GitHub 上建立的 **`goldenway-ai`** 儲存庫。
2. 進入該儲存庫的 **Settings（設定）➔ Pages**，將分支 (Branch) 設為 **`main`** (或 `master`) 並儲存。
3. 稍等片刻，您將獲得一個公開網址，例如：
   `https://[您的GitHub帳號].github.io/goldenway-ai/燈具以租代買/燈具以租代買AI助理.html`

### 第二步：在官網編輯器中嵌入 Iframe
在您的官網產品頁面（如 `product_d.php`）後台，切換到 **HTML / 原始碼模式**，並在需要呈現 AI 的位置貼上：
```html
<!-- 嵌入金鍏 LED 智慧 AI 諮詢與總結視窗 -->
<div id="gw-ai-section" style="margin: 30px 0; width: 100%;">
  <iframe 
    src="https://[您的GitHub帳號].github.io/goldenway-ai/燈具以租代買/燈具以租代買AI助理.html" 
    width="100%" 
    height="650" 
    style="border: none; border-radius: 16px; box-shadow: 0 10px 30px rgba(0,0,0,0.25);"
    scrolling="no">
  </iframe>
</div>
```

### 第三步：設置錨點跳轉按鈕 (選填)
若網頁上方已有「AI 總結 & 諮詢」按鈕，請將該按鈕的 `href` 屬性直接修改為 **`#gw-ai-section`**：
```html
<a href="#gw-ai-section" class="your-button-style">
    ✨ AI 總結 & 諮詢
</a>
```
點擊按鈕後，頁面會自動平滑滾動至下方的 AI 嵌入小視窗，實現完美的跳轉交互。

---

## 🛠️ 如何為新產品建立 AI 網頁（例如太陽能板介紹頁）

當未來有其他公司產品需要 AI 網頁助理時，您可以依照下列步驟快速自我複製：

1.  **新建與命名資料夾**：
    在專案根目錄下建立一個新資料夾（例如：`太陽能板`），並複製 `燈具以租代買/燈具以租代買AI助理.html` 檔案到該資料夾中，並改名（例如：`太陽能板/太陽能板AI助理.html`）。
2.  **修改摘要卡片**：
    以代碼或文字編輯器打開該 HTML 檔案，尋找 `getSummaryHTML()` 函數，並修改裡面的 HTML 標籤內容（核心一句話、關鍵要點、重要數據、行動建議）。
3.  **修改本地 FAQ 資料庫**：
    尋找 `FAQ_DATABASE` 對照表，更新適合新產品的關鍵字、問題標題及回答內容。
4.  **上傳與嵌入**：
    將新的檔案與資料夾上傳到 GitHub，並在官網新的產品頁面中，將 `<iframe>` 的 `src` 網址指向新的檔案網址即可（例如：`https://[您的GitHub帳號].github.io/goldenway-ai/太陽能板/太陽能板AI助理.html`）。

---

## 🔒 實時 API 金鑰安全代理設定 (PHP Proxy)

若您開啟了 Live Gemini API 智慧對談模式，且希望隱藏 API 金鑰防止他人竊取，請請網站管理工程師在主網域伺服器放置一個名為 **`ai-proxy.php`** 的檔案，程式碼如下：

```php
<?php
// ai-proxy.php
header('Content-Type: application/json; charset=utf-8');

// 🔐 請於此處貼上您的私密 Google Gemini API 金鑰
$apiKey = "您的_GEMINI_API_金鑰"; 

$inputJSON = file_get_contents('php://input');
$input = json_decode($inputJSON, true);

if (!$input) {
    http_response_code(400);
    echo json_encode(["error" => ["message" => "無效的請求內容"]]);
    exit;
}

$url = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=" . $apiKey;

$ch = curl_init($url);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, ['Content-Type: application/json']);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($input));

$response = curl_exec($ch);
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);

if (curl_errno($ch)) {
    http_response_code(500);
    echo json_encode(["error" => ["message" => curl_error($ch)]]);
    curl_close($ch);
    exit;
}

curl_close($ch);
http_response_code($httpCode);
echo $response;
?>
```

### 連接 PHP 代理：
在您的 `gw-ai-chat-[產品名].html` 檔案中，找到第 572 行的 `queryGeminiAPI`：
```javascript
const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${apiKey}`;
```
替換為呼叫您的伺服器代理路由：
```javascript
const url = `https://您的網域.com/ai-proxy.php`;
```
即可保證 API Key 的絕對安全。
