---
title: Set a table's log data plan in Azure Monitor Logs
description: Learn how to configure the table log data plan to optimize log ingestion and retention costs in Azure Monitor Logs.
author: guywi-ms
ms.author: guywild
ms.reviewer: adi.biran
ms.topic: how-to
ms.date: 11/09/2022
---

# Set a table's log data plan in Azure Monitor Logs

Azure Monitor Logs offers two log data plans that let you reduce log ingestion and retention costs and take advantage of Azure Monitor's advanced features and analytics capabilities based on your needs: 

- The default **Analytics** log data plan provides full analysis capabilities and makes log data available for queries, Azure Monitor features, such as alerts, and use by other services. 
- The **Basic** log data plan lets you save on the cost of ingesting and storing high-volume verbose logs in your Log Analytics workspace for debugging, troubleshooting, and auditing, but not for analytics and alerts. 

This article describes Azure Monitor's log data plans and explains how to configure the log data plan of the tables in your Log Analytics workspace.

> [!IMPORTANT]
> You can switch a table's plan once a week.<br/> The Basic Logs feature isn't available for workspaces in [legacy pricing tiers](cost-logs.md#legacy-pricing-tiers).

## Compare the Basic and Analytics log data plans 

The following table summarizes the two plans. 

| Category | Analytics | Basic |
|:---|:---|:---|
| Ingestion | Cost for ingestion. | Reduced cost for ingestion. |
| Log queries | No extra cost. Full query capabilities. | Extra cost.<br>[Subset of query capabilities](basic-logs-query.md#limitations). |
| Retention |  Configure retention from 30 days to 730 days. | Retention fixed at eight days. |
| Alerts | Supported. | Not supported. |

> [!NOTE]
> The Basic log data plan isn't available for workspaces in [legacy pricing tiers](cost-logs.md#legacy-pricing-tiers).

## When should I use Basic Logs?
The decision whether to configure a table for Basic Logs is based on the following criteria:

- The table currently [supports Basic Logs](#which-tables-support-basic-logs).
- You don't require more than eight days of data retention for the table.
- You only require basic queries of the data using a limited version of the query language.
- The cost savings for data ingestion over a month exceed the expected cost for any expected queries

## Which tables support Basic Logs?

By default, all tables in your Log Analytics workspace are Analytics tables, and they're available for query and alerts. You can currently configure the following tables for Basic Logs:

| Table | Details|
|:---|:---|
| Custom tables | All custom tables created with or migrated to the [data collection rule (DCR)-based logs ingestion API.](logs-ingestion-api-overview.md) |
| [ACSCallRecordingSummary](/azure/azure-monitor/reference/tables/acscallrecordingsummary) | Communication Services recording summary logs. |
| [ACSRoomsIncomingOperations](/azure/azure-monitor/reference/tables/acsroomsincomingoperations) | Communication Services Rooms incoming requests operations. |
| [AppTraces](/azure/azure-monitor/reference/tables/apptraces) | Application Insights Freeform traces. |
| [AMSKeyDeliveryRequests](/azure/azure-monitor/reference/tables/AMSKeyDeliveryRequests) | Azure Media Services HTTP request details for key, or license acquisition. |
| [AMSMediaAccountHealth](/azure/azure-monitor/reference/tables/AMSMediaAccountHealth) | Azure Media Account Health Status. |
| [ContainerAppConsoleLogs](/azure/azure-monitor/reference/tables/containerappconsoleLogs) | Azure Container Apps logs, generated within a Container Apps environment. |
| [ContainerLogV2](/azure/azure-monitor/reference/tables/containerlogv2) | Used in [Container insights](../containers/container-insights-overview.md) and includes verbose text-based log records. |

> [!NOTE]
> Tables created with the [Data Collector API](data-collector-api.md) don't support Basic Logs.

## Set a table's log data plan

# [Portal](#tab/portal-1)

To configure a table for Basic Logs or Analytics Logs in the Azure portal:

1. From the **Log Analytics workspaces** menu, select **Tables**.

    The **Tables** screen lists all the tables in the workspace.

1. Select the context menu for the table you want to configure and select **Manage table**.

    :::image type="content" source="media/basic-logs-configure/log-analytics-table-configuration.png" lightbox="media/basic-logs-configure/log-analytics-table-configuration.png" alt-text="Screenshot that shows the Manage table button for one of the tables in a workspace.":::

1. From the **Table plan** dropdown on the table configuration screen, select **Basic** or **Analytics**.

    The **Table plan** dropdown is enabled only for [tables that support Basic Logs](#which-tables-support-basic-logs).

    :::image type="content" source="media/basic-logs-configure/log-analytics-configure-table-plan.png" lightbox="media/basic-logs-configure/log-analytics-configure-table-plan.png" alt-text="Screenshot that shows the Table plan dropdown on the table configuration screen.":::

1. Select **Save**.

# [API](#tab/api-1)

To configure a table for Basic Logs or Analytics Logs, call the **Tables - Update** API:

```http
PATCH https://management.azure.com/subscriptions/<subscriptionId>/resourcegroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/tables/<tableName>?api-version=2021-12-01-preview
```

> [!IMPORTANT]
> Use the bearer token for authentication. Learn more about [using bearer tokens](https://social.technet.microsoft.com/wiki/contents/articles/51140.azure-rest-management-api-the-quickest-way-to-get-your-bearer-token.aspx).

**Request body**

|Name | Type | Description |
| --- | --- | --- |
|properties.plan | string  | The table plan. Possible values are `Analytics` and `Basic`.|

**Example**

This example configures the `ContainerLogV2` table for Basic Logs.

Container insights uses `ContainerLog` by default. To switch to using `ContainerLogV2` for Container insights, [enable the ContainerLogV2 schema](../containers/container-insights-logging-v2.md) before you convert the table to Basic Logs.

**Sample request**

```http
PATCH https://management.azure.com/subscriptions/ContosoSID/resourcegroups/ContosoRG/providers/Microsoft.OperationalInsights/workspaces/ContosoWorkspace/tables/ContainerLogV2?api-version=2021-12-01-preview
```

Use this request body to change to Basic Logs:

```http
{
    "properties": {
        "plan": "Basic"
    }
}
```

Use this request body to change to Analytics Logs:

```http
{
    "properties": {
        "plan": "Analytics"
    }
}
```

**Sample response**

This sample is the response for a table changed to Basic Logs:

Status code: 200

```http
{
    "properties": {
        "retentionInDays": 8,
        "totalRetentionInDays": 30,
        "archiveRetentionInDays": 22,
        "plan": "Basic",
        "lastPlanModifiedDate": "2022-01-01T14:34:04.37",
        "schema": {...}        
    },
    "id": "subscriptions/ContosoSID/resourcegroups/ContosoRG/providers/Microsoft.OperationalInsights/workspaces/ContosoWorkspace",
    "name": "ContainerLogV2"
}
```

# [CLI](#tab/cli-1)

To configure a table for Basic Logs or Analytics Logs, run the [az monitor log-analytics workspace table update](/cli/azure/monitor/log-analytics/workspace/table#az-monitor-log-analytics-workspace-table-update) command and set the `--plan` parameter to `Basic` or `Analytics`.

For example:

- To set Basic Logs:

    ```azurecli
    az monitor log-analytics workspace table update --subscription ContosoSID --resource-group ContosoRG  --workspace-name ContosoWorkspace --name ContainerLogV2  --plan Basic
    ```

- To set Analytics Logs:

    ```azurecli
    az monitor log-analytics workspace table update --subscription ContosoSID --resource-group ContosoRG  --workspace-name ContosoWorkspace --name ContainerLogV2  --plan Analytics
    ```

---

## View a table's log data plan

# [Portal](#tab/portal-2)

To check table configuration in the Azure portal, you can open the table configuration screen, as described in [Set table configuration](#set-a-tables-log-data-plan).

Alternatively:

1. From the **Azure Monitor** menu, select **Logs** and select your workspace for the [scope](scope.md). See the [Log Analytics tutorial](log-analytics-tutorial.md#view-table-information) for a walkthrough.
1. Open the **Tables** tab, which lists all tables in the workspace.

    Basic Logs tables have a unique icon:
    
    :::image type="content" source="media/basic-logs-configure/table-icon.png" alt-text="Screenshot that shows the Basic Logs table icon in the table list." lightbox="media/basic-logs-configure/table-icon.png":::

    You can also hover over a table name for the table information view, which indicates whether the table is configured as Basic Logs:

    :::image type="content" source="media/basic-logs-configure/table-info.png" alt-text="Screenshot that shows the Basic Logs table indicator in the table details." lightbox="media/basic-logs-configure/table-info.png":::

# [API](#tab/api-2)

To check the configuration of a table, call the **Tables - Get** API:

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}/tables/{tableName}?api-version=2021-12-01-preview
```

**Response body**

|Name | Type | Description |
| --- | --- | --- |
|properties.plan | string  | The table plan. Either `Analytics` or `Basic`. |
|properties.retentionInDays | integer  | The table's data retention in days. In `Basic Logs`, the value is eight days, fixed. In `Analytics Logs`, the value is between 7 and 730 days.|
|properties.totalRetentionInDays | integer  | The table's data retention that also includes the archive period.|
|properties.archiveRetentionInDays|integer|The table's archive period (read-only, calculated).|
|properties.lastPlanModifiedDate|String|Last time when the plan was set for this table. Null if no change was ever done from the default settings (read-only).

**Sample request**

```http
GET https://management.azure.com/subscriptions/ContosoSID/resourcegroups/ContosoRG/providers/Microsoft.OperationalInsights/workspaces/ContosoWorkspace/tables/ContainerLogV2?api-version=2021-12-01-preview
```

**Sample response**
 
Status code: 200
```http
{
    "properties": {
        "retentionInDays": 8,
        "totalRetentionInDays": 8,
        "archiveRetentionInDays": 0,
        "plan": "Basic",
        "lastPlanModifiedDate": "2022-01-01T14:34:04.37",
        "schema": {...},
        "provisioningState": "Succeeded"        
    },
    "id": "subscriptions/ContosoSID/resourcegroups/ContosoRG/providers/Microsoft.OperationalInsights/workspaces/ContosoWorkspace",
    "name": "ContainerLogV2"
}
```

# [CLI](#tab/cli-2)

To check the configuration of a table, run the [az monitor log-analytics workspace table show](/cli/azure/monitor/log-analytics/workspace/table#az-monitor-log-analytics-workspace-table-show) command.

For example:

```azurecli
az monitor log-analytics workspace table show --subscription ContosoSID --resource-group ContosoRG --workspace-name ContosoWorkspace --name Syslog --output table  
```

---

## Retain and archive Basic Logs

Analytics tables retain data based on a [retention and archive policy](data-retention-archive.md) you set.

Basic Logs tables retain data for eight days. When you change an existing table's plan to Basic Logs, Azure archives data that's more than eight days old but still within the table's original retention period.

## Next steps

- [Query data in Basic Logs](basic-logs-query.md)
- [Set retention and archive policies](../logs/data-retention-archive.md)

