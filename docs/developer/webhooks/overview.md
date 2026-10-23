# Webhooks Overview

Use Scholaro webhooks to receive data as soon as GPA reports and digital evaluations are created or updated, and to have report PDFs delivered to systems such as Slate and Salesforce.

Scholaro sends two kinds of events:

- **JSON events** (the `(data)` and `bulk_upload` events) post the event as JSON to your endpoint as soon as it happens. They are covered on this page.
- **File events** (the `(pdf)` events and `evaluation_report.created`) deliver report files to your endpoint on a schedule. See [PDF Delivery Events](pdf-delivery-events.md).

## Steps to receive webhooks

1. **Choose your events.** Decide which events you want to receive and which payloads you will parse. See the [Events Reference](events-reference.md) for every event, when it is sent, and its fields.
2. **Create an endpoint.** Set up a publicly accessible HTTPS URL on your server that accepts `POST` requests with a JSON body.
3. **Handle requests and return `200`.** Parse each event and respond with HTTP `200 OK` within 30 seconds. Scholaro records any other status code, or no response, as a failed delivery.
4. **Verify the signature (optional, recommended).** Use the `X-Scholaro-Signature` header to check that each request came from Scholaro. See [Signature Verification](signature-verification.md).
5. **Register your endpoint.** In the Scholaro [Developer dashboard](https://www.scholaro.com/app/developer/webhooks):
    1. Enter your URL under **Hosted Endpoint**.
    2. Under **Events to Send**, tick the events you want.
    3. If your endpoint requires it, turn on basic authentication and enter a username and password. Scholaro then sends them in an `Authorization: Basic` header.
    4. Turn the endpoint on, then click **Save**.

The **Signing Secret** for signature verification is shown on the same page.

## Requests

Each data event is a separate HTTP `POST` request:

- `Content-Type: application/json; charset=utf-8`
- `X-Scholaro-Signature`: the request signature (see [Signature Verification](signature-verification.md))
- `Authorization: Basic ...`, only if basic authentication is turned on

The body is a JSON event object:

```json
{
  "object": "event",
  "type": "gpa_report.created",
  "data": {
    "...": "..."
  }
}
```

`type` tells you which event was sent, and `data` holds the report. See the [Events Reference](events-reference.md) for every field.

## Delivery behavior

- **Created events are sent once per report.** **Updated events are sent every time** the report changes in a way that triggers them.
- **Only recent reports are sent.** GPA report and digital evaluation events are only sent for reports created within the last 14 days.
- **Failed deliveries are not retried automatically.** The Developer dashboard lists recent deliveries with your endpoint's response. Use **Resend Reports** to send selected reports again, or **Find Missing Reports** to send created events for recent reports that don't have one yet.
- **You can use a second endpoint.** You can add a second endpoint and choose which endpoint receives each event, for example report data to one system and PDFs to another.

!!! tip
    Return `200` as soon as you have received the request, and do any slow processing afterwards. Log each event and make your handler safe to run twice for the same report, because resends and updated events can deliver the same report more than once.

## Reviewing deliveries

The [Developer dashboard](https://www.scholaro.com/app/developer/webhooks) lists the JSON events sent to your endpoints in the last 14 days, newest first. It is available to admins of your organization.

| Column | Shows |
| --- | --- |
| **Report ID** | The GPA report or digital evaluation the event was about. `N/A` for `bulk_upload` events. |
| **Event** | The event that was sent. |
| **Endpoint** | The endpoint it was sent to. |
| **Created On** | When it was sent. |
| **Response** | The status code your endpoint returned, for example `200 OK` or `500 ERR`. A timeout is shown as `408`. |

Select a delivery to see the request Scholaro sent and the response it got.

- **Resend Reports** (or **Re-send** on a single delivery) sends the selected report events again. The event must still be ticked under **Events to Send**. `bulk_upload` events can't be re-sent.
- **Find Missing Reports** sends `created` events, data and PDF, for reports created in the last 14 days. It skips the data event for any report that already has a delivery in the list, even a failed one, so use **Resend Reports** for failed deliveries.
- PDF deliveries (the `(pdf)` events) aren't listed. See [PDF Delivery Events](pdf-delivery-events.md).

For bulk imports, see [SFTP processing and webhook delivery](../bulk-import.md#sftp-processing-and-webhook-delivery).

## Related docs

- [Events Reference](events-reference.md)
- [Bulk Import: Behavior and Troubleshooting](../bulk-import.md)
- [Signature Verification](signature-verification.md)
- [PDF Delivery Events](pdf-delivery-events.md)
- [TargetX / Salesforce File Delivery](targetx-salesforce-file-delivery.md)
- [Slate Integration Setup](../integrations/slate-integration-setup.md)
