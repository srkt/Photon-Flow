# Excel Column Mapper

A self-contained, browser-based ETL tool for cleaning and mapping Excel/CSV files to database destination tables. No installation, no internet connection, no external dependencies — a single HTML file.

-----

## Deployment

### IIS (Windows Server) — Recommended

1. Copy `excel_mapper.html` to `C:\inetpub\wwwroot\`
1. Open browser and navigate to `http://your-server/excel_mapper.html`

### SharePoint

1. Upload `excel_mapper.html` to a SharePoint Document Library
1. Share the direct link with your team

### Local / Network Drive

Double-click the file in any modern browser (Edge, Chrome, Firefox).

> **No internet required.** All processing happens in the browser. Files never leave the user’s machine.

-----

## How It Works

The tool has four steps shown in the header: **Upload → Map → Preview → Export**

### Step 1 — Upload

Drop or browse for a `.xlsx`, `.xls`, or `.csv` file. The first row is treated as headers.

> **Excel date serial headers** are automatically detected and converted. For example, a column header stored as `44927` will appear as `2023-01-01`.

-----

### Step 2 — Map

This is where all the work happens. Each row in the mapping table represents one output column.

#### Column Table

|Column             |Description                                                 |
|-------------------|------------------------------------------------------------|
|⠿                  |Drag handle — drag rows to reorder output column positions  |
|☐                  |Checkbox — select for bulk operations                       |
|#                  |Row number                                                  |
|Source Column      |Original column name from the file (or formula/script label)|
|Output Name        |What the column will be called in the exported file         |
|Transform / Formula|Operation to apply to the column values                     |
|Type               |`source`, `formula`, or `const`                             |
|Actions            |↑ ↓ move, Drop/Restore, ✕ delete (formula cols only)        |

#### Per-Column Transforms

|Transform           |What it does                                                       |
|--------------------|-------------------------------------------------------------------|
|No change           |Passes value through as-is                                         |
|Trim spaces         |Strips leading and trailing whitespace                             |
|→ Date (YYYY-MM-DD) |Parses and reformats dates to ISO format                           |
|→ Date (MM/DD/YYYY) |Parses and reformats dates to US format                            |
|→ UPPERCASE         |Converts text to uppercase                                         |
|→ lowercase         |Converts text to lowercase                                         |
|→ Title Case        |Capitalises first letter of each word                              |
|→ Number            |Strips currency symbols and converts to numeric                    |
|Remove special chars|Keeps only letters, numbers, spaces, `-_.,`                        |
|→ snake_case        |Converts to lowercase with underscores (useful for DB column names)|

#### Reordering Columns

- **Drag and drop** rows to any position
- Use **↑ ↓ arrow buttons** for precise one-step moves
- Output file columns will appear in the order shown

-----

### Bulk Operations

When one or more checkboxes are ticked, a bulk action bar appears at the top:

|Action           |Description                                                       |
|-----------------|------------------------------------------------------------------|
|Drop selected    |Marks all selected columns as dropped (excluded from output)      |
|Apply transform  |Sets the same transform on all selected columns at once           |
|Add prefix/suffix|Prepends or appends text to the output names of selected columns  |
|Find & replace   |Searches all output names across the whole sheet and replaces text|
|✕ Clear          |Deselects all                                                     |

**Bulk Rename** (top toolbar button) lets you run find/replace, add a prefix, and add a suffix all in one step — with scope set to “All columns” or “Selected only”.

-----

### Adding Columns

Click **＋ Add Column** to open the column builder. Three modes:

#### Formula

Write an Excel-style expression using column references in square brackets.

```
[FirstName] & " " & [LastName]
UPPER(TRIM([email]))
IF([Amount]>1000, "High", "Low")
ROUND([Price] * 1.15, 2)
LEFT([ProductCode], 3)
SUBSTITUTE([Phone], "-", "")
```

Click any **column pill** in the dialog to insert the reference at the cursor position. A live validator shows whether the formula is valid and a sample result.

**Supported functions:**

|Function    |Syntax                            |Description                               |
|------------|----------------------------------|------------------------------------------|
|`LEFT`      |`LEFT([col], n)`                  |First n characters                        |
|`RIGHT`     |`RIGHT([col], n)`                 |Last n characters                         |
|`MID`       |`MID([col], start, n)`            |n characters from position start (1-based)|
|`LEN`       |`LEN([col])`                      |Character count                           |
|`TRIM`      |`TRIM([col])`                     |Remove leading/trailing spaces            |
|`UPPER`     |`UPPER([col])`                    |Uppercase                                 |
|`LOWER`     |`LOWER([col])`                    |Lowercase                                 |
|`PROPER`    |`PROPER([col])`                   |Title case                                |
|`SUBSTITUTE`|`SUBSTITUTE([col], "old", "new")` |Replace all occurrences                   |
|`REPLACE`   |`REPLACE([col], start, n, "new")` |Replace n chars at position               |
|`ROUND`     |`ROUND([col], decimals)`          |Round to decimal places                   |
|`FLOOR`     |`FLOOR([col], step)`              |Round down to nearest step                |
|`CEILING`   |`CEILING([col], step)`            |Round up to nearest step                  |
|`ABS`       |`ABS([col])`                      |Absolute value                            |
|`MOD`       |`MOD([col], divisor)`             |Remainder                                 |
|`TEXT`      |`TEXT([col], "0.00")`             |Format number as string                   |
|`IF`        |`IF(condition, trueVal, falseVal)`|Conditional                               |
|`ISNULL`    |`ISNULL([col])`                   |True if empty or null                     |
|`COALESCE`  |`COALESCE([col1], [col2])`        |First non-empty value                     |
|`CONCAT`    |`CONCAT([a], " ", [b])`           |Join multiple values                      |
|`TODAY`     |`TODAY()`                         |Today’s date as YYYY-MM-DD                |

Use `&` or `+` to concatenate strings (both work, same as Excel).

#### Constant

Adds a fixed value to every row. Useful for tagging rows with a source system, client name, load date, etc.

```
CLIENT_A
2024
PENDING
```

#### Split Column

Splits an existing column by a delimiter and extracts one part.

- Pick the **source column**
- Enter the **delimiter** (e.g. `,` or `-` or type `space`)
- Enter the **part number** (1 = first part, 2 = second, etc.)

Example: splitting `[FullAddress]` by `,` with part `2` extracts the city from `"123 Main St, Springfield, IL"`.

-----

### Script Editor

For complex transformations that go beyond formulas, the toolbar has two script buttons that open a full-screen JavaScript editor with a console, a reference sidebar, and a saved script library.

#### ⌨ Column Script

Runs a JavaScript block that modifies the column mapping table in bulk. Use this when you need to rename, reformat, drop, or retransform many columns at once based on logic that would be tedious to apply manually.

The script has direct access to `colDefs` (the mapping array) and `srcHeaders`. Modify `colDefs` in place — changes are applied to the mapping table when you click **Apply**.

**Available colDef fields:**

|Field        |Type   |Description                               |
|-------------|-------|------------------------------------------|
|`d.srcCol`   |string |Original source column name               |
|`d.outName`  |string |Output column name (rename this)          |
|`d.transform`|string |Transform key (see transform list above)  |
|`d.dropped`  |boolean|Set to `true` to exclude from output      |
|`d.formula`  |string |Formula expression (for formula-type cols)|
|`d.type`     |string |`"source"`, `"formula"`, or `"const"`     |

**Examples:**

```js
// Rename all date headers to P_YYYYMMDD format
colDefs.forEach(d => {
  if (/^\d{4}-\d{2}-\d{2}$/.test(d.srcCol)) {
    d.outName = 'P_' + d.srcCol.replace(/-/g, '');
  }
});
```

```js
// Add dest_ prefix to all non-dropped columns
colDefs.forEach(d => {
  if (!d.dropped) d.outName = 'dest_' + d.outName;
});
```

```js
// Drop any column whose name contains "tmp" or "test"
colDefs.forEach(d => {
  if (/tmp|test/i.test(d.srcCol)) d.dropped = true;
});
```

```js
// Set trim transform on all source columns
colDefs.forEach(d => {
  if (d.type === 'source') d.transform = 'trim';
});
```

```js
// Rename columns from a lookup map
const map = {
  'cust_id':    'CustomerID',
  'ord_dt':     'OrderDate',
  'prod_code':  'ProductCode',
};
colDefs.forEach(d => {
  if (map[d.srcCol]) d.outName = map[d.srcCol];
});
```

-----

#### ⌨ Row Script

Runs a `transformRow(row)` function against every single row during Preview and Export. Use this for complex per-row logic: conditional value changes, multi-column derivations, date formatting, row filtering, or anything that the formula engine can’t express cleanly.

**Signature:**

```js
function transformRow(row) {
  const out = Object.assign({}, row); // clone — don't mutate row directly
  // ... modify out ...
  return out;       // return modified row to keep it
  // return null;   // return null to DROP the row from output
}
```

The function receives `row` as a plain object keyed by **source column names**. It must return either a modified row object or `null`/`false` to drop the row.

**Built-in helper functions** (available inside the script without importing):

|Helper           |Description                                    |
|-----------------|-----------------------------------------------|
|`trim(s)`        |Strip whitespace from string                   |
|`upper(s)`       |Uppercase                                      |
|`lower(s)`       |Lowercase                                      |
|`left(s, n)`     |First n characters                             |
|`right(s, n)`    |Last n characters                              |
|`isBlank(v)`     |True if null, undefined, or empty string       |
|`num(v)`         |Parse to number, stripping currency symbols    |
|`toDate(v)`      |Parse to JS Date (handles Excel serial numbers)|
|`fmtDate(v, fmt)`|Parse and format a date value                  |

**`fmtDate` format strings:**

|Format string |Example output|
|--------------|--------------|
|`'YYYY-MM-DD'`|`2024-03-15`  |
|`'YYYYMMDD'`  |`20240315`    |
|`'P_YYYYMMDD'`|`P_20240315`  |
|`'MM/DD/YYYY'`|`03/15/2024`  |

**Examples:**

```js
// Format all date columns as P_YYYYMMDD
function transformRow(row) {
  const out = Object.assign({}, row);
  for (const key of Object.keys(out)) {
    const d = toDate(out[key]);
    if (d) out[key] = fmtDate(out[key], 'P_YYYYMMDD');
  }
  return out;
}
```

```js
// Drop rows where Amount is zero or blank
function transformRow(row) {
  if (isBlank(row.Amount) || num(row.Amount) === 0) return null;
  return Object.assign({}, row);
}
```

```js
// Combine first + last name, clean phone
function transformRow(row) {
  const out = Object.assign({}, row);
  out.FullName = trim(row.FirstName + ' ' + row.LastName);
  out.Phone = out.Phone.replace(/[^\d]/g, '');
  return out;
}
```

```js
// Flag high-value orders, drop cancelled ones
function transformRow(row) {
  if (row.Status === 'CANCELLED') return null;
  const out = Object.assign({}, row);
  out.ValueBand = num(row.Amount) >= 10000 ? 'HIGH' : 'STANDARD';
  return out;
}
```

> **Note:** The Row Script runs **after** all column mapping and transforms are applied. If you need to work with the original source column names, use the Row Script. If you need to work with the output column names, be aware that renaming happens in the mapping table first.

-----

#### Script Editor UI

|Button               |Action                                                                                                                                      |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
|**▶ Test**           |Runs the script against the first 3 rows and prints results to the console — does not modify any data                                       |
|**✓ Apply**          |Column Script: applies changes to the mapping table immediately. Row Script: activates the script for all subsequent Preview and Export runs|
|**💾 Save**           |Prompts for a name and saves the script to the browser’s local storage                                                                      |
|**Load saved script**|Dropdown of previously saved scripts (filtered by mode — column scripts only appear in Column mode)                                         |
|**Delete**           |Removes the currently selected saved script from storage                                                                                    |

Scripts are saved per browser via `localStorage`. They persist across sessions but are browser-local — they are not stored in the HTML file itself. To share scripts between users, copy and paste the code manually or embed common scripts as defaults in the HTML.

Tab key inserts two spaces. Escape closes the editor.

-----

### Row Filter

Collapse/expand from the **Row Filter** card:

|Mode                 |Behaviour                                                        |
|---------------------|-----------------------------------------------------------------|
|Keep all rows        |No filtering (default)                                           |
|Drop fully empty rows|Removes rows where all active source columns are blank           |
|Filter by value      |Keeps only rows where a chosen column contains a specified string|

-----

### Step 3 — Preview

Shows the first 10 rows with all mapping rules applied — including any active Row Script. Review column names, order, and values before committing the full export.

-----

### Step 4 — Export

Exports two files to the browser’s Downloads folder:

- `filename_mapped.xlsx` — Excel workbook
- `filename_mapped.csv` — comma-separated, UTF-8

-----

## Processing Order

When exporting, operations are applied in this order:

1. **Row filter** — drop rows that don’t match the filter condition
1. **Column mapping** — apply per-column transforms and evaluate formula/const columns
1. **Column reordering** — output columns appear in the order shown in the mapping table
1. **Row Script** — `transformRow()` runs on each row after mapping (if a row script is active)

-----

## Technical Notes

- **No external dependencies** — the entire application is one HTML file with no CDN calls, no npm packages, no server-side code
- **Runs fully in the browser** — data is never uploaded anywhere
- **XLSX parser** is built-in, handling ZIP decompression via the browser’s native `DecompressionStream` API (requires Edge 80+, Chrome 80+, Firefox 113+)
- **Formula engine** uses JavaScript’s `Function` constructor in strict mode with a sandboxed function scope — only the listed functions are available
- **Script engine** similarly sandboxes user code — scripts run in strict mode with no access to the DOM or browser APIs beyond what is explicitly provided
- **Script library** is stored in `localStorage` under the key `colmapper_scripts` — it is browser-local and not shared between users or machines
- **Excel date serial detection** applies to header rows (auto-converted on load) and data values (via the date transform or `fmtDate` helper in scripts)

-----

## Browser Compatibility

|Browser        |Minimum Version|
|---------------|---------------|
|Microsoft Edge |80+            |
|Google Chrome  |80+            |
|Mozilla Firefox|113+           |

Internet Explorer is not supported.

-----

## Updating the Tool

Replace `excel_mapper.html` on the server with the new version. All users get the update immediately on next page load.

> Saved scripts are stored in each user’s browser `localStorage` and are **not** affected by updating the HTML file — they will still be there after an update.
