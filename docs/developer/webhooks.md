# Webhooks

Use Scholaro webhooks to receive data when GPA reports and digital evaluations are created or updated, and to deliver report PDFs to external systems.

The Webhooks section covers endpoint setup, every event and its payload, signature verification, PDF delivery, and setup guidance for downstream systems.

## In this section

### Overview

Start here: how to set up and register an endpoint, and how deliveries behave.

### Events Reference

Lists every webhook event, when it is sent, and the fields in its payload.

### Signature Verification

Explains how to check the `X-Scholaro-Signature` header, with code examples.

### PDF Delivery Events

Explains which GPA report and digital evaluation events support PDF delivery and how nightly processing works.

### TargetX / Salesforce File Delivery

Covers the Salesforce-specific setup required for delivering files through the Scholaro webhook configuration.

## Related docs

- [General Overview](webhooks/overview.md)
- [Events Reference](webhooks/events-reference.md)
- [Signature Verification](webhooks/signature-verification.md)
- [PDF Delivery Events](webhooks/pdf-delivery-events.md)
- [TargetX / Salesforce File Delivery](webhooks/targetx-salesforce-file-delivery.md)
- [Slate Integration Setup](integrations/slate-integration-setup.md)