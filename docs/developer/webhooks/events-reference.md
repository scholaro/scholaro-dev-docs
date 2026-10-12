# Events Reference

This page lists the Scholaro webhook events, when each one is sent, the value it carries in the `type` field, and every field in its payload.

To set up an endpoint, see the [Webhooks Overview](overview.md). To check that a request came from Scholaro, see [Signature Verification](signature-verification.md).

## Event names and `type` values

Data events are sent to your endpoint as JSON. The `type` field in the request body names the event.

### Data events

| Event in Scholaro | `type` value in the payload |
| --- | --- |
| `gpa_report.created (data)` | `gpa_report.created` |
| `gpa_report.updated (data)` | `gpa_report.updated` |
| `digital_evaluation.created (data)` | `digital_evaluation.created` |
| `digital_evaluation.updated (data)` | `digital_evaluation.updated` |
| `bulk_upload.started` | `bulk_upload.started` |
| `bulk_upload.completed` | `bulk_upload.completed` |

### PDF events

- `gpa_report.created (pdf)`
- `gpa_report.updated (pdf)`
- `digital_evaluation.created (pdf)`
- `digital_evaluation.updated (pdf)`

PDF events do not post JSON to your endpoint. When one occurs, Scholaro saves a copy of the report and a nightly process delivers the file. See [PDF Delivery Events](pdf-delivery-events.md).

## When events are sent

| Event | Sent when |
| --- | --- |
| `gpa_report.created` | A GPA report is created, in Scholaro or through the GPA Report API. Sent once per report. |
| `gpa_report.updated` | A different grading scale is applied to the report, or the applicant name or report name is edited in the GPA Reports list. Sent every time. |
| `digital_evaluation.created` | A digital evaluation is created. Sent once per evaluation. |
| `digital_evaluation.updated` | A GPA report is linked to or unlinked from the evaluation, one of its custom fields is edited, or the applicant name or evaluation name is edited in the Evaluations list. Sent every time. |
| `bulk_upload.started` | A bulk upload is accepted through the GPA Report API or SFTP and starts processing. |
| `bulk_upload.completed` | Every file in a bulk upload has finished processing. |

Editing grades or courses on a GPA report does not send an `updated` event.

GPA report and digital evaluation events are only sent for reports created within the last 14 days. Changes to older reports send nothing.

## Payload structure

Every data event is sent as an HTTP `POST` with a JSON body in this shape:

```json
{
  "object": "event",
  "type": "gpa_report.created",
  "data": {
    "...": "..."
  }
}
```

| Field | Type | Description |
| --- | --- | --- |
| `object` | string | Always `event` |
| `type` | string | The event name, for example `gpa_report.updated` (see the data events table above) |
| `data` | object | The GPA report, digital evaluation or bulk upload |

The `data` object has the same fields for `created` and `updated` events. It always holds the report's current values.

Fields with no value are sent as `null`. Numbers keep their stored precision, so they can include trailing zeros (`120.00`). Parse them as decimal numbers and don't rely on the number of decimal places.

## GPA report

Sent for `gpa_report.created (data)` and `gpa_report.updated (data)`.

| Field | Type | Description |
| --- | --- | --- |
| `id` | integer | Unique report ID. |
| `report_name` | string | Report name. |
| `full_name` | string | The applicant's full name. |
| `date` | string (date-time) | Date and time the report was created, for example `2026-04-23T15:11:34.09`. No time-zone offset is included. |
| `dob` | string or null | The applicant's date of birth as a short date. The format follows the language of the Scholaro user who triggered the event (`M/d/yyyy` in English). |
| `cust_field1` | string or null | Custom field 1. |
| `cust_field2` | string or null | Custom field 2. |
| `cust_field3` | string or null | Custom field 3. |
| `country` | string | Country of study. |
| `institution` | string or null | Foreign institution name. |
| `qualification` | string or null | Program name (Bachelor's, Master's, etc.). |
| `subject` | string or null | Subject or area of study. |
| `grading_scale` | string | Name of the grading scale applied to the report. |
| `gpa` | number or null | Cumulative GPA, including any adjustment applied to the report. |
| `total_credits` | number | Total converted (US) credits, to 2 decimal places. |
| `total_points` | number | Total grade points: the sum of each course's grade points multiplied by its converted credits. Not rounded. |
| `courses` | array or null | The report's courses (see [Course](#course)). `null` unless course details are enabled for your webhook. |

```json
{
  "object": "event",
  "type": "gpa_report.created",
  "data": {
    "id": 192904,
    "report_name": "Bachelor of Commerce",
    "full_name": "Priya Sharma",
    "date": "2026-04-23T15:11:34.09",
    "dob": "12/1/2002",
    "cust_field1": "1234567",
    "cust_field2": "001234567",
    "cust_field3": "2",
    "country": "India",
    "institution": "Osmania University",
    "qualification": "Bachelor of Commerce",
    "subject": "Finance",
    "grading_scale": "Most Common",
    "gpa": 3.77,
    "total_credits": 120.0,
    "total_points": 452.4,
    "courses": null
  }
}
```

### Course

Each item in `courses`, in transcript order.

| Field | Type | Description |
| --- | --- | --- |
| `id` | integer | Unique course row ID. |
| `name` | string or null | Course title as entered from the transcript. |
| `credits` | string or null | Credits as shown on the transcript. |
| `credits_converted` | number | Converted (US) credits. |
| `grade` | string or null | Grade as shown on the transcript. |
| `us_grade` | string or null | Converted US grade. |
| `grade_points` | number | Grade points for the converted grade. |
| `sort_order` | integer | Position of the course on the report. |

```json
"courses": [
  {
    "id": 5530112,
    "name": "Financial Accounting",
    "credits": "4",
    "credits_converted": 4.00,
    "grade": "A+",
    "us_grade": "A",
    "grade_points": 4.000,
    "sort_order": 1
  }
]
```

## Digital evaluation

Sent for `digital_evaluation.created (data)` and `digital_evaluation.updated (data)`.

| Field | Type | Description |
| --- | --- | --- |
| `id` | integer | Unique evaluation ID. |
| `evaluation_name` | string | Evaluation name. |
| `full_name` | string | The applicant's full name. |
| `date` | string (date-time) | Date and time the evaluation was created. No time-zone offset is included. |
| `cust_field1` | string or null | Custom field 1. |
| `cust_field2` | string or null | Custom field 2. |
| `cust_field3` | string or null | Custom field 3. |
| `country` | string | Country of study. |
| `institution` | string or null | Foreign institution name. |
| `accreditation` | string or null | The institution's accrediting or recognizing body, or `N/A` if none is recorded. |
| `credential` | string | Credential name, as the native name followed by the English name in brackets, or the English name alone. |
| `program_length` | number | Program length in years, to 1 decimal place. |
| `equivalency` | string | US equivalency statement. |
| `gpa_report` | integer or null | `id` of the GPA report linked to this evaluation, or `null` if there is none. |

```json
{
  "object": "event",
  "type": "digital_evaluation.created",
  "data": {
    "id": 113945,
    "evaluation_name": "Equivalency Report",
    "full_name": "Priya Sharma",
    "date": "2026-04-26T09:14:41.25",
    "cust_field1": "1234567",
    "cust_field2": "001234567",
    "cust_field3": "2",
    "country": "India",
    "institution": "Osmania University",
    "accreditation": "University Grants Commission (UGC)",
    "credential": "Bachelor of Commerce",
    "program_length": 3.0,
    "equivalency": "Three years of undergraduate coursework, comparable to a Bachelor's degree",
    "gpa_report": 192904
  }
}
```

## Bulk upload

Sent for bulk uploads made through the [GPA Report API](../gpa-report-integration-guide.md) or SFTP. These payloads have no `object` field, and fields with no value are left out rather than sent as `null`. Times are in UTC and may or may not end in `Z`, so parse them as UTC either way.

### `bulk_upload.started`

| Field | Type | Description |
| --- | --- | --- |
| `batch_id` | string (UUID) | Batch ID, as returned when the upload was accepted. |
| `status` | string | `Processing` |
| `total_items` | integer | Number of files in the upload. |
| `source` | string | `API` or `SFTP` |
| `created_on` | string (date-time) | When the upload was accepted. |

```json
{
  "type": "bulk_upload.started",
  "data": {
    "batch_id": "5797602d-f05f-495b-8cb9-220c8e2c5df5",
    "status": "Processing",
    "total_items": 2,
    "source": "API",
    "created_on": "2026-04-23T15:11:34.0912345Z"
  }
}
```

### `bulk_upload.completed`

Has the same fields as `bulk_upload.started`, with `status` set to `Completed`, plus:

| Field | Type | Description |
| --- | --- | --- |
| `completed_items` | integer | Files that produced a report. |
| `failed_items` | integer | Files that failed. |
| `completed_on` | string (date-time) | When processing finished. |
| `items` | array | One entry per file. |
| `items[].file_name` | string | Uploaded file name. |
| `items[].status` | string | `Completed`, `Failed` or `Cancelled` |
| `items[].report_ids` | array of integers | IDs of the GPA reports created from the file. |
| `items[].error_message` | string | Why the file failed. Left out when it succeeded. |

```json
{
  "type": "bulk_upload.completed",
  "data": {
    "batch_id": "5797602d-f05f-495b-8cb9-220c8e2c5df5",
    "status": "Completed",
    "total_items": 2,
    "completed_items": 1,
    "failed_items": 1,
    "source": "API",
    "created_on": "2026-04-23T15:11:34.09",
    "completed_on": "2026-04-23T15:14:02.5139876Z",
    "items": [
      { "file_name": "transcript-1.pdf", "status": "Completed", "report_ids": [192904] },
      { "file_name": "transcript-2.pdf", "status": "Failed", "report_ids": [], "error_message": "The document could not be read." }
    ]
  }
}
```

!!! note "GPA reports created through the API"
    A `gpa_report.created` event for a report created through the GPA Report API has the same fields as above, with three differences: it has no `object` field, fields with no value are left out instead of sent as `null`, and each course can include a `code` with the course code printed on the transcript.

## Changes since the v2 PDF guide

If you built your integration from the original *Scholaro Webhooks* PDF, note these changes:

- `degree_report.created` and `degree_report.updated` are now `digital_evaluation.created` and `digital_evaluation.updated`. In Scholaro, choose the `(data)` version of each event.
- The digital evaluation payload names the report in `evaluation_name` instead of `report_name`, and adds `institution` and `accreditation`.
- GPA reports can include `courses`.
- `gpa_report.updated` is also sent when the applicant name or report name is edited, not only when a different grading scale is selected.
- Bulk upload events and PDF events were added.

!!! warning "Earlier `type` values"
    For a period, `type` carried an internal event code instead of the event name, for example `premium-webhook-event-gpa-report-created-data` instead of `gpa_report.created`. If your integration matches on those codes, update it to the names in the [data events table](#data-events).

## Related docs

- [Webhooks Overview](overview.md)
- [Signature Verification](signature-verification.md)
- [PDF Delivery Events](pdf-delivery-events.md)
- [TargetX / Salesforce File Delivery](targetx-salesforce-file-delivery.md)
- [Slate Integration Setup](../integrations/slate-integration-setup.md)
