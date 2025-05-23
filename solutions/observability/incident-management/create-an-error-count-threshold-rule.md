---
mapped_pages:
  - https://www.elastic.co/guide/en/observability/current/apm-error-count-threshold-rule.html
  - https://www.elastic.co/guide/en/serverless/current/observability-create-error-count-threshold-alert-rule.html

navigation_title: "Error count threshold"
---

# Create an error count threshold rule [observability-create-error-count-threshold-alert-rule]


::::{note}

For Observability serverless projects, the **Editor** role or higher is required to create error count threshold rules. To learn more, refer to [Assign user roles and privileges](/deploy-manage/users-roles/cloud-organization/user-roles.md#general-assign-user-roles).

::::


Create an error count threshold rule to alert you when the number of errors in a service exceeds a defined threshold. Threshold rules can be set at different levels: environment, service, transaction type, and/or transaction name.

:::{image} /solutions/images/serverless-alerts-create-rule-error-count.png
:alt: Create rule for error count threshold alert
:screenshot:
:::

::::{tip}
These steps show how to use the **Alerts** UI. You can also create an error count threshold rule directly from any page within **Applications**. Click the **Alerts and rules** button, and select **Create error count rule**. When you create a rule this way, the **Name** and **Tags** fields will be prepopulated but you can still change these.

::::


To create your error count threshold rule:

1. In the Observability UI, go to **Alerts**.
2. Select **Manage Rules** from the **Alerts** page, and select **Create rule**.
3. Enter a **Name** for your rule, and any optional **Tags** for more granular reporting (leave blank if unsure).
4. Select the **Error count threshold** rule type from the APM use case.
5. Select the appropriate **Service**, **Environment**, and **Error Grouping Key** (or leave **ALL** to include all options). Alternatively, you can select **Use KQL Filter** and enter a KQL expression to limit the scope of your rule.
6. Enter the error threshold in **Is Above** (defaults to 25 errors).
7. Define the period to be assessed in **For the last** (defaults to last 5 minutes).
8. Choose how to **Group alerts by**. Every unique value will create an alert.
9. Define the interval to check the rule (for example, check every 1 minute).
10. (Optional) Set up **Actions**.
11. **Save** your rule.


## Add actions [observability-create-error-count-threshold-alert-rule-add-actions]

You can extend your rules with actions that interact with third-party systems, write to logs or indices, or send user notifications. You can add an action to a rule at any time. You can create rules without adding actions, and you can also define multiple actions for a single rule.

To add actions to rules, you must first create a connector for that service (for example, an email or external incident management system), which you can then use for different rules, each with their own action frequency.

:::::{dropdown} Connector types
Connectors provide a central place to store connection information for services and integrations with third party systems. The following connectors are available when defining actions for alerting rules:

* [Cases](https://www.elastic.co/guide/en/kibana/current/cases-action-type.html)
* [D3 Security](https://www.elastic.co/guide/en/kibana/current/d3security-action-type.html)
* [Email](https://www.elastic.co/guide/en/kibana/current/email-action-type.html)
* [{{ibm-r}}](https://www.elastic.co/guide/en/kibana/current/resilient-action-type.html)
* [Index](https://www.elastic.co/guide/en/kibana/current/index-action-type.html)
* [Jira](https://www.elastic.co/guide/en/kibana/current/jira-action-type.html)
* [Microsoft Teams](https://www.elastic.co/guide/en/kibana/current/teams-action-type.html)
* [Observability AI Assistant](https://www.elastic.co/guide/en/kibana/current/obs-ai-assistant-action-type.html)
* [{{opsgenie}}](https://www.elastic.co/guide/en/kibana/current/opsgenie-action-type.html)
* [PagerDuty](https://www.elastic.co/guide/en/kibana/current/pagerduty-action-type.html)
* [Server log](https://www.elastic.co/guide/en/kibana/current/server-log-action-type.html)
* [{{sn-itom}}](https://www.elastic.co/guide/en/kibana/current/servicenow-itom-action-type.html)
* [{{sn-itsm}}](https://www.elastic.co/guide/en/kibana/current/servicenow-action-type.html)
* [{{sn-sir}}](https://www.elastic.co/guide/en/kibana/current/servicenow-sir-action-type.html)
* [Slack](https://www.elastic.co/guide/en/kibana/current/slack-action-type.html)
* [{{swimlane}}](https://www.elastic.co/guide/en/kibana/current/swimlane-action-type.html)
* [Torq](https://www.elastic.co/guide/en/kibana/current/torq-action-type.html)
* [{{webhook}}](https://www.elastic.co/guide/en/kibana/current/webhook-action-type.html)
* [xMatters](https://www.elastic.co/guide/en/kibana/current/xmatters-action-type.html)

::::{note}
Some connector types are paid commercial features, while others are free. For a comparison of the Elastic subscription levels, go to [the subscription page](https://www.elastic.co/subscriptions).

::::


For more information on creating connectors, refer to [Connectors](/deploy-manage/manage-connectors.md).

:::::


:::::{dropdown} Action frequency
After you select a connector, you must set the action frequency. You can choose to create a **Summary of alerts** on each check interval or on a custom interval. For example, you can send email notifications that summarize the new, ongoing, and recovered alerts every twelve hours.

Alternatively, you can set the action frequency to **For each alert** and specify the conditions each alert must meet for the action to run. For example, you can send an email only when the alert status changes to critical.

:::{image} /solutions/images/serverless-alert-action-frequency.png
:alt: Configure when a rule is triggered
:screenshot:
:::

With the **Run when** menu you can choose if an action runs when the threshold for an alert is reached, or when the alert is recovered. For example, you can add a corresponding action for each state to ensure you are alerted when the rule is triggered and also when it recovers.

:::{image} /solutions/images/serverless-alert-apm-action-frequency-recovered.png
:alt: Choose between threshold met or recovered
:screenshot:
:::

:::::


:::::{dropdown} Action variables
Use the default notification message or customize it. You can add more context to the message by clicking the Add variable icon ![Add variable](/solutions/images/serverless-indexOpen.svg "") and selecting from a list of available variables.

:::{image} /solutions/images/serverless-action-variables-popup.png
:alt: Action variables list
:screenshot:
:::

The following variables are specific to this rule type. You can also specify [variables common to all rules](/explore-analyze/alerts-cases/alerts/rule-action-variables.md).

`context.alertDetailsUrl`
:   Link to the alert troubleshooting view for further context and details. This will be an empty string if the `server.publicBaseUrl` is not configured.

`context.environment`
:   The transaction type the alert is created for.

`context.errorGroupingKey`
:   The error grouping key the alert is created for.

`context.errorGroupingName`
:   The error grouping name the alert is created for.

`context.interval`
:   The length and unit of time period where the alert conditions were met.

`context.reason`
:   A concise description of the reason for the alert.

`context.serviceName`
:   The service the alert is created for.

`context.threshold`
:   Any trigger value above this value will cause the alert to fire.

`context.transactionName`
:   The transaction name the alert is created for.

`context.triggerValue`
:   The value that breached the threshold and triggered the alert.

`context.viewInAppUrl`
:   Link to the alert source.

:::::



## Example [create-error-count-threshold-example]

The error count threshold alert triggers when the number of errors in a service exceeds a defined threshold. Because some errors are more important than others, this guide will focus a specific error group ID.

Before continuing, identify the service name, environment name, and error group ID that you’d like to create an error count threshold rule for.

This guide will create an alert for an error group ID based on the following criteria:

* Service: `{your_service.name}`
* Environment: `{your_service.environment}`
* Error Grouping Key: `{your_error.ID}`
* Error count is above 25 errors for the last five minutes
* Group alerts by `service.name` and `service.environment`
* Check every 1 minute
* Send the alert via email to the site reliability team

From any page in **Applications**, select **Alerts and rules** → **Create threshold rule** → **Error count rule**. Change the name of the alert (if you wish), but do not edit the tags.

Based on the criteria above, define the following rule details:

* **Service**: `{your_service.name}`
* **Environment**: `{your_service.environment}`
* **Error Grouping Key**: `{your_error.ID}`
* **Is above:** `25 errors`
* **For the last:** `5 minutes`
* **Group alerts by:** `service.name` `service.environment`
* **Check every:** `1 minute`

Next, select the **Email** connector and click **Create a connector**. Fill out the required details: sender, host, port, etc., and select **Save**.

A default message is provided as a starting point for your alert. You can use the Mustache template syntax (`{{variable}}`) to pass additional alert values at the time a condition is detected to an action. A list of available variables can be accessed by clicking the Add variable icon ![Add variable](/solutions/images/serverless-indexOpen.svg "").

Select **Save**. The alert has been created and is now active!