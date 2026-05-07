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

#### Column Table Columns

|Column             |Description                                               |
|-------------------|----------------------------------------------------------|
|⠿                  |Drag handle — drag rows to reorder output column positions|
|☐                  |Checkbox — select for bulk operations                     |
|#                  |Row number                                                |
|Source Column      |Original column name from the file (or formula label)     |
|Output Name        |What the column will be called in the exported file       |
|Transform / Formula|Operation to apply to the column values                   |
|Type               |`source`, `formula`, or `const`                           |
|Actions            |↑ ↓ move, Drop/Restore, ✕ delete (formula cols only)      |

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

### Row Filter

Collapse/expand from the **Row Filter** card:

|Mode                 |Behaviour                                                        |
|---------------------|-----------------------------------------------------------------|
|Keep all rows        |No filtering (default)                                           |
|Drop fully empty rows|Removes rows where all active source columns are blank           |
|Filter by value      |Keeps only rows where a chosen column contains a specified string|

-----

### Step 3 — Preview

Shows the first 10 rows with all mapping rules applied. Review column names, order, and values before committing the full export.

-----

### Step 4 — Export

Exports two files to the browser’s Downloads folder:

- `filename_mapped.xlsx` — Excel workbook
- `filename_mapped.csv` — comma-separated, UTF-8

-----

## Technical Notes

- **No external dependencies** — the entire application is one HTML file with no CDN calls, no npm packages, no server-side code
- **Runs fully in the browser** — data is never uploaded anywhere
- **XLSX parser** is built-in, handling ZIP decompression via the browser’s native `DecompressionStream` API (requires Edge 80+, Chrome 80+, Firefox 113+)
- **Formula engine** uses JavaScript’s `Function` constructor in strict mode with a sandboxed function scope — only the listed functions are available
- **Excel date serial detection** applies to both header rows (auto-converted on load) and data values (via the date transform)

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

Replace `excel_mapper.html` on the server with the new version. All users get the update immediately on next page load — no cache clearing needed for IIS static files served without aggressive cache headers.
