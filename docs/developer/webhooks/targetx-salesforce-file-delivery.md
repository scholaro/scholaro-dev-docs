# TargetX / Salesforce File Delivery

Use this setup to deliver Scholaro files to TargetX or Salesforce through the Scholaro webhook configuration page.

## Prerequisites

Before configuring Scholaro, make sure you have:

- A Salesforce developer account
- An external client app created in Salesforce

## Configure the external client app

If you do not already have an external client app, create one in Salesforce.

### External Client App Policies

Under OAuth policies:

- Enable Client Credentials Flow
- Set "Run As" to your user

The selected Salesforce username will be used for authentication.

### External Client App Settings

Under OAuth scopes, add:

- Manage User Data via APIs (`api`)
- Full access (`full`)
- Perform Requests at any time (`refresh_token`, `offline_access`)

Also make sure Enabled Client Credentials Flow is turned on.

Then copy the following values from the OAuth settings section:

- Consumer Key
- Consumer Secret

## Configure the Scholaro webhook page

In Scholaro, open the developer webhooks page and enter the following values:

- **Username**: your Salesforce username, not your email address
- **Password**: your Salesforce password
- **Hosted Endpoint**: your Salesforce My Domain URL, including `https://`
- **Consumer Key**: the value from the Salesforce external app OAuth settings
- **Consumer Secret**: the value from the Salesforce external app OAuth settings

## Contact matching requirements

Create a Salesforce contact that matches an applicant on one of your GPA reports or digital evaluations.

Name matching should follow this rule:

- The first name must be only the first word of the applicant's name
- The last name must contain all remaining words

### Example

For an applicant named `Jon Franklin James Doe`:

- **First Name**: `Jon`
- **Last Name**: `Franklin James Doe`

## File delivery behavior

Files are not uploaded at the moment a report is created. Scholaro generates the PDF when a ticked `(pdf)` event occurs, then uploads it to the matching contact in Salesforce in the next scheduled run. Runs are twice a day, at 04:00 and 16:00 UTC, so allow up to 12 hours. See [PDF Delivery Events](pdf-delivery-events.md) for which events to tick and what to expect.

## Related docs

- [Webhooks Overview](overview.md)
- [PDF Delivery Events](pdf-delivery-events.md)
- [Bulk Import: Behavior and Troubleshooting](../bulk-import.md)