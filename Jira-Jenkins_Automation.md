# Jenkins-Jira Automation for Keda Scaling

This document outlines the steps to automate Keda scaling using Jenkins and Jira integration.

## A: Jenkins Side Automation

1.  **Select `keda_creation_automation` Job for Triggering Remotely**

    *   This job will be triggered by a webhook from Jira when specific conditions are met.

2.  **Create API Token for Jenkins**

    *   Navigate to your Jenkins user profile to create an API token.
    *   Steps:
        *   Top Right User Profile --> Configure
        *   API Token --> Add New Token
        *   Generate the token and save it securely.  **Note:** You will only see the token value once.

## B: Jira Side Automation

1.  **Create Custom Fields**

    *   Navigate to the Custom Fields section in Jira settings.
    *   Steps:
        *   Jira settings --> Issues --> Custom fields --> Create Custom Field
            *   OR
        *   Project Settings --> Add Custom Field --> Create Custom Field
    *   Create the following custom fields:
        *   `SCALED_OBJECT_NAME` (Text Field) - Custom Field ID: `customfield_10091`
        *   `SERVICE_NAME` (Text Field) - Custom Field ID: `customfield_10092`
        *   `MIN_REPLICAS` (Number Field) - Custom Field ID: `customfield_10093`
        *   `MAX_REPLICAS` (Number Field) - Custom Field ID: `customfield_10094`
        *   `THRESHOLD` (Number Field) - Custom Field ID: `customfield_10095`
        *   `NAMESPACE` (Text Field) - Custom Field ID: `customfield_10096`
        *   `OPCO_NAME` (Text Field) - Custom Field ID: `customfield_10097`

2.  **Add Fields to Project**

    *   Associate the custom fields with your Jira project.
    *   Steps:
        *   Project Settings --> Fields --> Add fields

3.  **Add Fields to Request Types**

    *   Make the custom fields visible and editable in your Jira request types.
    *   Steps:
        *   Project Settings --> Request types --> Select the request type --> Add fields

4.  **Create Automation Rule**

    *   Configure an automation rule to trigger the Jenkins job when a work item transitions to a specific status.
    *   Steps:
        *   Project Settings --> Automation --> Create rule
        *   **Trigger:**
            *   Select "Work item transitioned"
            *   Select "From Status" and "To Status" (e.g., From "Open" to "In Progress")
            *   Click "Save"
        *   **Condition:**
            *   Add a component --> "Work item fields condition"
            *   "Status equals" (choose the "To Status" selected above, e.g., "In Progress")
            *   Click "Save"
        *   **Action:**
            *   Add a component --> "Send web request"
            *   **Web request URL:**
                ```
                http://54.163.37.253:8080/job/keda_pipeline/buildWithParameters?token=keda-trigger-token&SCALED_OBJECT_NAME={{issue.fields.customfield_10091}}&SERVICE_NAME={{issue.fields.customfield_10092}}&MIN_REPLICAS={{issue.fields.customfield_10093}}&MAX_REPLICAS={{issue.fields.customfield_10094}}&THRESHOLD={{issue.fields.customfield_10095}}&NAMESPACE={{issue.fields.customfield_10096}}&OPCO_NAME={{issue.fields.customfield_10097}}
                ```
            *   **HTTP Method:** `POST`
            *   **Request Body:**  Leave empty
            *   **Headers:** (Optional, but recommended for security)
                *   `Content-Type: application/json`
            *   Click "Save"
        *   Name your rule and click "Turn it on".

