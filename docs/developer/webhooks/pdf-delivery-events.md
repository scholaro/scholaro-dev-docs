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

## How it works

PDF delivery happens in two steps, so a PDF does not reach your system at the moment the report is created.

1. **The PDF is generated.** When one of the supported events occurs, Scholaro creates the PDF and holds it for delivery. Nothing is sent to your system yet.
2. **The PDF is sent.** Twice a day, about 12 hours apart (currently 04:00 and 16:00 UTC), Scholaro sends every PDF that is waiting for your organization to your endpoint. For Slate, they are packaged into one ZIP file per run.

A report created just after a run waits for the next one, so allow up to 12 hours. The matching `(data)` event is still sent as soon as the report is created or updated, so the report data usually arrives hours before its PDF.

## What to expect

- **Each report is sent once per run.** If a report has more than one PDF waiting, for example because it was created and then updated before the run, only the newest PDF is sent.
- **A rejected delivery is tried again.** If your endpoint doesn't accept a delivery, Scholaro tries again on each run for up to 7 days after the PDF was generated. Scholaro removes its copy of the PDF once your endpoint has accepted it.
- **PDF deliveries aren't in the delivery list.** The delivery list in the Developer dashboard shows `(data)` events only.
- **Find Missing Reports doesn't send PDFs early.** It generates the PDF for any report from the last 14 days that doesn't have one yet. Those PDFs are still sent in the next scheduled run.

## Slate delivery behavior

For Slate integrations, the existing integration flow remains the same. Enabling PDF delivery adds more files to the process rather than replacing the existing setup.

## Related docs

- [Webhooks Overview](overview.md)
- [Events Reference](events-reference.md)
- [Slate Integration Setup](../integrations/slate-integration-setup.md)