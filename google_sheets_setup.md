# How to Setup Auto-Sync to Google Sheets

Because we are bypassing complicated Google Cloud authentications, we use a simple **Google Apps Script Webhook** that acts as a secure bridge directly into your spreadsheet. 

This takes 2 minutes and only needs to be done once per sheet.

### Step 1: Create the Webhook Script
1. Open a new or existing Google Sheet.
2. In the top menu, click **Extensions > Apps Script**.
3. Delete whatever code is there and paste the following 10 lines of code exactly:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSheet();
    var payload = JSON.parse(e.postData.contents);
    if (!payload.data || !payload.data.length) return ContentService.createTextOutput("No data");
    
    // Check if headers exist, if not create them
    if (sheet.getLastRow() === 0) {
      sheet.appendRow(["Name", "Website", "Domain", "Email"]);
    }
    
    // Append all rows
    var rows = payload.data.map(function(row) {
      return [row.Name, row.Website, row.Domain, row.Email];
    });
    
    // Write batch to sheet
    sheet.getRange(sheet.getLastRow() + 1, 1, rows.length, 4).setValues(rows);
    return ContentService.createTextOutput("Success");
  } catch (error) {
    return ContentService.createTextOutput(error.toString());
  }
}
```

### Step 2: Deploy and Get Your URL
1. Click the blue **Deploy** button (top right) and select **New deployment**.
2. Click the "**Select type**" gear icon ⚙️ and choose **Web app**.
3. Under **Description**, type anything (like "Auto Sync").
4. Under **Execute as**, select **Me**.
5. Under **Who has access**, select **Anyone**.
6. Click **Deploy**. (Google may ask you to authorize access to your own account. Click "Advanced" -> "Go to...").
7. Copy the **Web app URL** that ends in `/exec`.

### Step 3: Link to the Extension
1. Go back to Google Maps and open the `MapsScraper.net` extension popup.
2. Click the main Export Dropdown arrow.
3. Click the new **SYNC TO GOOGLE SHEETS** button.
4. Paste your Web app URL and click OK.

That's it! Every time the extension scrapes new leads, they will automatically pop up in your Google Sheet instantly!
