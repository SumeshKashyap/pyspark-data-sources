# Feature Specification: Add SharePoint/Teams Excel Reader to `pyspark-data-sources`

## Objective

Implement a new datasource in the **pyspark-data-sources** repository
that enables reading Excel files stored in **SharePoint Online** or
**Microsoft Teams** (backed by SharePoint) using **Microsoft Graph API**
and **Service Principal authentication**.

The implementation should follow the architecture, coding standards,
abstractions, and repository conventions already established in the
project.

------------------------------------------------------------------------

# Step 1 - Understand Repository Standards

Before implementing anything, study the repository standards and coding
patterns.

## Read

Repository:

`https://github.com/SumeshKashyap/pyspark-data-sources`

Mandatory documents:

`AGENTS.md`

Repository skills folder (Cursor Skills)

Read all relevant skills to understand:

-   Repository architecture
-   Coding conventions
-   File organization
-   Datasource implementation template
-   Error handling
-   Logging
-   Documentation style
-   Testing patterns

Do **not** start implementation until these conventions are understood.

------------------------------------------------------------------------

# Step 2 - Study Existing SharePoint Prototype

Read the prototype project located at

`/media/sumesh/New Volume/Ubuntu_Projects/Data Engineering/SharePoint_Read/sharepoint-graph-reader`

Focus especially on `main.py`.

Understand the following flow:

1.  Load configuration
2.  Authenticate using MSAL
3.  Acquire Microsoft Graph token
4.  Retrieve Site ID
5.  Locate Excel file
6.  Read Excel directly into memory
7.  Convert to dataframe

The implementation should reuse this authentication approach where
appropriate while adapting it to Spark.

------------------------------------------------------------------------

# Step 3 - Implement New Datasource

Inside the `pyspark-data-sources` repository, create a new datasource:

`excel_sharepoint_teams.py`

Follow the same implementation template as other datasource readers in
the repository.

Do **not** introduce a completely different design.

------------------------------------------------------------------------

# Functional Requirements

The datasource must support reading Excel files from:

-   SharePoint Online
-   Microsoft Teams document libraries

using:

-   Microsoft Graph API
-   OAuth2 Client Credentials
-   Service Principal
-   Sites.Selected permission model

## Authentication Parameters

The datasource should accept:

-   tenant_id
-   client_id
-   client_secret
-   site_host
-   site_path
-   file_path

Example:

``` python
site_host = "skltd2306.sharepoint.com"
site_path = "/sites/mySharePoint"
file_path = "/General/sample_sheet.xlsx"
```

## Excel Read Options

Support optional parameters:

-   sheet_name
-   header_row
-   start_row
-   start_column
-   use_columns
-   nrows

These should be translated appropriately into the underlying
Pandas/OpenPyXL reader before conversion to Spark.

## Reading Logic

Authenticate

↓

Acquire Graph token

↓

Resolve SharePoint Site ID

↓

Download Excel as byte stream

↓

Read Excel directly from memory

↓

Create Spark DataFrame

No temporary files should be created on disk. Use `BytesIO`.

## Spark Integration

Return a **Spark DataFrame**. Use Pandas only as an intermediate step.

## Output Support

Integrate with the repository's existing output abstraction if
available.

Otherwise support:

-   Parquet
-   Delta Lake

Do not duplicate existing writer functionality.

## Code Quality

-   Follow repository coding conventions
-   Use type hints
-   Include docstrings
-   Use repository logging utilities
-   Implement proper exception handling
-   Avoid duplicated code
-   Keep authentication isolated where practical

## Configuration

Avoid hardcoded values.

Everything should be configurable.

## Error Handling

Handle:

-   Authentication failures
-   Permission errors
-   Missing site
-   Missing file
-   Missing sheet
-   Invalid Excel
-   Network failures

Provide meaningful exception messages.

## Documentation

Document:

-   Authentication flow
-   Microsoft Graph permissions
-   Sites.Selected requirements
-   SharePoint Administrator site permission grant
-   Python dependencies
-   Usage examples

Example:

``` python
reader = SharePointTeamsExcelReader(...)

df = reader.read()

df.show()
```

## Testing

Follow the repository's testing framework.

Include tests for:

-   Authentication
-   Missing file
-   Invalid sheet
-   Successful read

Mock Graph API responses where appropriate.

## Non-Functional Requirements

-   Avoid unnecessary Graph requests
-   Stream files into memory
-   Support large Excel files where practical
-   Keep authentication reusable
-   Avoid temporary files

## Deliverables

1.  `excel_sharepoint_teams.py`
2.  Supporting helper classes if required
3.  Documentation
4.  Tests
5.  Usage example
6.  Dependency updates

## Implementation Guidance

-   Follow repository patterns first.
-   Reuse existing datasource interfaces, logging, configuration
    handling, and output abstractions.
-   Encapsulate Microsoft Graph authentication for reuse.
-   Minimize external dependencies.
-   Make the datasource feel like a native part of the framework.
