---
title: Monitor data from virtual machines with Azure Monitor Agent
description: Describes how to collect events and performance data from virtual machines by using Azure Monitor Agent.
ms.topic: conceptual
ms.date: 06/23/2022
author: guywild
ms.author: guywild
ms.reviewer: shseth

---

# Collect data from virtual machines with Azure Monitor Agent

This article describes how to collect events and performance counters from virtual machines by using Azure Monitor Agent.

To collect data from virtual machines by using Azure Monitor Agent, you'll:

1. Create [data collection rules (DCRs)](../essentials/data-collection-rule-overview.md) that define which data Azure Monitor Agent sends to which destinations.
1. Associate the data collection rule to specific virtual machines.

    You can associate virtual machines to multiple data collection rules. Define each data collection rule to address a particular requirement. Associate one or more data collection rules to a virtual machine based on the specific data you want the machine to collect.

## Create data collection rule and association

To send data to Log Analytics, create the data collection rule in the *same region* as your Log Analytics workspace. You can still associate the rule to machines in other supported regions.

### [Portal](#tab/portal)

1. On the **Monitor** menu, select **Data Collection Rules**.
1. Select **Create** to create a new data collection rule and associations.

    [ ![Screenshot that shows the Create button on the Data Collection Rules screen.](media/data-collection-rule-azure-monitor-agent/data-collection-rules-updated.png) ](media/data-collection-rule-azure-monitor-agent/data-collection-rules-updated.png#lightbox)

1. Enter a **Rule name** and specify a **Subscription**, **Resource Group**, **Region**, and **Platform Type**:

    - **Region** specifies where the DCR will be created. The virtual machines and their associations can be in any subscription or resource group in the tenant.
    - **Platform Type** specifies the type of resources this rule can apply to. The **Custom** option allows for both Windows and Linux types.

    [ ![Screenshot that shows the Basics tab of the Data Collection Rule screen.](media/data-collection-rule-azure-monitor-agent/data-collection-rule-basics-updated.png) ](media/data-collection-rule-azure-monitor-agent/data-collection-rule-basics-updated.png#lightbox)

1. On the **Resources** tab, add the resources to which to associate the data collection rule. Resources can be virtual machines, virtual machine scale sets, and Azure Arc for servers. The Azure portal installs Azure Monitor Agent on resources that don't already have it installed. The portal also enables Azure Managed Identity.

    > [!IMPORTANT]
    > The portal enables system-assigned managed identity on the target resources, along with existing user-assigned identities, if there are any. For existing applications, unless you specify the user-assigned identity in the request, the machine defaults to using system-assigned identity instead.

    If you need network isolation using private links, select existing endpoints from the same region for the respective resources or [create a new endpoint](../essentials/data-collection-endpoint-overview.md).

    [ ![Screenshot that shows the Resources tab of the Data Collection Rule screen.](media/data-collection-rule-azure-monitor-agent/data-collection-rule-virtual-machines-with-endpoint.png) ](media/data-collection-rule-azure-monitor-agent/data-collection-rule-virtual-machines-with-endpoint.png#lightbox)

1. On the **Collect and deliver** tab, select **Add data source** to add a data source and set a destination.
1. Select a **Data source type**.
1. Select which data you want to collect. For performance counters, you can select from a predefined set of objects and their sampling rate. For events, you can select from a set of logs and severity levels.

    [ ![Screenshot that shows the Azure portal form to select basic performance counters in a data collection rule.](media/data-collection-rule-azure-monitor-agent/data-collection-rule-data-source-basic-updated.png) ](media/data-collection-rule-azure-monitor-agent/data-collection-rule-data-source-basic-updated.png#lightbox)

1. Select **Custom** to collect logs and performance counters that aren't [currently supported data sources](azure-monitor-agent-overview.md#data-sources-and-destinations) or to [filter events by using XPath queries](#filter-events-using-xpath-queries). You can then specify an [XPath](https://www.w3schools.com/xml/xpath_syntax.asp) to collect any specific values. For an example, see [Sample DCR](data-collection-rule-sample-agent.md).

    [ ![Screenshot that shows the Azure portal form to select custom performance counters in a data collection rule.](media/data-collection-rule-azure-monitor-agent/data-collection-rule-data-source-custom-updated.png) ](media/data-collection-rule-azure-monitor-agent/data-collection-rule-data-source-custom-updated.png#lightbox)

1. On the **Destination** tab, add one or more destinations for the data source. You can select multiple destinations of the same or different types. For instance, you can select multiple Log Analytics workspaces, which is also known as multihoming.

    You can send Windows event and Syslog data sources to Azure Monitor Logs only. You can send performance counters to both Azure Monitor Metrics and Azure Monitor Logs.

    [ ![Screenshot that shows the Azure portal form to add a data source in a data collection rule.](media/data-collection-rule-azure-monitor-agent/data-collection-rule-destination.png) ](media/data-collection-rule-azure-monitor-agent/data-collection-rule-destination.png#lightbox)

1. Select **Add data source** and then select **Review + create** to review the details of the data collection rule and association with the set of virtual machines.
1. Select **Create** to create the data collection rule.

> [!NOTE]
> It might take up to 5 minutes for data to be sent to the destinations after you create the data collection rule and associations.

### [API](#tab/api)

1. Create a DCR file by using the JSON format shown in [Sample DCR](data-collection-rule-sample-agent.md).

1. Create the rule by using the [REST API](/rest/api/monitor/datacollectionrules/create#examples).

1. Create an association for each virtual machine to the data collection rule by using the [REST API](/rest/api/monitor/datacollectionruleassociations/create#examples).

### [PowerShell](#tab/powershell)

**Data collection rules**

| Action | Command |
|:---|:---|
| Get rules | [Get-AzDataCollectionRule](/powershell/module/az.monitor/get-azdatacollectionrule?view=azps-5.4.0&preserve-view=true) |
| Create a rule | [New-AzDataCollectionRule](/powershell/module/az.monitor/new-azdatacollectionrule?view=azps-6.0.0&viewFallbackFrom=azps-5.4.0&preserve-view=true) |
| Update a rule | [Set-AzDataCollectionRule](/powershell/module/az.monitor/set-azdatacollectionrule?view=azps-6.0.0&viewFallbackFrom=azps-5.4.0&preserve-view=true) |
| Delete a rule | [Remove-AzDataCollectionRule](/powershell/module/az.monitor/remove-azdatacollectionrule?view=azps-6.0.0&viewFallbackFrom=azps-5.4.0&preserve-view=true) |
| Update "Tags" for a rule | [Update-AzDataCollectionRule](/powershell/module/az.monitor/update-azdatacollectionrule?view=azps-6.0.0&viewFallbackFrom=azps-5.4.0&preserve-view=true) |

**Data collection rule associations**

| Action | Command |
|:---|:---|
| Get associations | [Get-AzDataCollectionRuleAssociation](/powershell/module/az.monitor/get-azdatacollectionruleassociation?view=azps-6.0.0&viewFallbackFrom=azps-5.4.0&preserve-view=true) |
| Create an association | [New-AzDataCollectionRuleAssociation](/powershell/module/az.monitor/new-azdatacollectionruleassociation?view=azps-6.0.0&viewFallbackFrom=azps-5.4.0&preserve-view=true) |
| Delete an association | [Remove-AzDataCollectionRuleAssociation](/powershell/module/az.monitor/remove-azdatacollectionruleassociation?view=azps-6.0.0&viewFallbackFrom=azps-5.4.0&preserve-view=true) |

### [Azure CLI](#tab/cli)

This capability is enabled as part of the Azure CLI monitor-control-service extension. [View all commands](/cli/azure/monitor/data-collection/rule).

### [Resource Manager template](#tab/arm)

For sample templates, see [Azure Resource Manager template samples for data collection rules in Azure Monitor](./resource-manager-data-collection-rules.md).

---

## Filter events using XPath queries

Since you're charged for any data you collect in a Log Analytics workspace, you should limit data collection from your agent to only the event data that you need. The basic configuration in the Azure portal provides you with a limited ability to filter out events.

[!INCLUDE [azure-monitor-cost-optimization](../../../includes/azure-monitor-cost-optimization.md)]

To specify more filters, use custom configuration and specify an XPath that filters out the events you don't need. XPath entries are written in the form `LogName!XPathQuery`. For example, you might want to return only events from the Application event log with an event ID of 1035. The `XPathQuery` for these events would be `*[System[EventID=1035]]`. Because you want to retrieve the events from the Application event log, the XPath is `Application!*[System[EventID=1035]]`

### Extract XPath queries from Windows Event Viewer

In Windows, you can use Event Viewer to extract XPath queries as shown in the screenshots.

When you paste the XPath query into the field on the **Add data source** screen, as shown in step 5, you must append the log type category followed by an exclamation point (!).

[ ![Screenshot that shows the steps to create an XPath query in the Windows Event Viewer.](media/data-collection-rule-azure-monitor-agent/data-collection-rule-extract-xpath.png) ](media/data-collection-rule-azure-monitor-agent/data-collection-rule-extract-xpath.png#lightbox)

For a list of limitations in the XPath supported by Windows event log, see [XPath 1.0 limitations](/windows/win32/wes/consuming-events#xpath-10-limitations).

> [!TIP]
> You can use the PowerShell cmdlet `Get-WinEvent` with the `FilterXPath` parameter to test the validity of an XPath query locally on your machine first. The following script shows an example:
>
> ```powershell
> $XPath = '*[System[EventID=1035]]'
> Get-WinEvent -LogName 'Application' -FilterXPath $XPath
> ```
>
> - In the preceding cmdlet, the value of the `-LogName` parameter is the initial part of the XPath query until the exclamation point (!). The rest of the XPath query goes into the `$XPath` parameter.
> - If the script returns events, the query is valid.
> - If you receive the message "No events were found that match the specified selection criteria," the query might be valid but there are no matching events on the local machine.
> - If you receive the message "The specified query is invalid," the query syntax is invalid.

Examples of using a custom XPath to filter events:

| Description |  XPath |
|:---|:---|
| Collect only System events with Event ID = 4648 |  `System!*[System[EventID=4648]]`
| Collect Security Log events with Event ID = 4648 and a process name of consent.exe | `Security!*[System[(EventID=4648)]] and *[EventData[Data[@Name='ProcessName']='C:\Windows\System32\consent.exe']]` |
| Collect all Critical, Error, Warning, and Information events from the System event log except for Event ID = 6 (Driver loaded) |  `System!*[System[(Level=1 or Level=2 or Level=3) and (EventID != 6)]]` |
| Collect all success and failure Security events except for Event ID 4624 (Successful logon) |  `Security!*[System[(band(Keywords,13510798882111488)) and (EventID != 4624)]]` |


## Next steps

- [Collect text logs by using Azure Monitor Agent](data-collection-text-log.md).
- Learn more about [Azure Monitor Agent](azure-monitor-agent-overview.md).
- Learn more about [data collection rules](../essentials/data-collection-rule-overview.md).
