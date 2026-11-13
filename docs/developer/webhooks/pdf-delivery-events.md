# PDF Delivery Events

Scholaro supports PDF delivery workflows for specific GPA report and digital evaluation events.

## Supported events

The following events are supported for PDF delivery:

- `gpa_report.created (pdf)`
- `gpa_report.updated (pdf)`
- `digital_evaluation.created (pdf)`
- `digital_evaluation.updated (pdf)`

These are listed separately from the `(data)` events on the Scholaro webhook page. See the [Events Reference](events-reference.md) for every event.

`evaluation_report.created` also delivers files: the documents for a credential evaluation ordered from Scholaro, sent twice a day after the evaluation is completed.

## Created and updated events

| Event | A PDF is generated when | How often |
| --- | --- | --- |
| `gpa_report.created (pdf)` | A GPA report is created in Scholaro, or by a bulk import through SFTP or the Bulk API | Once per report |
| `gpa_report.updated (pdf)` | A different grading scale is applied to the report, or the applicant name or report name is edited in the GPA Reports list | Every time |
| `digital_evaluation.created (pdf)` | A digital evaluation is created | Once per evaluation |
| `digital_evaluation.updated (pdf)` | A GPA report is linked to or unlinked from the evaluation, one of its custom fields is edited, or the applicant name or evaluation name is edited in the Evaluations list | Every time |

- **Tick only the created event** to receive each report once, as it was when it was created. Later changes are not sent.
- **Tick the updated event as well** to receive a new PDF after each change. The new PDF has the same file name as the earlier one, so your system should treat it as a replacement.
- Editing grades or courses on a GPA report does not generate a new PDF.
- A report created with the single-report [GPA Report API](../gpa-report-integration-guide.md) doesn't get a PDF at the moment it is created. Its created PDF is generated when the report is first opened in Scholaro Premium, or when you use **Find Missing Reports**.

## What you need to set up

A PDF is only generated and delivered if all of these are true:

- The `(pdf)` event is ticked under **Events to Send** in the [Developer dashboard](https://www.scholaro.com/app/developer/webhooks). The `(data)` and `(pdf)` events are ticked separately: ticking `gpa_report.created (data)` does not deliver PDFs.
- The event was already ticked when the report was created or updated. Scholaro doesn't generate PDFs for earlier events when you tick it later. For created events from the last 14 days, use **Find Missing Reports** to generate them.
- The endpoint the event is assigned to is turned on and set up for a system that accepts files: [Slate](../integrations/slate-integration-setup.md) or [TargetX / Salesforce](targetx-salesforce-file-delivery.md). If you use two endpoints, PDFs go to the endpoint the `(pdf)` event is assigned to.

## How it works

PDF delivery happens in two steps, so a PDF does not reach your system at the moment the report is created.

1. **The PDF is generated.** When one of the supported events occurs, Scholaro creates the PDF and holds it for delivery. Nothing is sent to your system yet.
2. **The PDF is sent.** Twice a day, at 04:00 and 16:00 UTC, Scholaro sends every PDF that is waiting for your organization to your endpoint. For Slate, they are packaged into one ZIP file per run.

The run times are fixed in UTC, so the local time moves by an hour with daylight saving time. In US Central Time, the runs are at 10:00 AM and 10:00 PM during standard time, and at 11:00 AM and 11:00 PM during daylight saving time. A run with many PDFs can take several minutes to finish.

A report created just after a run waits for the next one, so allow up to 12 hours. The matching `(data)` event is still sent as soon as the report is created or updated, so the report data usually arrives hours before its PDF.

## What to expect

- **Only the latest PDF of a report is sent.** If a report has more than one PDF waiting, for example because it was created and then updated before the run, or updated several times, Scholaro sends the newest PDF once and discards the older ones. If the created PDF had not been sent yet, the delivery is still labelled as created.
- **A rejected delivery is tried again.** If your endpoint doesn't accept a delivery, Scholaro tries again on each run for up to 7 days after the PDF was generated. Scholaro removes its copy of the PDF once your endpoint has accepted it.
- **PDF deliveries aren't in the delivery list.** The delivery list in the Developer dashboard shows `(data)` events only.
- **Find Missing Reports doesn't send PDFs early.** It generates the PDF for any report from the last 14 days that doesn't have one yet. Those PDFs are still sent in the next scheduled run.

## Slate delivery behavior

For Slate integrations, the existing integration flow remains the same. Enabling PDF delivery adds more files to the process rather than replacing the existing setup.

## Related docs

- [Webhooks Overview](overview.md)
- [Events Reference](events-reference.md)
- [Slate Integration Setup](../integrations/slate-integration-setup.md)
- [TargetX / Salesforce File Delivery](targetx-salesforce-file-delivery.md)
- [Bulk Import: Behavior and Troubleshooting](../bulk-import.md)