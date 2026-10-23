# Bulk Import: Behavior and Troubleshooting

This page explains what happens after you submit a bulk import through the [Bulk API](gpa-report-integration-guide.md#bulk-api-upload) or [SFTP](gpa-report-integration-guide.md#sftp-upload): how a batch is processed, where the reports appear, which webhook events are sent, and what to check when something goes wrong.

For upload requirements and the `manifest.csv` format, see the [GPA Report Integration Guide](gpa-report-integration-guide.md#manifestcsv-format).

## How a bulk import is processed

1. **Scholaro checks the upload.** It reads `manifest.csv` and checks that every file it names is present, that the file names are valid, and that the batch fits within your subscription's report limit.
2. **A batch is created.** If the checks pass, Scholaro creates a batch with status `Processing` and one item per manifest row, and sends `bulk_upload.started`. The Bulk API returns the `batch_id` in its response.
3. **Each file is processed.** Files are processed in the background, several at a time. Each item moves from `Pending` to `Processing`, then to `Completed` or `Failed`. A file that takes longer than 10 minutes fails with `Processing timed out after 10 minutes.`
4. **The batch finishes.** When every item has finished, the batch status becomes `Completed` and Scholaro sends `bulk_upload.completed` with the result of each file.

Some things to know:

- **`Completed` means the batch has finished, not that every file succeeded.** Check `failed_items` and each item's `status` and `error_message`.
- **One file can create more than one report.** If a transcript contains more than one credential, Scholaro creates a report for each one. Their IDs are listed in the item's `report_ids`. The item is `Completed` if at least one report was created.
- **A blank `applicant_name` is filled in from the transcript.** If no name can be read, the report is named `Unknown`.
- **If an SFTP upload fails the checks in step 1,** Scholaro records a batch with status `Failed` and sends `bulk_upload.completed` (with no `bulk_upload.started` before it). Each item's `error_message` says why the upload was rejected. A Bulk API upload that fails the checks gets a `400` response instead, and no batch is created.
- **Only one SFTP batch per organization is processed at a time.** An SFTP upload that arrives while another batch is still processing is skipped. Upload it again after the current batch finishes.

## Where imported reports appear

Reports created by a bulk import appear in Scholaro Premium under **My Assessments > GPA Reports**, alongside reports created in Scholaro:

- Every user in your organization can see them, not only the person who set up the integration.
- The list shows the newest reports first. If you don't see a report, check the timeframe filter above the list (it defaults to **Past 6 Months**) and set it to **All Time** if needed.
- Bulk-imported reports aren't marked differently from other reports. Use the report ID from the `report_ids` in the batch status or the `bulk_upload.completed` event to find a specific report.

While a batch is processing, the **Bulk Import** button on the GPA Reports toolbar shows its progress (for example, *Bulk import: 3/10 processed*) and a **Cancel** link. This applies to SFTP batches too. Scholaro Premium doesn't list past batches or per-file errors. Use the [Batch Status](gpa-report-integration-guide.md#batch-status) endpoint or the `bulk_upload.completed` event for those.

## Webhook events sent during a bulk import

If you have webhooks set up, a bulk import sends these events, in this order:

| Event | When | Sent |
| --- | --- | --- |
| `bulk_upload.started` | The batch is accepted | Once per batch |
| `gpa_report.created` | A report has been created from a file | Once per report, as each file finishes |
| `bulk_upload.completed` | Every file has finished, or an SFTP upload was rejected | Once per batch |

- Each event is sent only if it is ticked under **Events to Send** in the [Developer dashboard](https://www.scholaro.com/app/developer/webhooks).
- Failed deliveries are not retried automatically (see [Reviewing deliveries](webhooks/overview.md#reviewing-deliveries)).
- `integrity_score` and `integrity_risk` are always `null` in `gpa_report.created` events for bulk-imported reports, because a bulk import doesn't run the Document Integrity Check.

For every field in these payloads, see the [Events Reference](webhooks/events-reference.md#bulk-upload).

!!! note "PDF delivery for bulk-imported reports"
    A bulk import doesn't send `(pdf)` events by itself. If you deliver report PDFs to Slate or Salesforce, the PDFs of bulk-imported reports are sent when someone opens the report in Scholaro Premium, or when an admin uses **Find Missing Reports** or **Resend Reports** in the Developer dashboard. Run **Find Missing Reports** after a batch completes to deliver all of its PDFs. See [PDF Delivery Events](webhooks/pdf-delivery-events.md).

## SFTP processing and webhook delivery

These are two separate steps, and a problem in one doesn't mean a problem in the other:

| | SFTP processing | Webhook delivery |
| --- | --- | --- |
| What it is | Scholaro picks up your upload and creates GPA reports from it | Scholaro sends events about the batch and its reports to your endpoint |
| Direction | Your system → Scholaro | Scholaro → your system |
| Where to check it | The `bulk_upload.completed` event, the [Batch Status](gpa-report-integration-guide.md#batch-status) endpoint, and **GPA Reports** in Scholaro Premium | The delivery list in the [Developer dashboard](https://www.scholaro.com/app/developer/webhooks) |
| What a failure looks like | A `Failed` batch, or `Failed` items with an `error_message` | A delivery with a non-`200` response, or no delivery at all |

For example:

- **Reports were created, but your system didn't receive them.** Processing worked and delivery didn't. Check the delivery list for the response your endpoint returned, and check that the events are ticked.
- **No events arrived and no reports were created.** The upload was probably never processed. See [An SFTP upload did nothing](#an-sftp-upload-did-nothing) below.

## Troubleshooting

### An SFTP upload did nothing

If no `bulk_upload` event arrived and no reports appeared:

- **Check what you uploaded.** Your organization is set up for either ZIP uploads or individual-file uploads (see [SFTP Upload](gpa-report-integration-guide.md#sftp-upload)). An upload in the other format isn't picked up. Contact Scholaro if you're not sure which one your organization uses.
- **Individual files: upload `manifest.csv` last.** Uploading `manifest.csv` is what starts processing.
- **Another batch may still be processing.** Wait for its `bulk_upload.completed` event, then upload again.
- **Check that the events are ticked** under **Events to Send** and that your endpoint is enabled, then look for the reports under **GPA Reports**.

### The batch was rejected

A rejected SFTP upload is recorded as a `Failed` batch, and each item's `error_message` explains why:

| Message | What to do |
| --- | --- |
| `{zip} does not contain manifest.csv.` | Add `manifest.csv` to the root of the ZIP. |
| `manifest.csv has no rows with both file_name and country.` | Check the header row spelling and that every row has a `file_name` and a `country`. |
| `manifest.csv line {n}: a quoted field is never closed.` | A value on that line opens a double quote without closing it. See [CSV formatting rules](gpa-report-integration-guide.md#csv-formatting-rules). |
| `{file} is not in {zip}.` | Add the file to the ZIP, or correct its `file_name` in the manifest. |
| `{file} was not found in the container.` | Individual-file uploads: the file wasn't uploaded within 15 minutes of `manifest.csv`. Upload the file, then upload `manifest.csv` again. |
| `{file} is older than the previous batch. Upload it again with this manifest.` | The file was already in the folder before your last batch. Upload it again, then upload `manifest.csv`. |
| `{file} is not a plain file name at the top of the SFTP folder.` | Put the file in the top-level folder and list it in `file_name` without a folder path. |
| `Batch not started: {n} reports would exceed the subscription limit ({used} of {max} used).` | Upload fewer files, or contact Scholaro about your subscription. |

The Bulk API returns the equivalent message in a `400` response.

### A file failed

| Message | Cause and fix |
| --- | --- |
| `Country {name} not found.` | The `country` value must match the country name used in Scholaro exactly, for example `India`. Check spelling, and that a comma inside another value didn't shift this column (see below). |
| `Only .pdf files are allowed.` | Individual-file uploads: the file isn't a PDF, JPG, JPEG or PNG. |
| `No courses were found.`, `No credential data was found.` or `No reports were created from this file.` | Scholaro couldn't read coursework from the document. Check that the file is a complete, readable transcript. |
| `Processing timed out after 10 minutes.` | Upload the file again in a new batch. If it keeps failing, contact Scholaro with the `batch_id`. |

### Names or other values are in the wrong column

A value that contains a comma, such as `Smith, Jane`, must be in double quotes (`"Smith, Jane"`). Without quotes, the comma starts a new column and every value after it moves one column to the right. That can put the wrong name on a report, put a custom field value in the wrong field, or put a name in `country` so that the file fails. See [CSV formatting rules](gpa-report-integration-guide.md#csv-formatting-rules).

### The Bulk API rejected a ZIP created on a Mac

A ZIP created with macOS **Compress** includes hidden `.DS_Store` files and a `__MACOSX` folder, and Scholaro rejects any ZIP that contains files other than transcripts and `manifest.csv`. Create the ZIP from the command line instead, leaving those files out:

```bash
zip -r bulk-upload.zip . -x ".*" -x "__MACOSX/*"
```

### Reports were created, but no webhook arrived

- Check that `gpa_report.created`, `bulk_upload.started` and `bulk_upload.completed` are ticked under **Events to Send**, and that the endpoint they're assigned to is enabled.
- Check the delivery list in the Developer dashboard for the response your endpoint returned. Deliveries that failed are not retried automatically. Select them and use **Resend Reports**.
- Webhook events are only sent for reports created within the last 14 days.

### PDFs didn't reach Slate or Salesforce

A bulk import doesn't send PDFs by itself. Use **Find Missing Reports** in the Developer dashboard after the batch completes. See [PDF delivery for bulk-imported reports](#webhook-events-sent-during-a-bulk-import).

### The batch seems stuck

Check the batch with the [Batch Status](gpa-report-integration-guide.md#batch-status) endpoint. Each file can take up to 10 minutes. If a batch is still `Processing` after several hours, contact Scholaro. You can also cancel it with **Cancel** next to the **Bulk Import** button in Scholaro Premium.

## Contacting Scholaro

If you need help with a bulk import, include:

- the `batch_id`, if a batch was created
- the name of the ZIP or `manifest.csv` you uploaded, and when you uploaded it (with your time zone)
- the `file_name` and `error_message` of any failed items

## Related docs

- [GPA Report Integration Guide](gpa-report-integration-guide.md)
- [Webhooks Overview](webhooks/overview.md)
- [Events Reference](webhooks/events-reference.md)
- [PDF Delivery Events](webhooks/pdf-delivery-events.md)
