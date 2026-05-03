# Exercise Tracker / 運動記錄

---

## English

A lightweight, single-file web app for logging running and cycling workouts, with optional Google Sheets sync.

### Features

- **Two activity tabs** — Running (metres, pace min/km) and Bike (km, speed km/h)
- **Built-in stopwatch** — Start, pause, and reset; auto-fills the time fields on stop
- **Distance presets** — One-tap preset buttons for common distances
- **Google Sheets sync** — Automatically uploads each record on save; pull all records from Sheets on demand or on page load
- **Swipe to delete** — Swipe left on mobile to reveal the delete button; hover on desktop
- **No dependencies** — Pure HTML/CSS/JS, no build step, no frameworks

### Getting Started

```bash
python3 -m http.server 3001
# Open http://localhost:3001/running.html
```

### Google Sheets Integration

1. Create a Google Spreadsheet with two sheets named `running` and `bike`
2. Open Apps Script (Extensions → Apps Script), paste the code from the section below
3. Deploy as a web app (Execute as: Me, Who has access: Anyone)
4. Copy the deployment URL
5. Open the app, tap ⚙, and paste the URL

Once configured, every new record is sent to Sheets automatically, and the ↻ button syncs all records back from Sheets.

### Apps Script

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const ss = SpreadsheetApp.getActiveSpreadsheet();

    if (data.action === 'getAll') {
      return json({
        status:  'ok',
        running: parseRows(ss.getSheetByName('running').getDataRange().getValues(), 'running'),
        bike:    parseRows(ss.getSheetByName('bike').getDataRange().getValues(), 'bike')
      });
    }

    const sheet = ss.getSheetByName(data.tab === 'bike' ? 'bike' : 'running');

    if (data.action === 'delete') {
      const id = String(data.id);
      for (let i = sheet.getLastRow(); i >= 2; i--) {
        if (String(sheet.getRange(i, 1).getValue()) === id) { sheet.deleteRow(i); break; }
      }
      return json({ status: 'ok' });
    }

    const r = data.record;
    sheet.appendRow([r.id, r.date, r.distance, r.timeStr, r.metricStr, r.ms]);
    return json({ status: 'ok' });

  } catch(err) {
    return json({ status: 'error', message: err.toString() });
  }
}

function parseRows(rows, tab) {
  return rows
    .filter(row => row[0] && !isNaN(Number(row[0])))
    .map(row => ({
      id: Number(row[0]), date: formatDate(row[1]), distance: Number(row[2]),
      ms: Number(row[5]) || parseTimeStrToMs(String(row[3]), tab)
    }));
}

function formatDate(val) {
  if (val && typeof val.getFullYear === 'function')
    return Utilities.formatDate(val, Session.getScriptTimeZone(), "yyyy-MM-dd'T'HH:mm");
  return String(val);
}

function parseTimeStrToMs(timeStr, tab) {
  if (tab === 'bike') {
    const [hh, mm, ss] = timeStr.split(':');
    return ((parseInt(hh)||0)*3600 + (parseInt(mm)||0)*60 + (parseInt(ss)||0))*1000;
  }
  const [msPart, ccStr] = timeStr.split('.');
  const [mm, ss] = msPart.split(':');
  return ((parseInt(mm)||0)*60 + (parseInt(ss)||0))*1000 + (parseInt(ccStr)||0)*10;
}

function json(obj) {
  return ContentService.createTextOutput(JSON.stringify(obj)).setMimeType(ContentService.MimeType.JSON);
}
```

> After any script change, create a **new version** in Deploy → Manage Deployments.

---

## 中文

輕量級單檔網頁應用，用於記錄跑步與騎車運動，支援 Google 試算表同步。

### 功能特色

- **兩種運動頁籤** — 跑步（公尺、配速 min/km）與騎車（公里、速度 km/h）
- **內建碼表** — 開始、暫停、重置；停止後自動填入時間欄位
- **距離快速選擇** — 常用距離一鍵帶入
- **Google 試算表同步** — 新增記錄自動上傳；可隨時用 ↻ 從試算表拉回所有資料
- **滑動刪除** — 手機左滑顯示刪除按鈕；桌機滑鼠移入顯示
- **零依賴** — 純 HTML/CSS/JS，無需建置流程與框架

### 啟動方式

```bash
python3 -m http.server 3001
# 開啟 http://localhost:3001/running.html
```

### Google 試算表設定

1. 建立 Google 試算表，新增兩個分頁分別命名為 `running` 與 `bike`
2. 開啟 Apps Script（擴充功能 → Apps Script），貼上上方英文區的程式碼
3. 部署為網路應用程式（執行身分：我、存取權限：任何人）
4. 複製部署網址
5. 在 App 中點 ⚙，貼上網址後確認

設定完成後，每筆新記錄會自動寫入試算表；點 ↻ 可隨時從試算表同步回本地。

> 每次修改 script 後，需在「部署 → 管理部署」中建立**新版本**才會生效。
