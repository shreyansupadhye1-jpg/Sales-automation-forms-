# Elemantra Requirements Form — Backend Setup
## Google Apps Script · One-time setup · ~10 minutes

---

## What this does
When a client submits the Requirements Form:
1. A new row is added to your Google Sheet automatically
2. You get an email notification with all their details
3. (Optional) WhatsApp notification via a webhook

---

## STEP 1 — Create your Google Sheet

1. Go to **sheets.google.com** and create a new spreadsheet
2. Name it: `Elemantra — Client Requirements`
3. Leave it open — you'll need the URL in Step 3

---

## STEP 2 — Open Apps Script

1. In your Google Sheet, click **Extensions → Apps Script**
2. Delete all the default code in the editor
3. Paste the entire script below and click **Save** (💾)

---

## STEP 3 — Paste this script

```javascript
// ─────────────────────────────────────────────────────────────
// Elemantra Requirements Form — Google Apps Script Backend
// Paste this entire script into Extensions > Apps Script
// ─────────────────────────────────────────────────────────────

// CONFIGURE THESE TWO VALUES:
const NOTIFICATION_EMAIL = "your@email.com";       // ← your email address
const SHEET_NAME         = "Responses";            // ← sheet tab name (create this tab)

// Column headers — must match exactly what the form sends
const HEADERS = [
  "Submitted At", "Name", "Phone", "Location", "Configuration",
  "Carpet Area (sqft)", "Budget", "Family Members", "Timeline / Possession",
  "Floor Plan Available", "General Scope", "Flooring Preference",
  "Finish Preference", "Living & Dining Items", "Living & Dining Notes",
  "Kitchen Layout", "Kitchen Type", "Kitchen Items", "Kitchen Notes",
  "Master Bedroom", "Bedroom 2", "Bedroom 3", "Bedroom 4 / Extra",
  "Bathroom Items", "Bathroom Notes", "Colour / Material Preferences",
  "Special Requirements", "Appliances", "BOQ Options Needed", "How They Heard"
];

function doPost(e) {
  try {
    const ss     = SpreadsheetApp.getActiveSpreadsheet();
    let   sheet  = ss.getSheetByName(SHEET_NAME);

    // Create the sheet + headers if it doesn't exist yet
    if (!sheet) {
      sheet = ss.insertSheet(SHEET_NAME);
      sheet.appendRow(HEADERS);
      // Style the header row
      sheet.getRange(1, 1, 1, HEADERS.length)
        .setBackground("#c8602a")
        .setFontColor("#ffffff")
        .setFontWeight("bold");
      sheet.setFrozenRows(1);
    }

    // Parse the submitted JSON data
    const data = JSON.parse(e.postData.contents);

    // Build a row in the correct column order
    const row = HEADERS.map(h => data[h] || "");
    sheet.appendRow(row);

    // Auto-resize columns for readability
    sheet.autoResizeColumns(1, HEADERS.length);

    // Send email notification
    sendEmailNotification(data);

    return ContentService
      .createTextOutput(JSON.stringify({ status: "success" }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ status: "error", message: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function sendEmailNotification(data) {
  const subject = `New Lead: ${data["Name"] || "Unknown"} — ${data["Configuration"] || ""} in ${data["Location"] || ""}`;

  const body = `
New requirements form submission received.

─────────────────────────────
CLIENT DETAILS
─────────────────────────────
Name:           ${data["Name"]}
Phone:          ${data["Phone"]}
Location:       ${data["Location"]}
Configuration:  ${data["Configuration"]}
Carpet Area:    ${data["Carpet Area (sqft)"]} sqft
Budget:         ${data["Budget"]}
Family:         ${data["Family Members"]}
Timeline:       ${data["Timeline / Possession"]}
Floor Plan:     ${data["Floor Plan Available"]}
Submitted:      ${data["Submitted At"]}

─────────────────────────────
SCOPE OF WORK
─────────────────────────────
General Scope:       ${data["General Scope"]}
Flooring:            ${data["Flooring Preference"]}
Finish:              ${data["Finish Preference"]}

─────────────────────────────
LIVING & DINING
─────────────────────────────
Items:  ${data["Living & Dining Items"]}
Notes:  ${data["Living & Dining Notes"]}

─────────────────────────────
KITCHEN
─────────────────────────────
Layout: ${data["Kitchen Layout"]}
Type:   ${data["Kitchen Type"]}
Items:  ${data["Kitchen Items"]}
Notes:  ${data["Kitchen Notes"]}

─────────────────────────────
BEDROOMS
─────────────────────────────
Master:     ${data["Master Bedroom"]}
Bedroom 2:  ${data["Bedroom 2"]}
Bedroom 3:  ${data["Bedroom 3"]}
Bedroom 4:  ${data["Bedroom 4 / Extra"]}

─────────────────────────────
BATHROOMS
─────────────────────────────
Items:  ${data["Bathroom Items"]}
Notes:  ${data["Bathroom Notes"]}

─────────────────────────────
PREFERENCES
─────────────────────────────
Colours / Materials:  ${data["Colour / Material Preferences"]}
Special Requirements: ${data["Special Requirements"]}
Appliances:           ${data["Appliances"]}
BOQ Options:          ${data["BOQ Options Needed"]}
Source:               ${data["How They Heard"]}
  `;

  GmailApp.sendEmail(NOTIFICATION_EMAIL, subject, body);
}
```

---

## STEP 4 — Deploy as Web App

1. Click **Deploy → New deployment**
2. Click the gear icon ⚙ next to "Type" → select **Web app**
3. Fill in:
   - Description: `Elemantra Form Backend`
   - Execute as: **Me**
   - Who has access: **Anyone** ← important, this allows the form to post data
4. Click **Deploy**
5. Click **Authorize access** → choose your Google account → Allow
6. **Copy the Web App URL** — it looks like:
   `https://script.google.com/macros/s/AKfycb.../exec`

---

## STEP 5 — Paste the URL into the form

1. Open `Elemantra_Requirements_Form.html` in a text editor
2. Find this line near the top of the `<script>` section:
   ```
   const APPS_SCRIPT_URL = "YOUR_APPS_SCRIPT_URL_HERE";
   ```
3. Replace `YOUR_APPS_SCRIPT_URL_HERE` with the URL you copied
4. Save the file and re-upload to GitHub Pages

---

## STEP 6 — Test it

1. Open your hosted form in a browser
2. Fill in a test submission (use your own name/number)
3. Check your Google Sheet — a new row should appear within seconds
4. Check your email — notification should arrive within 1–2 minutes

---

## OPTIONAL — WhatsApp notification

To also get a WhatsApp message on every submission:

1. Sign up free at **make.com** (formerly Integromat)
2. Create a scenario: **Webhook → WhatsApp Business** (or use Twilio)
3. Get your Make webhook URL
4. Add this line inside the `sendEmailNotification` function in the Apps Script:

```javascript
// Add this inside sendEmailNotification(), after GmailApp.sendEmail(...)
const MAKE_WEBHOOK = "YOUR_MAKE_WEBHOOK_URL";
const waMessage = `New Elemantra lead!\nName: ${data["Name"]}\nPhone: ${data["Phone"]}\nConfig: ${data["Configuration"]} in ${data["Location"]}\nBudget: ${data["Budget"]}`;
UrlFetchApp.fetch(MAKE_WEBHOOK, {
  method: "post",
  contentType: "application/json",
  payload: JSON.stringify({ message: waMessage })
});
```

---

## What each submission looks like in your Sheet

| Submitted At | Name | Phone | Location | Configuration | Budget | ... |
|---|---|---|---|---|---|---|
| 27/04/2026, 3:45 PM | Priya Sharma | 98XXX XXXXX | Andheri West | 3 BHK | ₹35–50 Lakhs | ... |

Each row = one client. Every field from the form maps to a column.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| No row appearing in Sheet | Check that "Who has access" is set to **Anyone** in the deployment |
| No email arriving | Check spam folder; confirm `NOTIFICATION_EMAIL` is correct |
| Error on submit | Re-deploy the script (Deploy → Manage deployments → Edit → New version) |
| Form says submitted but nothing in sheet | The `no-cors` fetch mode means the form can't confirm success — check the sheet directly |
