# n8n Google Sheets Lead Data Cleanup

A portfolio project demonstrating a small ETL pipeline built with **n8n** and **Google Sheets**.

The workflow reads raw lead records from Google Sheets, validates required fields, normalizes the data, removes duplicate email addresses, adds a processing timestamp, and appends or updates clean records in a destination sheet.

Invalid records are routed to a separate branch where an `Error_Reason` field is generated for inspection in the n8n execution output.

## Workflow Overview

<img width="1526" height="562" alt="image" src="https://github.com/user-attachments/assets/de9f056e-b636-4b10-b6e1-b0e6f101d1a9" />

## Features

- Manual execution for testing
- Google Sheets row-added trigger
- Google Sheets row-updated trigger
- Required-field validation
- Null-safe expressions
- Name, email, and phone normalization
- Email-based duplicate removal
- ISO timestamp generation
- Append-or-update behavior using normalized email
- Rejected-record error descriptions
- Separate success and rejection branches

## Required Fields

A record is treated as valid when all of the following fields are present:

- `ID`
- `Full Name`
- `Email`
- `Phone Number`

Invalid records do not continue through the transformation pipeline.

## Data Transformations

| Source field | Output field | Transformation |
|---|---|---|
| `ID` | `ID` | Preserved |
| `Full Name` | `FullName_Normalized` | Trimmed and converted to lowercase |
| `Email` | `Email_Normalized` | Trimmed and converted to lowercase |
| `Phone Number` | `Phone_Number_Normalized` | All non-numeric characters removed |
| `Country` | `Country` | Preserved |
| `State` | `State` | Preserved |
| `City` | `City` | Preserved |
| `Qualification` | `Qualification` | Preserved |
| `Priority` | `Priority` | Preserved |
| — | `Date` | Current timestamp in ISO 8601 format |

## Duplicate Handling

The workflow removes duplicate items using:

```text
Email_Normalized
```

The final Google Sheets node also uses `Email_Normalized` as its matching column.

This means:

- a new normalized email is appended;
- an existing normalized email is updated;
- duplicate email records within the same execution are removed before writing.

## Invalid Record Handling

Records that fail validation are routed to an error branch.

The workflow generates:

```text
ID
Error_Reason
```

Example:

```json
{
  "ID": 104,
  "Error_Reason": "Missing Email; Missing Phone Number"
}
```

The rejected branch intentionally ends without writing to another spreadsheet. Rejection details can be inspected in the n8n execution data.

## Example

### Source record

```json
{
  "ID": 101,
  "Full Name": "  Jane Smith  ",
  "Email": " JANE.SMITH@EXAMPLE.COM ",
  "Phone Number": "+1 (555) 123-4567",
  "Country": "United States",
  "State": "California",
  "City": "San Diego",
  "Qualification": "Qualified",
  "Priority": "High"
}
```

### Clean record

```json
{
  "ID": 101,
  "FullName_Normalized": "jane smith",
  "Email_Normalized": "jane.smith@example.com",
  "Phone_Number_Normalized": "15551234567",
  "Country": "United States",
  "State": "California",
  "City": "San Diego",
  "Qualification": "Qualified",
  "Priority": "High",
  "Date": "2026-06-25T12:00:00.000Z"
}
```

## Repository Structure

```text
n8n-google-sheets-data-cleanup/
├── workflows/
│   └── google-sheets-data-cleanup.json
├── docs/
│   └── workflow-overview.png
├── sample-data/
│   └── sample-leads.csv
├── .gitignore
├── LICENSE
└── README.md
```

`sample-data/` and `LICENSE` are optional but recommended.

## Prerequisites

- An n8n instance
- A Google account
- Google Sheets credentials configured in n8n
- A spreadsheet containing:
  - a source sheet for raw lead records;
  - a destination sheet for cleaned records.

## Expected Source Columns

```text
ID
Full Name
Email
Phone Number
Country
State
City
Qualification
Priority
```

## Expected Destination Columns

```text
ID
FullName_Normalized
Email_Normalized
Phone_Number_Normalized
Country
State
City
Qualification
Priority
Date
```

## Setup

1. Clone or download this repository.
2. Import `workflows/google-sheets-data-cleanup.json` into n8n.
3. Configure your own Google Sheets credentials.
4. Open every Google Sheets and Google Sheets Trigger node.
5. Select your own spreadsheet and sheet names.
6. Confirm that all trigger nodes reference the intended source sheet.
7. Confirm that the destination sheet contains the expected output columns.
8. Run the workflow using the manual trigger.
9. Test both valid and invalid source records.
10. Activate the workflow only after verifying the results.

## Suggested Test Cases

Test at least these records before activation:

| Test | Expected result |
|---|---|
| Complete valid record | Written to the clean sheet |
| Email with uppercase letters and spaces | Normalized to lowercase and trimmed |
| Formatted phone number | Stored with digits only |
| Duplicate email | Duplicate removed or existing row updated |
| Missing full name | Routed to rejected branch |
| Missing email | Routed to rejected branch |
| Missing phone number | Routed to rejected branch |
| Multiple missing fields | Combined `Error_Reason` value |

## Security

The public workflow export must not contain:

- Google Sheet IDs
- private spreadsheet URLs
- credential IDs
- credential names tied to a personal account
- n8n instance IDs
- API keys or tokens
- real lead or customer data
- private email addresses
- internal company information

Replace private values with placeholders such as:

```text
YOUR_GOOGLE_SHEET_ID
YOUR_SOURCE_SHEET
YOUR_DESTINATION_SHEET
CONFIGURE_YOUR_GOOGLE_CREDENTIALS
```

Anyone importing the workflow must configure their own credentials and spreadsheet references.

## Limitations

This project is designed as a portfolio demonstration, not a production data platform.

Potential improvements include:

- retry settings for Google Sheets operations;
- centralized error notifications;
- rejected-record persistence;
- stricter email validation;
- international phone-number normalization;
- batch controls for large spreadsheets;
- automated test fixtures;
- execution monitoring and alerting.

## Skills Demonstrated

- n8n workflow development
- ETL workflow design
- Google Sheets integration
- Data validation
- Data normalization
- Duplicate handling
- Branching and error handling
- Idempotent update logic
- Technical documentation

## Screenshot

Add a screenshot of the full workflow canvas here:

```markdown
![Workflow overview](docs/workflow-overview.png)
```

## License

This repository is intended for portfolio and educational use.

Add a license such as MIT only when you are comfortable allowing others to reuse and modify the workflow.
