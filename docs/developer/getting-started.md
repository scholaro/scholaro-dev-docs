# Getting Started

This page is the fastest starting point for institutional developers integrating Scholaro GPA reports into their own systems.

> Note: If you need event-based delivery or downstream system setup, see the [Webhooks](webhooks.md) section and the [Slate Integration Setup](integrations/slate-integration-setup.md) guide.

## Choose an integration method

| Method | Best for | Response pattern |
| --- | --- | --- |
| API (Single Report) | Real-time processing, one transcript at a time | Immediate request and response |
| Bulk API Upload | Batch uploads via ZIP | Asynchronous batch processing |
| SFTP Upload | Automated workflows and high-volume pipelines | File drop workflow |

!!! note
    All three methods use the same underlying processing and produce the same GPA report output.

## Before you begin

Make sure you have:

- Institutional Premium access
- Your API key or SFTP credentials from Scholaro
- Transcript files in supported formats
- A workflow for receiving reports or status updates

## Authentication

For API requests, include your API key in the `X-API-Key` header.

```http
X-API-Key: your_api_key_here
```

Developer dashboard:

```text
https://www.scholaro.com/app/developer/webhooks
```

## Which method should you use?

### Single API

Use this when you:

- need fast validation during development
- send one transcript at a time
- want the simplest implementation path

### Bulk API

Use this when you:

- already receive transcript files in batches
- want to upload a ZIP plus manifest file
- can poll for completion or process results later

See [Bulk Import: Behavior and Troubleshooting](bulk-import.md) for what happens after an upload.

### SFTP

Use this when you:

- already use file-based vendor integrations
- need a low-touch operational workflow
- expect higher volume and routine uploads

## Recommended rollout path

1. Start with the Single API.
2. Validate inputs and report outputs.
3. Move to Bulk API if you need batching.
4. Add webhooks for downstream automation.
5. Use SFTP only if a file-based workflow is a better fit than API calls.

## Fast developer checklist

- [ ] Confirm access and credentials
- [ ] Test one sample transcript with the Single API
- [ ] Review the GPA report response shape
- [ ] Decide whether you need batch processing
- [ ] Decide whether you want webhook notifications

## Related pages

- [GPA Report Integration Guide](gpa-report-integration-guide.md)
- [Bulk Import: Behavior and Troubleshooting](bulk-import.md)
- [Webhooks](webhooks.md)
- [Slate Integration Setup](integrations/slate-integration-setup.md)
