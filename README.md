# Tutor Tracker

A lightweight, personal web app for tracking peer tutoring sessions, hours, earnings, and payment status — built with vanilla HTML, CSS, and JavaScript, using **Google Sheets as a persistent database**.

No frameworks. No backend server. No subscriptions. Just open the file in a browser and go.

<img width="1882" height="867" alt="image" src="https://github.com/user-attachments/assets/b910c3f1-37bc-4233-9b6d-1001d47989d3" />



<img width="1901" height="857" alt="image" src="https://github.com/user-attachments/assets/c67d4ef4-a6e6-412a-95ea-2650d1725fc9" />


---

## Why I Built This

As a peer tutor, I found myself losing track of:
- How many hours I'd tutored each week and month
- How much I was owed versus how much I'd actually received
- Which months had been paid and which hadn't (payments often come delayed)

Existing apps were overkill or required accounts and subscriptions. I wanted something minimal, fast to update, and visually clear — so I built it myself.

---

## Features

- **Quick session logging** — minimal inputs, date auto-fills, +/− hour stepper, one tap for attended/cancelled
- **Weekly summary** — hours logged this week vs your max cap, earnings so far this week
- **Monthly summary** — total hours and earnings for the current month
- **Trend charts** — weekly hours line chart (with max cap indicator), monthly earned vs received bar chart, stacked received vs owed chart
- **Payment tracking** — colour-coded tiles per month: green (paid), blue (partial), red (pending)
- **Overdue alert** — automatic warning banner if any month has been unpaid for 2+ months
- **Full session history** — filterable by month and attendance status, with delete support
- **Google Sheets backend** — all data lives in your own Google Sheet, accessible from any device, forever
- **Dark theme** — deep burgundy, crimson, and sky blue palette

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML, CSS, JavaScript (vanilla) |
| Storage | Google Sheets via Apps Script Web App |
| Charts | Chart.js |
| Fonts | Playfair Display, DM Sans (Google Fonts) |
| Hosting | GitHub Pages |

---

## Setup

This is a one-time setup that takes about 10 minutes.

### 1. Create your Google Sheet

Create a new Google Sheet and name the first tab `Sheet1`. Add these exact headers in row 1:

```
id | date | subject | hours | rate | attended | earned | semester
```

### 2. Add the Apps Script

In your sheet: **Extensions → Apps Script** → delete any existing code → paste the following → save:

```javascript
function doGet(e) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var callback = e.parameter.callback;
  var action = e.parameter.action;
  var payload = e.parameter.payload ? JSON.parse(e.parameter.payload) : null;
  var result;

  if (payload && payload.action === 'add') {
    var sheet = ss.getSheetByName('Sheet1');
    var r = payload.row;
    sheet.appendRow([r.id, r.date, r.subject, r.hours, r.rate, r.attended, r.earned, r.semester]);
    result = {ok: true};
  }
  else if (payload && payload.action === 'delete') {
    var sheet = ss.getSheetByName('Sheet1');
    var data = sheet.getDataRange().getValues();
    for (var i = 1; i < data.length; i++) {
      if (String(data[i][0]) === String(payload.id)) {
        sheet.deleteRow(i + 1);
        break;
      }
    }
    result = {ok: true};
  }
  else if (payload && payload.action === 'addPayment') {
    var ps = ss.getSheetByName('Payments');
    if (!ps) {
      ps = ss.insertSheet('Payments');
      ps.appendRow(['id', 'month', 'amount', 'note', 'date']);
    }
    var p = payload.row;
    ps.appendRow([p.id, p.month, p.amount, p.note, p.date]);
    result = {ok: true};
  }
  else if (action === 'getPayments') {
    var ps = ss.getSheetByName('Payments');
    if (!ps) {
      result = [];
    } else {
      var d = ps.getDataRange().getValues();
      var h = d[0];
      result = d.slice(1).map(function(r) {
        var o = {};
        h.forEach(function(hh, i) { o[hh] = r[i]; });
        return o;
      });
    }
  }
  else {
    var sheet = ss.getSheetByName('Sheet1');
    var data = sheet.getDataRange().getValues();
    var headers = data[0];
    result = data.slice(1).map(function(r) {
      var obj = {};
      headers.forEach(function(h, i) { obj[h] = r[i]; });
      return obj;
    });
  }

  var json = JSON.stringify(result);
  if (callback) {
    return ContentService.createTextOutput(callback + '(' + json + ')')
      .setMimeType(ContentService.MimeType.JAVASCRIPT);
  }
  return ContentService.createTextOutput(json)
    .setMimeType(ContentService.MimeType.JSON);
}
```

### 3. Deploy as a Web App

- Click **Deploy → New deployment**
- Type: **Web app**
- Execute as: **Me**
- Who has access: **Anyone**
- Click **Deploy** → authorize when prompted → copy the Web App URL

### 4. Host on GitHub Pages

- Upload `tutor-tracker.html` to a GitHub repository
- Go to **Settings → Pages → Deploy from main branch**
- Open your GitHub Pages URL in any browser

> **Important:** The app must be served from a URL (like GitHub Pages or a local server) — not opened directly as a local file — due to browser CORS restrictions on Google Apps Script requests.

### 5. Connect

- Open your GitHub Pages URL
- Paste your Apps Script Web App URL
- Fill in your subject, rate, max hours per week, and semester tag
- Click **Connect**

---

## Usage

### Logging a session
Go to **Log Session** — date auto-fills to today, subject and rate are pre-filled from settings. Use the +/− buttons to set hours, tap Yes/No for attendance, and click **Add session**. It saves directly to your Google Sheet.

### Recording a payment
Go to **Payments** — select the month, enter the amount received, add an optional note, and click **Record**. The payment tiles update immediately showing the new balance.

### Changing semester
When a new semester starts, go to **Settings** → update the **Subject** and **Semester tag** fields. All new sessions will be tagged with the new semester while old data remains intact.

---

## Data Privacy

This app does not track student names or any personal information about the people you tutor. Sessions are logged by date, subject, hours, and attendance only. All data is stored exclusively in your own private Google Sheet — nothing is sent to any third-party server.

---

## Customisation

All configuration lives in the **Settings** tab inside the app:

| Setting | Description |
|---|---|
| Apps Script URL | Your Google Apps Script Web App deployment URL |
| Default subject | Pre-fills the subject field when logging sessions |
| Default rate | Your hourly rate in QAR (or any currency) |
| Max hours/week | Used for the weekly cap progress bar and alert |
| Semester tag | Tags new sessions — change each semester |

---

## Limitations

- Requires an active internet connection to sync with Google Sheets
- Google Apps Script has a daily execution quota (20,000 reads/day for free accounts — far beyond any realistic tutoring use)
- Must be hosted on a URL rather than opened as a local file

---

## License

MIT — free to use, modify, and adapt for your own tutoring or freelance tracking needs.
