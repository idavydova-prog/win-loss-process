# One-Time Import to Google Sheets

## Step 1: Import the combined CSV

1. Open your G-Sheet: https://docs.google.com/spreadsheets/d/18lv8mvWU2wZD9gWZrXPVBHlxITTIkKEnMi3xDLF5OYE/
2. File > Import > Upload
3. Drag in: `All Reports - Combined.csv` (from this folder)
4. Choose "Insert new sheet"
5. Rename the new tab to "Raw Data"

## Step 2: Run the split script (one time)

1. Extensions > Apps Script
2. Delete any existing code in the editor
3. Paste the script below
4. Click Run (play button)
5. Authorize when prompted (first time only)
6. Wait ~10 seconds — 12 tabs will be created automatically

## The Script

```javascript
function splitIntoTabs() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const raw = ss.getSheetByName("Raw Data");
  if (!raw) {
    SpreadsheetApp.getUi().alert("No 'Raw Data' tab found. Import the CSV first and rename the tab to 'Raw Data'.");
    return;
  }

  const data = raw.getDataRange().getValues();
  const header = data[0].slice(1); // all columns except "Report"

  // Group rows by Report column
  const groups = {};
  for (let i = 1; i < data.length; i++) {
    const report = data[i][0];
    if (!report) continue;
    if (!groups[report]) groups[report] = [];
    groups[report].push(data[i].slice(1));
  }

  // Create one tab per report
  const tabOrder = [
    "Core May FY27", "Core Jun FY27", "Core Jul FY27",
    "MC May FY27", "MC Jun FY27", "MC Jul FY27",
    "CC May FY27", "CC Jun FY27", "CC Jul FY27",
    "Tableau May FY27", "Tableau Jun FY27", "Tableau Jul FY27"
  ];

  const tabLinks = {};
  const ssUrl = ss.getUrl();

  for (const name of tabOrder) {
    const rows = groups[name];
    if (!rows || rows.length === 0) continue;

    // Delete existing tab with same name if it exists
    let sheet = ss.getSheetByName(name);
    if (sheet) ss.deleteSheet(sheet);

    sheet = ss.insertSheet(name);
    sheet.appendRow(header);

    // Bold header
    sheet.getRange(1, 1, 1, header.length).setFontWeight("bold");

    // Write data
    if (rows.length > 0) {
      sheet.getRange(2, 1, rows.length, rows[0].length).setValues(rows);
    }

    // Auto-resize columns
    for (let c = 1; c <= header.length; c++) {
      sheet.autoResizeColumn(c);
    }

    // Save tab link
    tabLinks[name] = ssUrl + "#gid=" + sheet.getSheetId();
  }

  // Create Summary tab with links to each report
  let summary = ss.getSheetByName("Summary");
  if (summary) ss.deleteSheet(summary);
  summary = ss.insertSheet("Summary", 0); // insert as first tab

  // Header
  summary.getRange(1, 1, 1, 5).setValues([["Cloud", "May FY27", "Jun FY27", "Jul FY27", "Records"]]);
  summary.getRange(1, 1, 1, 5).setFontWeight("bold");

  // Data rows with hyperlinks
  const clouds = [
    { name: "Core", months: ["Core May FY27", "Core Jun FY27", "Core Jul FY27"] },
    { name: "Marketing Cloud", months: ["MC May FY27", "MC Jun FY27", "MC Jul FY27"] },
    { name: "Commerce Cloud", months: ["CC May FY27", "CC Jun FY27", "CC Jul FY27"] },
    { name: "Tableau", months: ["Tableau May FY27", "Tableau Jun FY27", "Tableau Jul FY27"] }
  ];

  let row = 2;
  for (const cloud of clouds) {
    summary.getRange(row, 1).setValue(cloud.name);
    let totalRecords = 0;
    for (let m = 0; m < 3; m++) {
      const tabName = cloud.months[m];
      const link = tabLinks[tabName];
      const count = groups[tabName] ? groups[tabName].length : 0;
      totalRecords += count;
      if (link) {
        const formula = '=HYPERLINK("' + link + '", "' + tabName + ' (' + count + ')")';
        summary.getRange(row, m + 2).setFormula(formula);
      } else {
        summary.getRange(row, m + 2).setValue(tabName + " (0)");
      }
    }
    summary.getRange(row, 5).setValue(totalRecords);
    row++;
  }

  // Total row
  summary.getRange(row, 1).setValue("TOTAL");
  summary.getRange(row, 1, 1, 5).setFontWeight("bold");
  summary.getRange(row, 5).setFormula("=SUM(E2:E5)");

  // Formatting
  summary.setColumnWidth(1, 150);
  summary.setColumnWidth(2, 200);
  summary.setColumnWidth(3, 200);
  summary.setColumnWidth(4, 200);
  summary.setColumnWidth(5, 80);

  SpreadsheetApp.getUi().alert("Done! 12 data tabs + Summary tab created. Copy links from the Summary tab.");
}
```

## Result

After running, you'll have 12 tabs:
- Core May FY27, Core Jun FY27, Core Jul FY27
- MC May FY27, MC Jun FY27, MC Jul FY27
- CC May FY27, CC Jun FY27, CC Jul FY27
- Tableau May FY27, Tableau Jun FY27, Tableau Jul FY27

Each tab has the same 13 columns (Account through Status). You can delete the "Raw Data" tab afterward if you want.
