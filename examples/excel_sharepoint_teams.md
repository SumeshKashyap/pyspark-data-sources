# SharePoint / Teams Excel Reader Example

Read Excel files stored in **SharePoint Online** or **Microsoft Teams** document libraries
directly into a Spark DataFrame — no intermediate files written to disk.

## Prerequisites

### Install dependencies

```bash
pip install pyspark-data-sources[sharepoint-excel]
```

### Azure AD app registration

1. Register an application in [Azure AD (Entra ID)](https://portal.azure.com).
2. Create a **client secret** and note the `client_id`, `client_secret`, and `tenant_id`.
3. Under **API permissions**, add one of the following **Application** permissions and grant
   admin consent:
   - `Sites.Selected` *(recommended — least privilege)*
   - `Sites.Read.All`

### Grant site access (Sites.Selected only)

With `Sites.Selected`, a SharePoint administrator must explicitly grant the app read access
to the target site. Use the Graph API:

```bash
POST https://graph.microsoft.com/v1.0/sites/{site-id}/permissions
Content-Type: application/json
Authorization: Bearer <admin-token>

{
    "roles": ["read"],
    "grantedToIdentities": [
        {
            "application": {
                "id": "<your-app-client-id>",
                "displayName": "<your-app-display-name>"
            }
        }
    ]
}
```

> **Tip:** To find the `site-id`, call:
> `GET https://graph.microsoft.com/v1.0/sites/{site_host}:{site_path}:`

---

## End-to-End Example

### Step 1: Create a Spark session and register the data source

```python
from pyspark.sql import SparkSession
from pyspark_datasources import SharePointExcelDataSource

spark = SparkSession.builder.appName("sharepoint-excel-example").getOrCreate()
spark.dataSource.register(SharePointExcelDataSource)
```

### Step 2: Read an Excel file from SharePoint Online

```python
df = (
    spark.read.format("sharepoint_excel")
    .option("tenant_id",    "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx")
    .option("client_id",    "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx")
    .option("client_secret","<your-client-secret>")
    .option("site_host",    "contoso.sharepoint.com")
    .option("site_path",    "/sites/MySharePoint")
    .option("file_path",    "/General/sales_report.xlsx")
    .load()
)
df.show()
```

### Step 3: Read a specific sheet with a row limit

```python
df = (
    spark.read.format("sharepoint_excel")
    .option("tenant_id",    "<tenant-id>")
    .option("client_id",    "<client-id>")
    .option("client_secret","<client-secret>")
    .option("site_host",    "contoso.sharepoint.com")
    .option("site_path",    "/sites/MySharePoint")
    .option("file_path",    "/General/sales_report.xlsx")
    .option("sheet_name",   "Q1 2024")
    .option("nrows",        "1000")
    .load()
)
df.show()
```

### Step 4: Read from a Microsoft Teams channel

Microsoft Teams stores channel files in a SharePoint site. Use the Teams site path:

```python
df = (
    spark.read.format("sharepoint_excel")
    .option("tenant_id",    "<tenant-id>")
    .option("client_id",    "<client-id>")
    .option("client_secret","<client-secret>")
    .option("site_host",    "contoso.sharepoint.com")
    .option("site_path",    "/teams/DataTeam")        # Teams site
    .option("file_path",    "/General/forecast.xlsx") # file in channel
    .load()
)
df.show()
```

### Step 5: Apply a custom schema for typed columns

By default all columns are returned as strings. Provide a custom schema for typed access:

```python
from pyspark.sql.types import StructType, StructField, StringType, DoubleType, LongType

schema = StructType([
    StructField("product",  StringType()),
    StructField("quantity", LongType()),
    StructField("revenue",  DoubleType()),
])

df = (
    spark.read.format("sharepoint_excel")
    .schema(schema)
    .option("tenant_id",    "<tenant-id>")
    .option("client_id",    "<client-id>")
    .option("client_secret","<client-secret>")
    .option("site_host",    "contoso.sharepoint.com")
    .option("site_path",    "/sites/MySharePoint")
    .option("file_path",    "/General/sales_report.xlsx")
    .load()
)
df.printSchema()
df.show()
```

### Step 6: Write results to Parquet or Delta Lake

```python
# Write to Parquet
df.write.parquet("/data/lake/sales_report")

# Write to Delta Lake (requires delta library)
df.write.format("delta").save("/data/lake/delta/sales_report")
```

---

## All Options

| Option | Required | Description |
|---|---|---|
| `tenant_id` | ✅ | Azure AD tenant ID |
| `client_id` | ✅ | Azure AD application (client) ID |
| `client_secret` | ✅ | Azure AD application client secret |
| `site_host` | ✅ | SharePoint host (e.g., `contoso.sharepoint.com`) |
| `site_path` | ✅ | Site path (e.g., `/sites/MySite` or `/teams/MyTeam`) |
| `file_path` | ✅ | Drive-relative path to the Excel file (e.g., `/General/data.xlsx`) |
| `sheet_name` | ➖ | Sheet name to read. Defaults to first sheet. |
| `header_row` | ➖ | 0-indexed row number to use as column headers (pandas `header`). Default: `0`. |
| `start_row` | ➖ | Number of rows to skip at the top before the header (pandas `skiprows`). Default: `0`. |
| `start_column` | ➖ | 0-indexed column to start reading from; earlier columns are dropped. Default: `0`. |
| `use_columns` | ➖ | Comma-separated column names or Excel range (e.g., `"A:C"` or `"Name,Age"`). |
| `nrows` | ➖ | Maximum number of data rows to read. |

## Features

- **In-memory download**: Excel file is streamed into `BytesIO` — no temporary files on disk.
- **SharePoint and Teams**: Works with any SharePoint site, including Teams-backed libraries.
- **All-string schema by default**: Safe for exploration; use `.schema()` for typed access.
- **Full Excel read options**: Sheet selection, header configuration, column filtering, row limits.
- **Typed writes**: Standard Spark DataFrame; write to Parquet, Delta, CSV, or any Spark sink.
- **Least-privilege auth**: Supports `Sites.Selected` for scoped access to individual sites.
