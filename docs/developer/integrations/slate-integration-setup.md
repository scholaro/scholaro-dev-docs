# Slate Integration Setup

This guide explains how to configure Slate so Scholaro can send documents directly into your Slate instance.

The setup consists of four parts:

1. Create a service account in Slate
2. Create a source format in Slate
3. Configure the webhook in Scholaro
4. Remap the source format after the first file is received

## Related Slate documentation

- [User Accounts](https://knowledge.technolutions.com/hc/en-us/articles/360026832392-User-Accounts#types-0-0)
- [Importing Data with Web Services](https://knowledge.technolutions.com/hc/en-us/articles/360033201452-Importing-Data-with-Web-Services#pull-data-from-a-web-service-into-slate-0-0)
- [Creating a Custom Source Format](https://knowledge.technolutions.com/hc/en-us/articles/360033287711-Creating-a-Custom-Source-Format)
- [Upload Dataset Stages](https://knowledge.technolutions.com/hc/en-us/articles/360033699511)

---

## 1. Create the service account in Slate

In Slate, go to **Database -> User Permissions**.

![User Permissions navigation](../images/slate/user-permissions-navigation.png)

If needed, continue into the user permissions area.

![User Permissions navigation step 2](../images/slate/user-permissions-navigation-2.png)

Click **New User**.

![New User button](../images/slate/new-user-button.png)

Create the service account with the following values:

- **First Name**: Scholaro
- **Last Name**: Service Account
- **Email**: Use a monitored email address that your team can access. You will use this same email again later.
- **User Type**: Service Account
- **User ID**: scholaro
- **Password**: Use a strong password and store it securely. You will need it again for the webhook configuration.

Example service account form:

![Service account form](../images/slate/service-account-form.png)

!!! warning
    Save the email address and password in a secure place. You will need both when configuring the webhook in Scholaro.

---

## 2. Create the source format in Slate

In Slate, go to **Database -> Source Formats**.

![Source Formats navigation step 1](../images/slate/source-formats-navigation-1.png)

Continue into the Source Formats area if needed.

![Source Formats navigation step 2](../images/slate/source-formats-navigation-2.png)

Click **New Source Format**.

![New Source Format button](../images/slate/new-source-format-button.png)

### General tab

On the **General** tab, enter the following values. Do **not** click **Save** yet.

- **Status**: Active
- **Name**: Scholar DIP
- **Format**: DIP
- **Type**: One-Time / Differential
- **Remap As Of Date**: Today's date in `yyyy-MM-dd` format
- **Remap Active**: Inactive
- **Scope**: Person/Dataset Record
- **Dataset**: Person/Application Records
- **Unsafe**: Unsafe
- **Hide**: Create Source Interactions
- **Disable Update Queue**: Allow records to enter update queue upon import (allow rules to fire)
- **Update Only**: Allow record creation
- **Dedupe Records**: Enable record creation for all rows
- **Notification**: Failures only
- **Notification Email**: Use the same monitored email address used for the service account
- **Read Permission**: Optional
- **Upload/Build Permission**: Optional
- **Source Metadata**: Optional

!!! note
    You may set **Notification** to **Successes and Failures** if you want to receive an email every time the import runs.

Example of the General tab:

![Source Format General tab](../images/slate/source-format-general-tab.png)

If you already clicked **Save**, that is fine. Open the source format again and click **Edit** to continue.

![Source Format Edit button](../images/slate/source-format-edit-button.png)

### Format Definition tab

Go to the **Format Definition** tab and enter the following XML, then click **Save**.

![Format Definition XML](../images/slate/format-definition-xml.png)

```xml
<layout b="&#x9;" h="1" type="dip" index="index*.txt" />
```

After saving, open the **Web Services** menu for the source format and locate the **Standard POST URL**.

![Standard POST URL](../images/slate/standard-post-url.png)

If your screen shows an alternate view, use the **Standard POST URL** shown there.

![Standard POST URL alternate view](../images/slate/standard-post-url-1.png)

!!! tip
    Copy the **Standard POST URL** somewhere safe. You will enter it in Scholaro in the next section.

## 3. Configure the webhook in Scholaro

Log in to the Scholaro webhook page:

`https://www.scholaro.com/app/developer/webhooks`

Enter the following values:

- **URL**: Paste the **Standard POST URL** from the Slate source format
- **Events to send**: Tick an event for each kind of file you want in Slate (see the table below)
- **Authentication**: Use basic authentication
  - **Username**: `scholaro`
  - **Password**: The service account password you created in Slate
- **Enabled**: Checked

| To receive in Slate | Tick |
| --- | --- |
| Credential evaluations ordered from Scholaro | `evaluation_report.created` |
| GPA report PDFs | `gpa_report.created (pdf)`, and `gpa_report.updated (pdf)` if you also want a new PDF after a report changes |
| Digital evaluation PDFs | `digital_evaluation.created (pdf)`, and `digital_evaluation.updated (pdf)` if you also want a new PDF after an evaluation changes |

The `(data)` events send JSON, not files, so the DIP source format on this endpoint can't import them. If you also want the report data, add a second endpoint for the `(data)` events (see [Webhooks Overview](../webhooks/overview.md#delivery-behavior)).

A PDF is only generated if its `(pdf)` event is already ticked when the report is created or updated. See [PDF Delivery Events](../webhooks/pdf-delivery-events.md#what-you-need-to-set-up).

Example webhook configuration:

![Scholaro webhook configuration](../images/slate/scholaro-webhook-config.png)

Once saved, Scholaro will be able to send ZIP files into your Slate instance.

### Delivery schedule

Scholaro doesn't send a file to Slate at the moment a report is created. Files are sent in two scheduled runs a day:

| Run | UTC | US Central, standard time | US Central, daylight saving time |
| --- | --- | --- | --- |
| Morning | 16:00 | 10:00 AM | 11:00 AM |
| Evening | 04:00 | 10:00 PM | 11:00 PM |

- Each run sends one ZIP file with everything that is waiting for your organization. If nothing is waiting, no file is sent.
- A report created just after a run waits for the next one, so a PDF can reach Slate up to 12 hours after its report was created. A run with many PDFs can take several minutes.
- Slate then imports the file on its own schedule, so allow extra time before the material appears on a record.
- If Slate doesn't accept a ZIP file, Scholaro sends its PDFs again on each run for up to 7 days.
- If a report changes more than once between runs, only its latest PDF is sent.

For bulk imports through SFTP or the Bulk API, see [PDF delivery for bulk-imported reports](../bulk-import.md#webhook-events-sent-during-a-bulk-import).

## 4. Remap the source format after the first file is received

After Scholaro sends the first ZIP file, return to the **Scholar DIP** source format in Slate.

!!! note "When the first ZIP file arrives"
    Saving the webhook doesn't send a file. Expect the first ZIP file at the next scheduled run after a report or evaluation is created, which can be up to 12 hours later. See [Delivery schedule](#delivery-schedule).

You should see a successful web service post from `scholaro`. Once the file is ready for mapping, Slate will show imported rows and the **Remap** option will become available.

![Imported rows and Remap](../images/slate/imported-rows-remap.png)

### Field mappings

Click **Remap** and configure the following field mappings:

- `first` -> **Record > First Name**
- `last` -> **Record > Last Name**
- `email` -> **Record > Email**
- `type` -> **Material > Material Code**
- `Filename` -> **Material > Material Filename**

Example field mappings screen:

![Field mappings](../images/slate/field-mappings.png)

### Custom field columns

ZIP files for **GPA Reports** and **Digital Evaluations** also include one extra column in `index.txt` for each custom field your organization defines in Scholaro. Each column is named after the custom field and appears in the field's sort order, after the `Filename` column.

You can map these columns in the same **Remap** screen. For example, if your organization collects an **Application ID** custom field, map that column to the Slate field you use as the record key so files attach to the right record.

!!! note
    The `email` column is the email address of the Scholaro user who created the report, not the applicant's email. A custom field such as an application ID is usually a more reliable way to match records.

Keep the following in mind:

- A custom field that is only shown on one report type is blank on rows for the other report type.
- Renaming a custom field in Scholaro renames its column. After a rename, open the source format and **Remap** the new column in Slate.

### Prompt value mappings

Go to the **Prompt Value Mappings** screen and select the `type` source field.

![Prompt type field](../images/slate/prompt-type-field.png)

Map the incoming document types to the correct **Material Types** in your Slate instance, then save.

For GPA report and digital evaluation PDFs, the `type` value is one of:

- `GPA Report (created)`
- `GPA Report (updated)`
- `Digital Evaluation (created)`
- `Digital Evaluation (updated)`

Map every value for the events you ticked. If you tick another event later, return to this screen and map its value too.

![Prompt material type mapping](../images/slate/prompt-material-type-mapping.png)

When finished, return to the source format.

![Return to source format](../images/slate/return-to-source-format.png)

### Activate remap

Open the source format and click **Edit**.

![Source Format Edit button](../images/slate/source-format-edit-button.png)

Set **Remap Active** to **Active**, then click **Save**.

![Remap Active](../images/slate/remap-active.png)

At that point, the source format should be fully configured for ongoing imports.

## Summary

To complete the integration successfully:

- Create a Slate service account for Scholaro
- Create the **Scholar DIP** source format
- Use the source format's **Standard POST URL** in the Scholaro webhook
- Wait for the first ZIP file to arrive
- Complete field and prompt mappings
- Set **Remap Active** to **Active**

Once these steps are complete, Scholaro can send supported files directly into your Slate environment.

## Related docs

- [PDF Delivery Events](../webhooks/pdf-delivery-events.md)
- [Events Reference](../webhooks/events-reference.md)
- [Webhooks Overview](../webhooks/overview.md)
- [Bulk Import: Behavior and Troubleshooting](../bulk-import.md)