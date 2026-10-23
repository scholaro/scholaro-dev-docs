# GPA Report Integration Guide

The Scholaro GPA Report integration supports multiple submission methods depending on how your team receives and processes transcripts.

> Note: If you are configuring downstream file delivery or event-based processing, see the [Webhooks](webhooks.md) section and the [Slate Integration Setup](integrations/slate-integration-setup.md) guide.

> Note: For how bulk imports are processed, where imported reports appear in Scholaro Premium, and troubleshooting, see [Bulk Import: Behavior and Troubleshooting](bulk-import.md).

## Integration methods

=== "Single API"

    Best for real-time processing, one transcript at a time.

=== "Bulk API"

    Best for ZIP uploads with a `manifest.csv` file.

=== "SFTP"

    Best for automated file-based workflows and higher-volume processing.

!!! note
    All methods use the same underlying processing and produce the same GPA report output.

## Authentication

Include your API key in every request.

```http
X-API-Key: your_api_key_here
```

Developer dashboard:

```text
https://www.scholaro.com/app/developer/webhooks
```

## Single Report API

### Endpoint

```http
POST https://www.scholaro.com/appAPI/api/gpa-report
```

> Note: For single GPA report requests, use the current `POST /appAPI/api/gpa-report` endpoint. Do not use the older `/new` path.

### Request format

Send data as `multipart/form-data`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `country` | string | Yes | Country of study |
| `file` | PDF file | Yes | Transcript file |
| `applicant_name` | string | No | Applicant name |
| `report_name` | string | No | Custom report name |
| `date_of_birth` | string | No | Available if enabled |
| `custom_field_1` | string | No | Optional |
| `custom_field_2` | string | No | Optional |
| `custom_field_3` | string | No | Optional |

### Example request

```bash
curl -X POST https://www.scholaro.com/appAPI/api/gpa-report \
  -H "X-API-Key: your_api_key_here" \
  -F "country=India" \
  -F "applicant_name=Jane Smith" \
  -F "report_name=Fall 2024 GPA Report" \
  -F file=@/path/to/transcript.pdf
```

### Example response

The response includes report metadata, GPA totals, and the `grades` array for the evaluated coursework.

```json
{
  "message": "string",
  "url": "string",
  "gpa_report": {
    "report_id": 123456,
    "applicant_name": "string",
    "report_name": "string",
    "country": "string",
    "institution": "string",
    "total_credits": 0.0,
    "total_points": 0.0,
    "gpa": 0.0,
    "grades": [
      {
        "class": "string",
        "grade_points": 0.0,
        "credits": 0.0,
        "grade": "string"
      }
    ]
  }
}
```

!!! note
    The transcript file must be a valid PDF.

!!! note
    This API is available only to institutional Premium users.

## Bulk API Upload

### Endpoint

```http
POST https://www.scholaro.com/appAPI/api/gpa-report/bulk
```

### Upload requirements

Send `multipart/form-data` with a single ZIP file.

```text
file: bulk-upload.zip
```

Requirements:

- Must be a `.zip` file
- Max size: 200 MB
- Must include `manifest.csv`
- Must include transcript files in PDF, JPG, JPEG, or PNG format

!!! warning
    The ZIP must not contain any other files. A ZIP created with macOS **Compress** includes hidden `.DS_Store` files and a `__MACOSX` folder, and is rejected. See [Bulk Import troubleshooting](bulk-import.md#the-bulk-api-rejected-a-zip-created-on-a-mac).

### manifest.csv format

Each row represents one report.

```csv
file_name,country,applicant_name,report_name,date_of_birth,custom_field_1,custom_field_2,custom_field_3
brazil-transcript.pdf,Brazil,Jane Smith,Fall 2024 Report,01/15/1995,Field1,Field2,Field3
india-transcript.pdf,India,John Doe,Spring 2024 Report,,,,
```

### Fields

| Field | Required | Description |
| --- | --- | --- |
| `file_name` | Yes | Name of transcript file in ZIP |
| `country` | Yes | Country for GPA conversion. Must match the country name used in Scholaro exactly, for example `India` |
| `applicant_name` | No | Extracted automatically if empty |
| `report_name` | No | Custom report name |
| `date_of_birth` | No | If enabled. Use the `YYYY-MM-DD` format |
| `custom_field_1` | No | Optional field. Used only if custom fields are enabled for your organization |
| `custom_field_2` | No | Optional field. Used only if custom fields are enabled for your organization |
| `custom_field_3` | No | Optional field. Used only if custom fields are enabled for your organization |

### CSV formatting rules

- The first row must be a header row with the field names above. Field names are not case-sensitive, and the columns can be in any order. Columns with other names are ignored.
- Separate values with commas, and save the file as UTF-8.
- **Put any value that contains a comma in double quotes.** Without quotes, the comma starts a new column and every value after it moves one column to the right.
- To include a double quote inside a quoted value, write it twice (`""`).
- Rows without a `file_name` or `country` are skipped.
- `file_name` must match the transcript's file name. Matching is not case-sensitive. Don't include a folder path.

For example, this row has a name that contains a comma, and a report name that contains quotes:

```csv
file_name,country,applicant_name,report_name,date_of_birth,custom_field_1,custom_field_2,custom_field_3
smith-transcript.pdf,India,"Smith, Jane","Fall 2024 ""Priority"" Report",1995-01-15,Field1,,
```

Spreadsheet programs such as Excel add the quotes for you when you save as **CSV UTF-8**.

### Accepted response

```json
{
  "batch_id": "5797602d-f05f-495b-8cb9-220c8e2c5df5",
  "total_items": 5,
  "status_url": "/api/gpa-report/bulk/{batch_id}"
}
```

### Example request

```bash
curl -X POST https://www.scholaro.com/appAPI/api/gpa-report/bulk \
  -H "X-API-Key: your_api_key_here" \
  -F file=@/path/to/bulk-upload.zip
```

### Common error responses

| Status | Description |
| --- | --- |
| `400` | Invalid request, missing ZIP, manifest, files, or limits exceeded |
| `401` | Unauthorized, invalid API key |

## Batch Status

### Endpoint

```http
GET https://www.scholaro.com/appAPI/api/gpa-report/bulk/{batch_id}
```

### Example response

```json
{
  "batch_id": "5797602d-f05f-495b-8cb9-220c8e2c5df5",
  "status": "Completed",
  "total_items": 5,
  "completed_items": 3,
  "failed_items": 2,
  "created_on": "2024-12-15T10:30:00Z",
  "completed_on": "2024-12-15T10:35:00Z",
  "items": [
    {
      "file_name": "brazil-transcript.pdf",
      "status": "Completed",
      "report_id": 737271,
      "error_message": null
    }
  ]
}
```

### Status values

Batch status values:

- `Processing`
- `Completed`
- `Cancelled`
- `Failed` (an SFTP upload that was rejected before processing started)

Item status values:

- `Pending`
- `Processing`
- `Completed`
- `Failed`
- `Cancelled`

!!! note
    `Completed` means every file has finished processing, not that every file succeeded. Check `failed_items` and each item's `error_message`. For what each message means, see [Bulk Import troubleshooting](bulk-import.md#troubleshooting).

## SFTP Upload

### Connection details

| Setting | Value |
| --- | --- |
| Host | `scholaronorthussftp.blob.core.windows.net` |
| Port | `22` |
| Protocol | `SFTP` |
| Username | Provided by Scholaro |
| Password | Provided by Scholaro |

### Upload steps

Scholaro sets up your organization for one of two upload formats: a single ZIP file, or individual files. Contact Scholaro if you're not sure which one your organization uses. An upload in the other format isn't processed.

=== "ZIP upload"

    Upload a single ZIP file containing `manifest.csv` and all referenced transcript files. Uploading the ZIP triggers processing.

    Requirements:

    - Must include `manifest.csv` at the root of the archive
    - Must include every transcript file referenced in the manifest
    - Transcript files must be PDF, JPG, JPEG, or PNG
    - Manifest format matches the [Bulk API manifest](#manifestcsv-format)

=== "Individual files"

    Upload the transcript files and `manifest.csv` to the top-level SFTP folder, with **`manifest.csv` last**. Uploading `manifest.csv` triggers processing.

    Requirements:

    - Upload every transcript file referenced in the manifest. If a file hasn't arrived, Scholaro waits up to 15 minutes for it after `manifest.csv` is uploaded, then rejects the upload.
    - Upload the transcript files again for each new batch. Files that were already in the folder before your last batch don't count.
    - Don't use subfolders. List each file in `file_name` by its name only.
    - Transcript files must be PDF, JPG, JPEG, or PNG
    - Manifest format matches the [Bulk API manifest](#manifestcsv-format)

### Example session

=== "ZIP upload"

    ```bash
    sftp scholaronorthussftp.myuser@scholaronorthussftp.blob.core.windows.net
    put bulk-upload.zip
    ```

=== "Individual files"

    ```bash
    sftp scholaronorthussftp.myuser@scholaronorthussftp.blob.core.windows.net
    put brazil-transcript.pdf
    put india-transcript.pdf
    put manifest.csv
    ```

!!! note
    If `manifest.csv` is missing from the ZIP, or any file referenced in the manifest is missing, the upload is rejected. Scholaro records the rejected upload as a batch with status `Failed`, and sends a `bulk_upload.completed` event whose items explain why.

!!! note
    Uploading the ZIP (or `manifest.csv`) triggers processing. Subscription limits apply — if the batch would exceed the report limit, the upload is rejected in the same way.

!!! warning
    Do not upload multiple batches for the same organization concurrently. Only one active batch per org is processed at a time, and an upload that arrives while another batch is processing is skipped.

For what happens after the upload, where the reports appear, and what to check when an upload doesn't work, see [Bulk Import: Behavior and Troubleshooting](bulk-import.md).
