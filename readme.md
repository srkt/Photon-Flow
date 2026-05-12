# Photon-Flow

Photon-Flow is a web-based ETL tool built to make data preparation simple and super fast.

The project is moving beyond a single Excel utility into a broader extraction, transformation, and loading workflow. The current working step is the Excel Transformer module, which handles browser-based Excel and CSV reshaping.

## Excel Transformer

### Current ETL Step

Excel Transformer is the first transformation step in Photon-Flow. It can map columns, filter rows, pivot, unpivot, aggregate, preview results, export output files, and save reusable transformation configs.

The Excel Transformer interface should be treated as a focused step inside the larger Photon-Flow workflow. Where possible, its sections should be minimizable so users can collapse completed areas and see only the active steps, similar to an accordion.

The app runs from one file: `excel-transformer.html`.

No server is required for normal use. File parsing, transformations, preview, exports, and saved configs run in the browser.

---

## Quick Start

1. Open `excel-transformer.html` in a modern browser.
2. Upload an `.xlsx`, `.xls`, or `.csv` file.
3. Choose the operation type:
   - `Regular transform`
   - `Pivot`
   - `Unpivot`
   - `Aggregate`
4. Configure the visible settings for that operation.
5. Preview the result.
6. Export the output file.
7. After export, choose whether to save or download the transformation config.

Use `Reupload File` at the top of the app to start over with a new source file without returning to the initial screen.

As Photon-Flow grows, this Excel Transformer flow should remain one compact step in the larger ETL process rather than the whole product surface.

---

## Deployment

### Local or Network Drive

Double-click `excel-transformer.html` and open it in Edge, Chrome, or Firefox.

### IIS

1. Copy `excel-transformer.html` to `C:\inetpub\wwwroot\`.
2. Open `http://your-server/excel-transformer.html`.

### SharePoint

1. Upload `excel-transformer.html` to a SharePoint document library.
2. Share the direct file link with users.

All processing happens in the browser. Source files are not uploaded to a backend.

---

## Config JSON Workflow

Transformation settings can be saved as a config JSON so the same rules can be reused later.

Supported config actions:

- Save config to browser `localStorage`.
- Download config as a `.json` file.
- Load a saved local config.
- Drop or browse for a config JSON file.
- Validate the config against the uploaded data file before applying it.

The app checks whether referenced source columns exist in the uploaded file. If there are mismatches, it reports the discrepancies before applying the config.

The app only asks whether to save or export an updated config after an output export. Previewing does not trigger the config-save prompt.

Saved configs are browser-local for now. They are not shared between users, machines, or browsers.

---

## Operation Modes

Only the UI for the selected operation is shown. For example, Pivot hides regular transform and Unpivot controls; Unpivot hides Pivot and regular transform controls.

For the Photon-Flow experience, operation sections should be collapsible/minimizable when practical. The goal is to keep the page fast to scan: users should be able to collapse setup, mapping, filter, preview, and export areas once each step is complete.

### Regular Transform

Use Regular Transform for column-level cleanup and row-level filtering.

Main capabilities:

- Rename output columns.
- Reorder columns by dragging or using move buttons.
- Keep selected columns or exclude columns from output.
- Add formula, constant, and split-derived columns.
- Change data type and formatting.
- Apply data-type-specific transforms and formats. Text fields show text cleanup/casing options, number fields show numeric transforms/formats, and date fields show date extraction/format options.
- Apply bulk rename, prefix, suffix, find/replace, and bulk transforms.
- Use undo and reset while working.

### Data Types, Transforms, and Formats

Transform and format dropdowns are filtered by the selected data type. This keeps the UI focused so date fields do not show number-only options, number fields do not show text cleanup options, and so on.

#### Text

Text transforms:

- No change
- Trim spaces
- Collapse spaces
- To UPPERCASE
- To lowercase
- To Title Case
- Remove special characters
- Remove digits
- Keep digits only
- To snake_case

Text formats:

- No format
- Trim
- Collapse spaces
- UPPERCASE
- lowercase
- Title Case
- snake_case

#### Number

Number transforms:

- No change
- Parse number
- Absolute value
- Negate
- Round
- Floor
- Ceil

Number formats:

- No format
- Integer
- Decimal 2
- Decimal 4
- Currency
- Percent

#### Date

Date transforms:

- No change
- To `YYYY-MM-DD`
- To `MM/DD/YYYY`
- Extract year
- Extract month number
- Extract month name
- Extract day
- Extract quarter

Date formats:

- No format
- `YYYY-MM-DD`
- `MM/DD/YYYY`
- `YYYYMMDD`
- `MMM YYYY`
- `YYYY-MM`
- Month name

#### Boolean

Boolean transforms:

- No change
- Trim spaces
- To UPPERCASE
- To lowercase

Boolean formats:

- No format
- `true / false`
- `Yes / No`
- `Y / N`

These same type-aware controls are used in regular mapping, bulk transforms, pivot and aggregate measures, and unpivot generated fields.

### Row Filters

Regular Transform supports filtering by multiple columns from the UI.

Filter behavior is based on field type:

- Text fields support text-oriented matching.
- Number fields support numeric comparisons.
- Date fields support date-oriented comparisons.
- Custom JavaScript filters can be used for advanced logic.

### Pivot

Use Pivot when rows need to be summarized across one or more pivot dimensions.

Typical setup:

- Select grouping fields.
- Select the pivot field.
- Select one or more value fields.
- Choose aggregation behavior.
- Preview the pivoted output before export.

### Unpivot

Use Unpivot when wide columns need to be converted into rows.

Checking Unpivot opens a centered draggable settings popup. Use `Settings` to reopen the current configuration and `Clear` to remove the active unpivot setup.

Unpivot types:

| Type | Purpose |
| --- | --- |
| `Standard` | Converts selected measure columns into attribute/value rows while repeating fixed ID columns. |
| `Delimited Header` | Splits selected source headers by a delimiter and creates one output row per selected cell. |
| `Paired Metric` | Groups related columns with matching prefixes into one row containing multiple metric fields. |

#### Standard Unpivot

Fixed ID columns remain unchanged and are repeated for every generated row.

Selected unpivot columns are converted into:

- An attribute column containing the original column name.
- A value column containing the original cell value.

Example:

Input:

| Student_ID | Math_Score | Science_Score |
| --- | ---: | ---: |
| 1 | 90 | 85 |

Output:

| Student_ID | Attribute | Value |
| --- | --- | ---: |
| 1 | Math_Score | 90 |
| 1 | Science_Score | 85 |

The attribute and value output column names can be customized. Type, transform, and format options are available for generated fields.

#### Delimited Header Unpivot

Delimited Header uses a vertical split approach. Every selected source cell creates its own output row.

For each selected column:

1. Split the header by the selected delimiter.
2. Put the first split part into the configured first attribute column.
3. Put the remaining split text into the configured second attribute column.
4. Put the cell value into the configured value column.

Example with delimiter `_`:

Input:

| id | 2024_Sales | 2024_Profit |
| --- | ---: | ---: |
| 1 | 100 | 40 |

Output:

| id | Year | Metric | Amount |
| --- | --- | --- | ---: |
| 1 | 2024 | Sales | 100 |
| 1 | 2024 | Profit | 40 |

The first attribute name, second attribute name, and value column name are user-configurable. Attribute and value fields can also have type, transform, and format settings.

#### Paired Metric Unpivot

Paired Metric groups related columns by a shared prefix and outputs multiple value columns on the same generated row.

Example with delimiter `_`:

Input:

| id | Q1_Sales | Q1_Profit | Q2_Sales | Q2_Profit |
| --- | ---: | ---: | ---: | ---: |
| 1 | 100 | 40 | 150 | 55 |

Output:

| id | Period | Sales | Profit |
| --- | --- | ---: | ---: |
| 1 | Q1 | 100 | 40 |
| 1 | Q2 | 150 | 55 |

This is useful when columns share a common identifier such as month, quarter, year, or scenario and each group contains related metrics.

### Aggregate

Use Aggregate when rows should be grouped and summarized without building a pivot table.

Typical setup:

- Select group-by fields.
- Select measure fields.
- Choose aggregation functions.
- Preview and export the summarized output.

---

## Scripts

The app includes JavaScript scripting for advanced cases.

### Column Script

Column scripts modify the column mapping definition. Use them to rename, drop, reorder, or change transforms across many columns.

Example:

```js
colDefs.forEach(d => {
  if (/tmp|test/i.test(d.srcCol)) d.dropped = true;
});
```

### Row Script

Row scripts run during Preview and Export and can modify or drop rows.

Example:

```js
function transformRow(row) {
  if (!row.Amount || Number(row.Amount) === 0) return null;
  const out = Object.assign({}, row);
  out.Amount = Number(row.Amount);
  return out;
}
```

Scripts are saved in browser `localStorage` and remain local to that browser.

---

## Preview and Export

Preview shows transformed rows before writing a file. Use it to verify column names, values, filters, pivot output, unpivot output, and aggregate output.

Export downloads the transformed result. After export, the app can ask whether to save or download the latest transformation config.

---

## Processing Notes

- Header rows are treated as field names.
- Excel date serial headers are detected and converted where possible.
- Config validation checks saved mappings against the uploaded file headers.
- Local saved configs and scripts use browser `localStorage`.
- Browser local storage is not a database and is not shared across devices.
- For future database-backed configs, the current config JSON can be used as the portable format.

---

## Browser Compatibility

| Browser | Minimum Version |
| --- | --- |
| Microsoft Edge | 80+ |
| Google Chrome | 80+ |
| Mozilla Firefox | 113+ |

Internet Explorer is not supported.

---

## Updating the Tool

Replace the deployed `excel-transformer.html` file with the new version. Users receive the update the next time they reload the page.

Saved configs and scripts remain in each user's browser `localStorage` unless that browser data is cleared.
