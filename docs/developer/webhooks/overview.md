# Webhooks Overview

Use Scholaro webhooks to receive data as soon as GPA reports and digital evaluations are created or updated, and to have report PDFs delivered to systems such as Slate and Salesforce.

Scholaro sends two kinds of events:

- **Data events** post the report's data as JSON to your endpoint as soon as the event happens. They are covered on this page.
- **PDF events** save a copy of the report, and a nightly process delivers the file. See [PDF Delivery Events](pdf-delivery-events.md).

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

## Related docs

- [Events Reference](events-reference.md)
- [Signature Verification](signature-verification.md)
- [PDF Delivery Events](pdf-delivery-events.md)
- [TargetX / Salesforce File Delivery](targetx-salesforce-file-delivery.md)
- [Slate Integration Setup](../integrations/slate-integration-setup.md)
