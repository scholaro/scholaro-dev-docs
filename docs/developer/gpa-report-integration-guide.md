# GPA Report Integration Guide

The Scholaro GPA Report integration supports multiple submission methods depending on how your team receives and processes transcripts.

> Note: If you are configuring downstream file delivery or event-based processing, see the [Webhooks](webhooks.md) section and the [Slate Integration Setup](integrations/slate-integration-setup.md) guide.

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
- Max size: 100 MB
- Must include `manifest.csv`
- Must include transcript files in PDF, JPG, JPEG, or PNG format

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
| `country` | Yes | Country for GPA conversion |
| `applicant_name` | No | Extracted automatically if empty |
| `report_name` | No | Custom report name |
| `date_of_birth` | No | If enabled |
| `custom_field_1` | No | Optional field |
| `custom_field_2` | No | Optional field |
| `custom_field_3` | No | Optional field |

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

Item status values:

- `Pending`
- `Processing`
- `Completed`
- `Failed`

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

Upload a single ZIP file containing `manifest.csv` and all referenced transcript files. Uploading the ZIP triggers processing.

Requirements:

- Must include `manifest.csv` at the root of the archive
- Must include every transcript file referenced in the manifest
- Transcript files must be PDF, JPG, JPEG, or PNG
- Manifest format matches the [Bulk API manifest](#manifestcsv-format)

### Example session

```bash
sftp scholaronorthussftp.myuser@scholaronorthussftp.blob.core.windows.net
put bulk-upload.zip
```

!!! note
    The ZIP must contain `manifest.csv`. If any file referenced in the manifest is missing from the archive, the upload is rejected and no batch is created.

!!! note
    Uploading the ZIP triggers processing. Subscription limits apply — if the batch would exceed the report limit, no batch is created.

!!! warning
    Do not upload multiple ZIPs for the same organization concurrently. Only one active batch per org is processed at a time.
