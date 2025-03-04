---
title: What's new in Microsoft Sentinel
description: This article describes new features in Microsoft Sentinel from the past few months.
author: yelevin
ms.author: yelevin
ms.topic: conceptual
ms.date: 09/06/2022
ms.custom: ignite-fall-2021
---

# What's new in Microsoft Sentinel

This article lists recent features added for Microsoft Sentinel, and new features in related services that provide an enhanced user experience in Microsoft Sentinel.

The listed features were released in the last three months. For information about earlier features delivered, see our [Tech Community blogs](https://techcommunity.microsoft.com/t5/azure-sentinel/bg-p/AzureSentinelBlog/label-name/What's%20New).

[!INCLUDE [reference-to-feature-availability](includes/reference-to-feature-availability.md)]

## November 2022

- [Common Event Format (CEF) via AMA (Preview)](#common-event-format-cef-via-ama-preview)
- [Monitor the health of automation rules and playbooks](#monitor-the-health-of-automation-rules-and-playbooks)
- [Updated Microsoft Sentinel Logstash plugin](#updated-microsoft-sentinel-logstash-plugin)

### Common Event Format (CEF) via AMA (Preview)

The [Common Event Format (CEF) via AMA](connect-cef-ama.md) connector allows you to quickly filter and upload logs over CEF from multiple on-premises appliances to Microsoft Sentinel via the Azure Monitor Agent (AMA).

The AMA supports Data Collection Rules (DCRs), which you can use to filter the logs before ingestion, for quicker upload, efficient analysis, and querying.

### Monitor the health of automation rules and playbooks

To ensure proper functioning and performance of your security orchestration, automation, and response operations in your Microsoft Sentinel service, keep track of the health of your automation rules and playbooks by monitoring their execution logs.

Set up notifications of health events for relevant stakeholders, who can then take action. For example, define and send email or Microsoft Teams messages, create new tickets in your ticketing system, and so on.

- Learn what [health monitoring in Microsoft Sentinel](health-audit.md) can do for you.
- [Turn on health monitoring](enable-monitoring.md) in Microsoft Sentinel.
- Monitor the health of your [automation rules and playbooks](monitor-automation-health.md).
- See more information about the [*SentinelHealth* table schema](health-table-reference.md).

### Updated Microsoft Sentinel Logstash plugin

A [new version of the Microsoft Sentinel Logstash plugin](connect-logstash-data-connection-rules.md) leverages the new Azure Monitor Data Collection Rules (DCR) based Logs Ingestion API. The new plugin: 

- Provides data transformation capabilities like filtering, masking, and enrichment. 
- Allows full control over the output schema, including configuration of the column names and types. 
- Can forward logs from external data sources into both custom tables and standard tables. 
- Provides performance improvements, compression, and better telemetry and error handling.

## October 2022

- [Account enrichment fields removed from Azure AD Identity Protection connector](#account-enrichment-fields-removed-from-azure-ad-identity-protection-connector)
- [Microsoft 365 Defender now integrates Azure Active Directory Identity Protection (AADIP)](#microsoft-365-defender-now-integrates-azure-active-directory-identity-protection-aadip)
- [Out of the box anomaly detection on the SAP audit log (Preview)](#out-of-the-box-anomaly-detection-on-the-sap-audit-log-preview)
- [IoT device entity page (Preview)](#iot-device-entity-page-preview)

### Account enrichment fields removed from Azure AD Identity Protection connector

As of **September 30, 2022**, alerts coming from the **Azure Active Directory Identity Protection connector** no longer contain the following fields:

- CompromisedEntity
- ExtendedProperties["User Account"]
- ExtendedProperties["User Name”]

We are working to adapt Microsoft Sentinel's built-in queries and other operations affected by this change to look up these values in other ways (using the *IdentityInfo* table).

In the meantime, or if you've built any custom queries or rules directly referencing these fields, you'll need another way to get this information. Use the following two-step process to have your queries look up these values in the *IdentityInfo* table:

1. If you haven't already, **enable the UEBA solution** to sync the *IdentityInfo* table with your Azure AD logs. Follow the instructions in [this document](enable-entity-behavior-analytics.md).  
(If you don't intend to use UEBA in general, you can ignore the last instruction  about selecting data sources on which to enable entity behavior analytics.)

1. Incorporate the query below in your existing queries or rules to look up this data by joining the *SecurityAlert* table with the *IdentityInfo* table.

    ```kusto
    SecurityAlert
    | where TimeGenerated > ago(7d)
    | where ProductName == "Azure Active Directory Identity Protection"
    | mv-expand Entity = todynamic(Entities)
    | where Entity.Type == "account"
    | extend AadTenantId = tostring(Entity.AadTenantId)
    | extend AadUserId = tostring(Entity.AadUserId)
    | join kind=inner (
        IdentityInfo
        | where TimeGenerated > ago(14d)
        | distinct AccountTenantId, AccountObjectId, AccountUPN, AccountDisplayName
        | extend UserAccount = AccountUPN
        | extend UserName = AccountDisplayName
        | where isnotempty(AccountDisplayName) and isnotempty(UserAccount)
        | project AccountTenantId, AccountObjectId, UserAccount, UserName
        )
        on
        $left.AadTenantId == $right.AccountTenantId,
        $left.AadUserId == $right.AccountObjectId
    | extend CompromisedEntity = iff(CompromisedEntity == "N/A" or isempty(CompromisedEntity), UserAccount, CompromisedEntity)
    | project-away AadTenantId, AadUserId, AccountTenantId, AccountObjectId
    ```

For information on looking up data to replace enrichment fields removed from the UEBA UserPeerAnalytics table, See [Heads up: Name fields being removed from UEBA UserPeerAnalytics table](#heads-up-name-fields-being-removed-from-ueba-userpeeranalytics-table) for a sample query.

### Microsoft 365 Defender now integrates Azure Active Directory Identity Protection (AADIP)

As of **October 24, 2022**, [Microsoft 365 Defender](/microsoft-365/security/defender/) will be integrating [Azure Active Directory Identity Protection (AADIP)](../active-directory/identity-protection/index.yml) alerts and incidents. Customers can choose between three levels of integration:

- **Show high-impact alerts only (Default)** includes only alerts about known malicious or highly suspicious activities that might require attention. These alerts are  chosen by Microsoft security researchers and are mostly of Medium and High severities.
- **Show all alerts** includes all AADIP alerts, including activity that might not be unwanted or malicious.
- **Turn off all alerts** disables any AADIP alerts from appearing in your Microsoft 365 Defender incidents.

Microsoft Sentinel customers (who are also AADIP subscribers) with [Microsoft 365 Defender integration](microsoft-365-defender-sentinel-integration.md) enabled will automatically start receiving AADIP alerts and incidents in their Microsoft Sentinel incidents queue. Depending on your configuration, this may affect you as follows:

- If you already have your AADIP connector enabled in Microsoft Sentinel, and you've enabled incident creation, you may receive duplicate incidents. To avoid this, you have a few choices, listed here in descending order of preference:

    | Preference | Action in Microsoft 365 Defender | Action in Microsoft Sentinel |
    | - | - | - |
    | **1** | Keep the default AADIP integration of **Show high-impact alerts only**. | Disable any [**Microsoft Security** analytics rules](detect-threats-built-in.md) that create incidents from AADIP alerts. |
    | **2** | Choose the **Show all alerts** AADIP integration. | Create automation rules to automatically close incidents with unwanted alerts.<br><br>Disable any [**Microsoft Security** analytics rules](detect-threats-built-in.md) that create incidents from AADIP alerts. |
    | **3** | Don't use Microsoft 365 Defender for AADIP alerts:<br>Choose the **Turn off all alerts** option for AADIP integration. | Leave enabled those [**Microsoft Security** analytics rules](detect-threats-built-in.md) that create incidents from AADIP alerts. |

- If you don't have your [AADIP connector](data-connectors-reference.md#azure-active-directory-identity-protection) enabled, you must enable it. Be sure **not** to enable incident creation on the connector page. If you don't enable the connector, you may receive AADIP incidents without any data in them.

- If you're first enabling your Microsoft 365 Defender connector now, the AADIP connection will be made automatically behind the scenes. You won't need to do anything else.

### Out of the box anomaly detection on the SAP audit log (Preview)

The Microsoft Sentinel for SAP solution now includes the [**SAP - Dynamic Anomaly Detection analytics** rule](https://aka.ms/Sentinel4sapDynamicAnomalyAuditRuleBlog), adding an out of the box capability to identify suspicious anomalies across the SAP audit log events. 

Learn how to [use the new rule for anomaly detection](sap/configure-audit-log-rules.md#anomaly-detection).

### IoT device entity page (Preview)

The new [IoT device entity page](entity-pages.md) is designed to help the SOC investigate incidents that involve IoT/OT devices in their environment, by providing the full OT/IoT context through Microsoft Defender for IoT to Sentinel. This enables SOC teams to detect and respond more quickly across all domains to the entire attack timeline.

Learn more about [investigating IoT device entities in Microsoft Sentinel](iot-advanced-threat-monitoring.md).

## September 2022

- [Create automation rule conditions based on custom details (Preview)](#create-automation-rule-conditions-based-on-custom-details-preview)
- [Add advanced "Or" conditions to automation rules (Preview)](#add-advanced-or-conditions-to-automation-rules-preview)
- [Heads up: Name fields being removed from UEBA UserPeerAnalytics table](#heads-up-name-fields-being-removed-from-ueba-userpeeranalytics-table)
- [Windows DNS Events via AMA connector (Preview)](#windows-dns-events-via-ama-connector-preview)
- [Create and delete incidents manually (Preview)](#create-and-delete-incidents-manually-preview)
- [Add entities to threat intelligence (Preview)](#add-entities-to-threat-intelligence-preview)

### Create automation rule conditions based on custom details (Preview)

You can set the value of a [custom detail surfaced in an incident](surface-custom-details-in-alerts.md) as a condition of an automation rule. Recall that custom details are data points in raw event log records that can be surfaced and displayed in alerts and the incidents generated from them. Through custom details you can get to the actual relevant content in your alerts without having to dig through query results.

Learn how to [add a condition based on a custom detail](create-manage-use-automation-rules.md#conditions-based-on-custom-details-preview).

### Add advanced "Or" conditions to automation rules (Preview)

You can now add OR conditions to automation rules. Also known as condition groups, these allow you to combine several rules with identical actions into a single rule, greatly increasing your SOC's efficiency.

For more information, see [Add advanced conditions to Microsoft Sentinel automation rules](add-advanced-conditions-to-automation-rules.md).

### Heads up: Name fields being removed from UEBA UserPeerAnalytics table

As of **September 30, 2022**, the UEBA engine will no longer perform automatic lookups of user IDs and resolve them into names. This change will result in the removal of four name fields from the *UserPeerAnalytics* table: 

- UserName
- UserPrincipalName
- PeerUserName
- PeerUserPrincipalName 

The corresponding ID fields remain part of the table, and any built-in queries and other operations will execute the appropriate name lookups in other ways (using the IdentityInfo table), so you shouldn’t be affected by this change in nearly all circumstances. 

The only exception to this is if you’ve built custom queries or rules directly referencing any of these name fields. In this scenario, you can incorporate the following lookup queries into your own, so you can access the values that would have been in these name fields. 

The following query resolves **user** and **peer identifier fields**: 

```kusto
UserPeerAnalytics 
| where TimeGenerated > ago(24h) 
// join to resolve user identifier fields 
| join kind=inner ( 
    IdentityInfo  
    | where TimeGenerated > ago(14d) 
    | distinct AccountTenantId, AccountObjectId, AccountUPN, AccountDisplayName 
    | extend UserPrincipalNameIdentityInfo = AccountUPN 
    | extend UserNameIdentityInfo = AccountDisplayName 
    | project AccountTenantId, AccountObjectId, UserPrincipalNameIdentityInfo, UserNameIdentityInfo 
) on $left.AADTenantId == $right.AccountTenantId, $left.UserId == $right.AccountObjectId 
// join to resolve peer identifier fields 
| join kind=inner ( 
    IdentityInfo  
    | where TimeGenerated > ago(14d) 
    | distinct AccountTenantId, AccountObjectId, AccountUPN, AccountDisplayName 
    | extend PeerUserPrincipalNameIdentityInfo = AccountUPN 
    | extend PeerUserNameIdentityInfo = AccountDisplayName 
    | project AccountTenantId, AccountObjectId, PeerUserPrincipalNameIdentityInfo, PeerUserNameIdentityInfo 
) on $left.AADTenantId == $right.AccountTenantId, $left.PeerUserId == $right.AccountObjectId 
```
If your original query referenced the user or peer names (not just their IDs), substitute this query in its entirety for the table name (“UserPeerAnalytics”) in your original query. 

### Windows DNS Events via AMA connector (Preview)

You can now use the new [Windows DNS Events via AMA connector](connect-dns-ama.md) to stream and filter events from your Windows Domain Name System (DNS) server logs to the `ASimDnsActivityLog` normalized schema table. You can then dive into your data to protect your DNS servers from threats and attacks.

### Create and delete incidents manually (Preview)

Microsoft Sentinel **incidents** have two main sources: 

- They are generated automatically by detection mechanisms that operate on the logs and alerts that Sentinel ingests from its connected data sources.

- They are ingested directly from other connected Microsoft security services (such as [Microsoft 365 Defender](microsoft-365-defender-sentinel-integration.md)) that created them.

There can, however, be data from sources *not ingested into Microsoft Sentinel*, or events not recorded in any log, that justify launching an investigation. For this reason, Microsoft Sentinel now allows security analysts to manually create incidents from scratch for any type of event, regardless of its source or associated data, in order to manage and document the investigation.

Since this capability raises the possibility that you'll create an incident in error, Microsoft Sentinel also allows you to delete incidents right from the portal as well.

- [Learn more about creating incidents manually](create-incident-manually.md).
- [Learn more about deleting incidents](delete-incident.md).

### Add entities to threat intelligence (Preview)

Microsoft Sentinel now allows you to flag entities as malicious, right from within the investigation graph. You'll then be able to view this indicator both in Logs and in the Threat Intelligence blade in Sentinel.

Learn how to [add an entity to your threat intelligence](add-entity-to-threat-intelligence.md).

## Next steps

> [!div class="nextstepaction"]
>[On-board Azure Sentinel](quickstart-onboard.md)

> [!div class="nextstepaction"]
>[Get visibility into alerts](get-visibility.md)
