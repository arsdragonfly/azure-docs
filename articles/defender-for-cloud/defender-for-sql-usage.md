---
title: How to enable Microsoft Defender for SQL servers on machines
description: Learn how to protect your Microsoft SQL servers on Azure VMs, on-premises, and in hybrid and multicloud environments with Microsoft Defender for Cloud.
ms.topic: how-to
ms.custom: ignite-2022
ms.author: benmansheim
author: bmansheim
ms.date: 07/28/2022
---

# Enable Microsoft Defender for SQL servers on machines 

This Microsoft Defender plan detects anomalous activities indicating unusual and potentially harmful attempts to access or exploit databases on the SQL server.

You'll see alerts when there are suspicious database activities, potential vulnerabilities, or SQL injection attacks, and anomalous database access and query patterns.

Microsoft Defender for SQL servers on machines extends the protections for your Azure-native SQL servers to fully support hybrid environments and protect SQL servers hosted in Azure, multicloud environments, and even on-premises machines:

- [SQL Server on Virtual Machines](https://azure.microsoft.com/services/virtual-machines/sql-server/)

- On-premises SQL servers:

    - [Azure Arc-enabled SQL Server](/sql/sql-server/azure-arc/overview)
    
    - [SQL Server running on Windows machines without Azure Arc](../azure-monitor/agents/agent-windows.md)
    
- Multicloud SQL servers:

    - [Connect your AWS accounts to Microsoft Defender for Cloud](quickstart-onboard-aws.md)

    - [Connect your GCP project to Microsoft Defender for Cloud](quickstart-onboard-gcp.md)

    > [!NOTE]
    > Enable database protection for your multicloud SQL servers through the [AWS connector](quickstart-onboard-aws.md?pivots=env-settings#connect-your-aws-account) or the [GCP connector](quickstart-onboard-gcp.md?pivots=env-settings#configure-the-databases-plan).

This plan includes functionality for identifying and mitigating potential database vulnerabilities and detecting anomalous activities that could indicate threats to your databases.

A vulnerability assessment service discovers, tracks, and helps you remediate potential database vulnerabilities. Assessment scans provide an overview of your SQL machines' security state, and details of any security findings.

Learn more about [vulnerability assessment for Azure SQL servers on machines](defender-for-sql-on-machines-vulnerability-assessment.md).

## Availability

|Aspect|Details|
|----|:----|
|Release state:|General availability (GA)|
|Pricing:|**Microsoft Defender for SQL servers on machines** is billed as shown on the [pricing page](https://azure.microsoft.com/pricing/details/defender-for-cloud/)|
|Protected SQL versions:|[SQL Server versions currently supported by Microsoft](/mem/configmgr/core/plan-design/configs/support-for-sql-server-versions) in: <br>- [SQL on Azure virtual machines](/azure/azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview)<br>- [SQL Server on Azure Arc-enabled servers](/sql/sql-server/azure-arc/overview)<br>- On-premises SQL servers on Windows machines without Azure Arc<br>|
|Clouds:|:::image type="icon" source="./media/icons/yes-icon.png"::: Commercial clouds<br>:::image type="icon" source="./media/icons/yes-icon.png"::: Azure Government<br>:::image type="icon" source="./media/icons/no-icon.png"::: Azure China 21Vianet|

## Set up Microsoft Defender for SQL servers on machines

To enable this plan:

[Step 1. Install the agent extension](#step-1-install-the-agent-extension)

[Step 2. Provision the Log Analytics agent on your SQL server's host:](#step-2-provision-the-log-analytics-agent-on-your-sql-servers-host)

[Step 3. Enable the optional plan in Defender for Cloud's environment settings page:](#step-3-enable-the-optional-plan-in-defender-for-clouds-environment-settings-page)

### Step 1. Install the agent extension

- **SQL Server on Azure VM** - Register your SQL Server VM with the SQL IaaS Agent extension as explained in [Register SQL Server VM with SQL IaaS Agent Extension](/azure/azure-sql/virtual-machines/windows/sql-agent-extension-manually-register-single-vm).

- **SQL Server on Azure Arc-enabled servers** - Install the Azure Arc agent by following the installation methods described in the [Azure Arc documentation](../azure-arc/servers/manage-vm-extensions.md).

### Step 2. Provision the Log Analytics agent on your SQL server's host:

<a name="auto-provision-mma"></a>

- **SQL Server on Azure VM** - If your SQL machine is hosted on an Azure VM, you can [customize the Log Analytics agent configuration](working-with-log-analytics-agent.md).
- **SQL Server on Azure Arc-enabled servers** - If your SQL Server is managed by [Azure Arc](../azure-arc/index.yml) enabled servers, you can deploy the Log Analytics agent using the Defender for Cloud recommendation “Log Analytics agent should be installed on your Windows-based Azure Arc machines (Preview)”.

- **SQL Server on-premises** - If your SQL Server is hosted on an on-premises Windows machine without Azure Arc, you can connect the machine to Azure by either:
    
    - **Deploy Azure Arc** - You can connect any Windows machine to Defender for Cloud. However, Azure Arc provides deeper integration across *all* of your Azure environment. If you set up Azure Arc, you'll see the **SQL Server – Azure Arc** page in the portal and your security alerts will appear on a dedicated **Security** tab on that page. So the first and recommended option is to [set up Azure Arc on the host](../azure-arc/servers/onboard-portal.md#install-and-validate-the-agent-on-windows) and follow the instructions for **SQL Server on Azure Arc**, above.
        
    - **Connect the Windows machine without Azure Arc** - If you choose to connect a SQL Server running on a Windows machine without using Azure Arc, follow the instructions in [Connect Windows machines to Azure Monitor](../azure-monitor/agents/agent-windows.md).


### Step 3. Enable the optional plan in Defender for Cloud's environment settings page:

1. From Defender for Cloud's menu, open the **Environment settings** page.

    - If you're using **Microsoft Defender for Cloud's default workspace** (named “default workspace-\<your subscription ID>-\<region>”), select the relevant **subscription**.

    - If you're using **a non-default workspace**, select the relevant **workspace** (enter the workspace's name in the filter if necessary).

1. Set the option for **Microsoft Defender for SQL servers on machines** plan to **on**. 

    :::image type="content" source="./media/security-center-advanced-iaas-data/sql-servers-on-vms-in-pricing-small.png" alt-text="Screenshot of Microsoft Defender for Cloud's 'Defender plans' page with optional plans.":::

    The plan will be enabled on all SQL servers connected to the selected workspace. The protection will be fully active after the first restart of the SQL Server instance.

    >[!TIP] 
    > To create a new workspace, follow the instructions in [Create a Log Analytics workspace](../azure-monitor/logs/quick-create-workspace.md).


1. Optionally, configure email notification for security alerts. 

    You can set a list of recipients to receive an email notification when Defender for Cloud alerts are generated. The email contains a direct link to the alert in Microsoft Defender for Cloud with all the relevant details. For more information, see [Set up email notifications for security alerts](configure-email-notifications.md).


## Microsoft Defender for SQL alerts
Alerts are generated by unusual and potentially harmful attempts to access or exploit SQL machines. These events can trigger alerts shown in the [alerts reference page](alerts-reference.md#alerts-sql-db-and-warehouse).

## Explore and investigate security alerts

Microsoft Defender for SQL alerts are available in:

- The Defender for Cloud's security alerts page
- The machine's security page
- The [workload protections dashboard](workload-protections-dashboard.md)
- Through the direct link in the alert emails

To view alerts:

1. Select **Security alerts** from Defender for Cloud's menu and select an alert.

1. Alerts are designed to be self-contained, with detailed remediation steps and investigation information in each one. You can investigate further by using other Microsoft Defender for Cloud and Microsoft Sentinel capabilities for a broader view:

    * Enable SQL Server's auditing feature for further investigations. If you're a Microsoft Sentinel user, you can upload the SQL auditing logs from the Windows Security Log events to Sentinel and enjoy a rich investigation experience. [Learn more about SQL Server Auditing](/sql/relational-databases/security/auditing/create-a-server-audit-and-server-audit-specification?preserve-view=true&view=sql-server-ver15).

    * To improve your security posture, use Defender for Cloud's recommendations for the host machine indicated in each alert to reduce the risks of future attacks.

    [Learn more about managing and responding to alerts](managing-and-responding-alerts.md).


## FAQ - Microsoft Defender for SQL servers on machines

### If I enable this Microsoft Defender plan on my subscription, are all SQL servers on the subscription protected? 

No. To defend a SQL Server deployment on an Azure virtual machine, or a SQL Server running on an Azure Arc-enabled machine, Defender for Cloud requires:

- a Log Analytics agent on the machine
- the relevant Log Analytics workspace to have the Microsoft Defender for SQL solution enabled

The subscription *status*, shown in the SQL server page in the Azure portal, reflects the default workspace status and applies to all connected machines. Only the SQL servers on hosts with a Log Analytics agent reporting to that workspace are protected by Defender for Cloud.

### Is there a performance effect from deploying Microsoft Defender for Azure SQL on machines?

The focus of **Microsoft Defender for SQL on machines** is obviously security. But we also care about your business and so we've prioritized performance to ensure the minimal effect on your SQL servers. 

The service has a split architecture to balance data uploading and speed with performance: 

- Some of our detectors, including an [extended events trace](/azure/azure-sql/database/xevent-db-diff-from-svr) named `SQLAdvancedThreatProtectionTraffic`, run on the machine for real-time speed advantages.
- Other detectors run in the cloud to spare the machine from heavy computational loads.

Lab tests of our solution showed CPU usage averaging 3% for peak slices, comparing it against benchmark loads. An analysis of our current user data shows a negligible effect on CPU and memory usage.

Of course, performance always varies between environments, machines, and loads. The statements and numbers above are provided as a general guideline, not a guarantee for any individual deployment.

## Next steps

For related information, see these resources:

- [How Microsoft Defender for Azure SQL can protect SQL servers anywhere](https://www.youtube.com/watch?v=V7RdB6RSVpc).
- [Security alerts for SQL Database and Azure Synapse Analytics](alerts-reference.md#alerts-sql-db-and-warehouse)
- [Set up email notifications for security alerts](configure-email-notifications.md)
- [Learn more about Microsoft Sentinel](../sentinel/index.yml)
