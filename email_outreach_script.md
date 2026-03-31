# Google Sheets Email Outreach Setup

This single script will add a new button to your Google Sheet toolbar allowing you to mass-email your MapScraper leads with one click. It automatically creates a "Status" column to track who received the email, so you never send duplicates.

### Step 1: Add the Script
1. Open your Google Sheet where the leads are currently syncing.
2. In the top menu, click **Extensions > Apps Script**.
3. In the left sidebar, click the **+** (Add a file) button and choose **Script**, or just use the existing `Code.gs` file. (Do not overwrite your `doPost` webhook script! You can just paste this below it, or in a new file).
4. Paste the following script:

```javascript
function onOpen() {
  var ui = SpreadsheetApp.getUi();
  ui.createMenu('📬 Email Outreach')
      .addItem('Send Emails', 'sendEmails')
      .addToUi();
}

function sendEmails() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var dataRange = sheet.getDataRange();
  var data = dataRange.getValues();
  
  if (data.length < 2) {
    Browser.msgBox('Notice', 'No data found to process.', Browser.Buttons.OK);
    return;
  }
  
  // Find column indexes based on header names
  var headers = data[0];
  var emailIdx = headers.indexOf('Email');
  var nameIdx = headers.indexOf('Name');
  var statusIdx = headers.indexOf('Status');
  
  // If no email column, we can't do anything
  if (emailIdx === -1) {
    Browser.msgBox('Error', 'Could not find an "Email" column.', Browser.Buttons.OK);
    return;
  }
  
  // If no status column exists, add it to the end
  if (statusIdx === -1) {
    statusIdx = headers.length;
    sheet.getRange(1, statusIdx + 1).setValue('Status');
    sheet.getRange(1, statusIdx + 1).setFontWeight("bold");
    SpreadsheetApp.flush(); // Update the sheet immediately
  }

  // ============================================
  // ⚙️ EMAIL CONFIGURATION
  // ============================================
  var subject = "Important Information for {{Name}}";
  
  var bodyTemplate = "Hi {{Name}},\n\n" +
    "I was looking for businesses in your area and came across your profile. " +
    "I'd love to connect and share some exciting news.\n\n" +
    "Best regards,\n" +
    "Your Name";
  // ============================================

  var sentCount = 0;
  
  // Loop through all rows starting from row 2
  for (var i = 1; i < data.length; i++) {
    var row = data[i];
    var emailAddress = String(row[emailIdx]).trim();
    var name = nameIdx > -1 ? String(row[nameIdx]).trim() : "there";
    if (!name || name === "undefined") name = "there";
    var status = row[statusIdx];
    
    // Check if it has a valid email and hasn't been sent yet
    if (emailAddress && emailAddress.indexOf('@') !== -1 && status !== 'Sent') {
      
      // Personalize the template
      var personalizedSubject = subject.replace(/{{Name}}/g, name);
      var personalizedMessage = bodyTemplate.replace(/{{Name}}/g, name);
      
      try {
        MailApp.sendEmail(emailAddress, personalizedSubject, personalizedMessage);
        
        // Mark as "Sent" instantly so it saves the state
        sheet.getRange(i + 1, statusIdx + 1).setValue('Sent');
        SpreadsheetApp.flush(); 
        
        sentCount++;
      } catch (e) {
        // If an email address is invalid or bounces
        sheet.getRange(i + 1, statusIdx + 1).setValue('Failed: ' + e.message);
      }
    }
  }
  
  Browser.msgBox('Success', sentCount + ' emails were successfully sent!', Browser.Buttons.OK);
}
```

### Step 2: Configure Your Email Draft
Look at the section labeled **`⚙️ EMAIL CONFIGURATION`** inside the script. 
You can customize the **subject** and the **bodyTemplate** text to whatever you want. Be sure to keep `\n\n` where you want paragraph line breaks! Any time you put `{{Name}}`, the script will legally swap it with the scanned business name from your sheet.

### Step 3: Run It!
1. Save the script (click the floppy disk icon 💾 at the top).
2. Go completely out of the script editor back to your **Google Sheet**.
3. Refresh the Google Sheet page.
4. You will see a brand new menu at the top of Google Sheets called **`📬 Email Outreach`** (next to Help).
5. Simply click **📬 Email Outreach -> Send Emails**. 
6. (Google will ask you for permission specifically to "Send Emails as You" the very first time. Click Advanced -> Go to script).

As it runs, you will see a `Status` column magically appear, and "Sent" will drop in row-by-row as the emails fire off!
