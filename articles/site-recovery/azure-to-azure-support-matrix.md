---
title: Support matrix for Azure VM disaster recovery with Azure Site Recovery
description: Summarizes support for Azure VMs disaster recovery to a secondary region with Azure Site Recovery.
ms.topic: article
ms.date: 11/11/2022
author: ankitaduttaMSFT
ms.author: ankitadutta
---

# Support matrix for Azure VM disaster recovery between Azure regions

This article summarizes support and prerequisites for disaster recovery of Azure VMs from one Azure region to another, using the [Azure Site Recovery](site-recovery-overview.md) service.


## Deployment method support

**Deployment** |  **Support**
--- | ---
**Azure portal** | Supported.
**PowerShell** | Supported. [Learn more](azure-to-azure-powershell.md)
**REST API** | Supported.
**CLI** | Not currently supported.


## Resource move/migrate support

**Resource action** | **Details**
--- | ---
**Move vaults across resource groups** | Not supported.
**Move compute/storage/network resources across resource groups** | Not supported.<br/><br/> If you move a VM or associated components such as storage/network after the VM is replicating, you need to disable and then re-enable replication for the VM.
**Replicate Azure VMs from one subscription to another for disaster recovery** | Supported within the same Azure Active Directory tenant.
**Migrate VMs across regions within supported geographical clusters (within and across subscriptions)** | Supported within the same Azure Active Directory tenant.
**Migrate VMs within the same region** | Not supported.
**Azure Dedicated Hosts** | Not supported.

## Region support

Azure Site Recovery allows you to perform global disaster recovery. You can replicate and recover VMs between any two Azure regions in the world. If you have concerns around data sovereignty, you may choose to limit replication within your specific geographic cluster. The various geographic clusters are as follows:

**Geographic cluster** | **Azure regions**
-- | --
America | Canada East, Canada Central, South Central US, West Central US, East US, East US 2, West US, West US 2, West US 3, Central US, North Central US
Europe | UK West, UK South, North Europe, West Europe, South Africa West, South Africa North, Norway East, France Central, Switzerland North, Germany West Central, UAE North (UAE is treated as part of the Europe geo cluster)
Asia | South India, Central India, West India, Southeast Asia, East Asia, Japan East, Japan West, Korea Central, Korea South, Qatar Central
JIO | JIO India West<br/><br/>Replication cannot be done between JIO and non-JIO regions for Virtual Machines present in JIO subscriptions. This is because JIO subscriptions can have resources only in JIO regions.
Australia    | Australia East, Australia Southeast, Australia Central, Australia Central 2
Azure Government    | US GOV Virginia, US GOV Iowa, US GOV Arizona, US GOV Texas, US DOD East, US DOD Central
Germany    | Germany Central, Germany Northeast
China | China East, China North, China North2, China East2
Brazil | Brazil South
Restricted Regions reserved for in-country disaster recovery |Switzerland West reserved for Switzerland North, France South reserved for France Central, Norway West for Norway East customers, JIO India Central for JIO India West customers, Brazil Southeast for Brazil South customers, South Africa West for South Africa North customers, Germany North for Germany West Central customers, UAE Central for UAE North customers.<br/><br/> To use restricted regions as your primary or recovery region, please get yourselves allowlisted by raising a request [here](/troubleshoot/azure/general/region-access-request-process) for both source and target subscriptions.

>[!NOTE]
>
> - For **Brazil South**, you can replicate and fail over to these regions: Brazil Southeast, South Central US, West Central US, East US, East US 2, West US, West US 2, and North Central US.
> - Brazil South can only be used as a source region from which VMs can replicate using Site Recovery. It can't act as a target region. Note that if you fail over from Brazil South as a source region to a target, failback to Brazil South from the target region is supported. Brazil Southeast can only be used as a target region.
> - If the region in which you want to create a vault doesn't show, make sure your subscription has access to create resources in that region.
> - If you can't see a region within a geographic cluster when you enable replication, make sure your subscription has permissions to create VMs in that region.

## Cache storage

This table summarizes support for the cache storage account used by Site Recovery during replication.

**Setting** | **Support** | **Details**
--- | --- | ---
General purpose V2 storage accounts (Hot and Cool tier) | Supported | Usage of GPv2 is recommended because GPv1 does not support ZRS (Zonal Redundant Storage). 
Premium storage | Not supported | Standard storage accounts are used for cache storage, to help optimize costs.
Region |  Same region as virtual machine  | Cache storage account should be in the same region as the virtual machine being protected.
Subscription  | Can be different from source virtual machines | Cache storage account need not be in the same subscription as the source virtual machine(s).
Azure Storage firewalls for virtual networks  | Supported | If you are using firewall enabled cache storage account or target storage account, ensure you ['Allow trusted Microsoft services'](../storage/common/storage-network-security.md#exceptions).<br></br>Also, ensure that you allow access to at least one subnet of source Vnet.<br></br>Note: Do not restrict virtual network access to your storage accounts used for Site Recovery. You should allow access from 'All networks'.
Soft delete | Not supported | Soft delete is not supported because once it is enabled on cache storage account, it increases cost. ASR performs very frequent creates/deletes of log files while replicating causing costs to increase.
Encryption at rest (CMK) | Supported | Storage account encryption can be configured with customer managed keys (CMK)

The table below lists the limits in terms of number of disks that can replicate to a single storage account.

**Storage account type**    |    **Churn = 4 MBps per disk**    |    **Churn = 8 MBps per disk**
---    |    ---    |    ---
V1 storage account    |    300 disks    |    150 disks
V2 storage account    |    750 disks    |    375 disks

As average churn on the disks increases, the number of disks that a storage account can support decreases. The above table may be used as a guide for making decisions on number of storage accounts that need to be provisioned.

Please note that the above limits are specific to Azure-to-Azure and Zone-to-Zone DR scenarios.

## Replicated machine operating systems

Site Recovery supports replication of Azure VMs running the operating systems listed in this section. Note that if an already-replicating machine is later upgraded (or downgraded) to a different major kernel, you need to disable replication and re-enable replication after the upgrade.

### Windows


**Operating system** | **Details**
--- | ---
Windows Server 2022 | Supported.
Windows Server 2019 | Supported for Server Core, Server with Desktop Experience.
Windows Server 2016  | Supported Server Core, Server with Desktop Experience.
Windows Server 2012 R2 | Supported.
Windows Server 2012 | Supported.
Windows Server 2008 R2 with SP1/SP2 | Supported.<br/><br/> From version [9.30](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery) of the Mobility service extension for Azure VMs, you need to install a Windows [servicing stack update (SSU)](https://support.microsoft.com/help/4490628) and [SHA-2 update](https://support.microsoft.com/help/4474419) on machines running Windows Server 2008 R2 SP1/SP2.  SHA-1 isn't supported from September 2019, and if SHA-2 code signing isn't enabled the agent extension won't install/upgrade as expected. Learn more about [SHA-2 upgrade and requirements](https://aka.ms/SHA-2KB).
Windows 10 (x64) | Supported.
Windows 8.1 (x64) | Supported.
Windows 8 (x64) | Supported.
Windows 7 (x64) with SP1 onwards | From version [9.30](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery) of the Mobility service extension for Azure VMs, you need to install a Windows [servicing stack update (SSU)](https://support.microsoft.com/help/4490628) and [SHA-2 update](https://support.microsoft.com/help/4474419) on machines running Windows 7 with SP1.  SHA-1 isn't supported from September 2019, and if SHA-2 code signing isn't enabled the agent extension won't install/upgrade as expected.. Learn more about [SHA-2 upgrade and requirements](https://aka.ms/SHA-2KB).



#### Linux

**Operating system** | **Details**
--- | ---
Red Hat Enterprise Linux | 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6,[7.7](https://support.microsoft.com/help/4528026/update-rollup-41-for-azure-site-recovery), [7.8](https://support.microsoft.com/help/4564347/), [7.9](https://support.microsoft.com/help/4578241/), [8.0](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery), 8.1, [8.2](https://support.microsoft.com/help/4570609/), [8.3](https://support.microsoft.com/help/4597409/), [8.4](https://support.microsoft.com/topic/883a93a7-57df-4b26-a1c4-847efb34a9e8) (4.18.0-305.30.1.el8_4.x86_64 or higher), [8.5](https://support.microsoft.com/topic/883a93a7-57df-4b26-a1c4-847efb34a9e8) (4.18.0-348.5.1.el8_5.x86_64 or higher), [8.6](https://support.microsoft.com/en-us/topic/update-rollup-62-for-azure-site-recovery-e7aff36f-b6ad-4705-901c-f662c00c402b)
CentOS | 6.5, 6.6, 6.7, 6.8, 6.9, 6.10 </br> 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7, [7.8](https://support.microsoft.com/help/4564347/), [7.9 pre-GA version](https://support.microsoft.com/help/4578241/), 7.9 GA version is supported from 9.37 hot fix patch** </br> 8.0, 8.1, [8.2](https://support.microsoft.com/help/4570609), [8.3](https://support.microsoft.com/help/4597409/), 8.4 (4.18.0-305.30.1.el8_4.x86_64 or later), 8.5 (4.18.0-348.5.1.el8_5.x86_64 or later), 8.6
Ubuntu 14.04 LTS Server | Includes support for all 14.04.*x* versions; [Supported kernel versions](#supported-ubuntu-kernel-versions-for-azure-virtual-machines);
Ubuntu 16.04 LTS Server | Includes support for all 16.04.*x* versions; [Supported kernel version](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)<br/><br/> Ubuntu servers using password-based authentication and sign-in, and the cloud-init package to configure cloud VMs, might have password-based sign-in disabled on failover (depending on the cloudinit configuration). Password-based sign in can be re-enabled on the virtual machine by resetting the password from the Support > Troubleshooting > Settings menu (of the failed over VM in the Azure portal.
Ubuntu 18.04 LTS Server | Includes support for all 18.04.*x* versions; [Supported kernel version](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)<br/><br/> Ubuntu servers using password-based authentication and sign-in, and the cloud-init package to configure cloud VMs, might have password-based sign-in disabled on failover (depending on the cloudinit configuration). Password-based sign in can be re-enabled on the virtual machine by resetting the password from the Support > Troubleshooting > Settings menu (of the failed over VM in the Azure portal.
Ubuntu 20.04 LTS server | Includes support for all 20.04.*x* versions; [Supported kernel version](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)
Debian 7 | Includes support for all 7. *x* versions [Supported kernel versions](#supported-debian-kernel-versions-for-azure-virtual-machines)
Debian 8 | Includes support for all 8. *x* versions [Supported kernel versions](#supported-debian-kernel-versions-for-azure-virtual-machines)
Debian 9 | Includes support for 9.1 to 9.13. Debian 9.0 is not supported. [Supported kernel versions](#supported-debian-kernel-versions-for-azure-virtual-machines)
Debian 10 | [Supported kernel versions](#supported-debian-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 12 | SP1, SP2, SP3, SP4, SP5  [(Supported kernel versions)](#supported-suse-linux-enterprise-server-12-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 15 | 15, SP1, SP2, SP3 [(Supported kernel versions)](#supported-suse-linux-enterprise-server-15-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 11 | SP3<br/><br/> Upgrade of replicating machines from SP3 to SP4 isn't supported. If a replicated machine has been upgraded, you need to disable replication and re-enable replication after the upgrade.
SUSE Linux Enterprise Server 11 | SP4
Oracle Linux | 6.4, 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, [7.7](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery), [7.8](https://support.microsoft.com/help/4573888/), [7.9](https://support.microsoft.com/help/4597409), [8.0](https://support.microsoft.com/help/4573888/), [8.1](https://support.microsoft.com/help/4573888/), [8.2](https://support.microsoft.com/topic/update-rollup-55-for-azure-site-recovery-kb5003408-b19c8190-5f88-43ea-85b1-d9e0cc5ca7e8), [8.3](https://support.microsoft.com/topic/update-rollup-55-for-azure-site-recovery-kb5003408-b19c8190-5f88-43ea-85b1-d9e0cc5ca7e8) (running the Red Hat compatible kernel or Unbreakable Enterprise Kernel Release 3, 4, 5, and 6 (UEK3, UEK4, UEK5, UEK6), [8.4](https://support.microsoft.com/topic/update-rollup-59-for-azure-site-recovery-kb5008707-66a65377-862b-4a4c-9882-fd74bdc7a81e), 8.5 <br/><br/>8.1 (running on all UEK kernels and RedHat kernel <= 3.10.0-1062.* are supported in [9.35](https://support.microsoft.com/help/4573888/) Support for rest of the RedHat kernels is available in [9.36](https://support.microsoft.com/help/4578241/))

> [!NOTE]
> For Linux versions, Azure Site Recovery does not support custom OS kernels. Only the stock kernels that are part of the distribution minor version release/update are supported.

> [!NOTE] 
> To support latest Linux kernels within 15 days of release, Azure Site Recovery rolls out hot fix patch on top of latest mobility agent version. This fix is rolled out in between two major version releases. To update to latest version of mobility agent (including hot fix patch), follow steps mentioned in [this article](service-updates-how-to.md#azure-vm-disaster-recovery-to-azure). This patch is currently rolled out for mobility agents used in Azure to Azure DR scenario.

#### Supported Ubuntu kernel versions for Azure virtual machines

**Release** | **Mobility service version** | **Kernel version** |
--- | --- | --- |
14.04 LTS | [9.51](https://support.microsoft.com/en-us/topic/update-rollup-64-for-azure-site-recovery-kb5020102-23db9799-102c-4378-9754-2f19f6c7858a) | No new 14.04 LTS kernels supported in this release. |
14.04 LTS | [9.50](https://support.microsoft.com/en-us/topic/update-rollup-63-for-azure-site-recovery-kb5017421-992e63af-aa94-4ea6-8d1b-2dd89a9cc70b) | No new 14.04 LTS kernels supported in this release. |
14.04 LTS | [9.49](https://support.microsoft.com/en-us/topic/update-rollup-62-for-azure-site-recovery-e7aff36f-b6ad-4705-901c-f662c00c402b) | No new 14.04 LTS kernels supported in this release. |
14.04 LTS | [9.48](https://support.microsoft.com/topic/update-rollup-61-for-azure-site-recovery-kb5012960-a1cc029b-03ad-446f-9365-a00b41025d39) | No new 14.04 LTS kernels supported in this release. |
14.04 LTS | [9.47](https://support.microsoft.com/topic/update-rollup-60-for-azure-site-recovery-k5011122-883a93a7-57df-4b26-a1c4-847efb34a9e8) | No new 14.04 LTS kernels supported in this release. |
|||
16.04 LTS | [9.51](https://support.microsoft.com/en-us/topic/update-rollup-64-for-azure-site-recovery-kb5020102-23db9799-102c-4378-9754-2f19f6c7858a) | No new 16.04 LTS kernels supported in this release. |
16.04 LTS | [9.50](https://support.microsoft.com/en-us/topic/update-rollup-63-for-azure-site-recovery-kb5017421-992e63af-aa94-4ea6-8d1b-2dd89a9cc70b) | No new 16.04 LTS kernels supported in this release. |
16.04 LTS | [9.49](https://support.microsoft.com/en-us/topic/update-rollup-62-for-azure-site-recovery-e7aff36f-b6ad-4705-901c-f662c00c402b) | No new 16.04 LTS kernels supported in this release. |
16.04 LTS | [9.48](https://support.microsoft.com/topic/update-rollup-61-for-azure-site-recovery-kb5012960-a1cc029b-03ad-446f-9365-a00b41025d39) | 4.15.0-1112-azure, 4.15.0-1113-azure |
16.04 LTS | [9.47](https://support.microsoft.com/topic/update-rollup-60-for-azure-site-recovery-k5011122-883a93a7-57df-4b26-a1c4-847efb34a9e8) | No new 16.04 LTS kernels supported in this release. |
|||
18.04 LTS | [9.51](https://support.microsoft.com/en-us/topic/update-rollup-64-for-azure-site-recovery-kb5020102-23db9799-102c-4378-9754-2f19f6c7858a) |4.15.0-1151-azure </br> 4.15.0-193-generic </br> 5.4.0-1091-azure </br> 5.4.0-126-generic</br>4.15.0-1153-azure </br>4.15.0-194-generic </br>5.4.0-1094-azure </br>5.4.0-128-generic </br>5.4.0-131-generic |
18.04 LTS |[9.50](https://support.microsoft.com/en-us/topic/update-rollup-63-for-azure-site-recovery-kb5017421-992e63af-aa94-4ea6-8d1b-2dd89a9cc70b) | 4.15.0-1149-azure </br> 4.15.0-1150-azure </br> 4.15.0-191-generic </br> 4.15.0-192-generic </br>5.4.0-1089-azure </br>5.4.0-1090-azure </br>5.4.0-124-generic|
18.04 LTS |[9.49](https://support.microsoft.com/en-us/topic/update-rollup-62-for-azure-site-recovery-e7aff36f-b6ad-4705-901c-f662c00c402b) | 4.15.0-1139-azure </br> 4.15.0-1142-azure </br> 4.15.0-1145-azure </br> 4.15.0-1146-azure </br> 4.15.0-180-generic </br> 4.15.0-184-generic </br> 4.15.0-187-generic </br> 4.15.0-188-generic </br> 4.15.0-189-generic </br> 5.4.0-1080-azure </br> 5.4.0-1083-azure </br> 5.4.0-1085-azure </br> 5.4.0-1086-azure </br> 5.4.0-113-generic </br> 5.4.0-117-generic </br> 5.4.0-120-generic </br> 5.4.0-121-generic </br> 5.4.0-122-generic |
18.04 LTS |[9.48](https://support.microsoft.com/topic/update-rollup-61-for-azure-site-recovery-kb5012960-a1cc029b-03ad-446f-9365-a00b41025d39) | 4.15.0-1134-azure </br> 4.15.0-1136-azure </br> 4.15.0-1137-azure </br> 4.15.0-1138-azure  </br> 4.15.0-173-generic </br> 4.15.0-175-generic </br> 4.15.0-176-generic </br> 4.15.0-177-generic </br> 5.4.0-105-generic </br> 5.4.0-1073-azure </br> 5.4.0-1074-azure </br> 5.4.0-1077-azure </br> 5.4.0-1078-azure </br> 5.4.0-107-generic  </br> 5.4.0-109-generic </br> 5.4.0-110-generic |
18.04 LTS |[9.47](https://support.microsoft.com/topic/update-rollup-60-for-azure-site-recovery-k5011122-883a93a7-57df-4b26-a1c4-847efb34a9e8) | 5.4.0-92-generic </br> 4.15.0-166-generic </br> 4.15.0-1129-azure </br> 5.4.0-1065-azure </br> 4.15.0-1130-azure </br> 4.15.0-167-generic </br> 5.4.0-1067-azure </br> 5.4.0-1068-azure </br> 5.4.0-94-generic </br> 5.4.0-96-generic </br> 5.4.0-97-generic </br> 5.4.0-99-generic </br> 4.15.0-1131-azure </br> 4.15.0-169-generic </br> 5.4.0-100-generic </br> 5.4.0-1069-azure </br> 5.4.0-1070-azure |
|||
20.04 LTS | [9.51](https://support.microsoft.com/en-us/topic/update-rollup-64-for-azure-site-recovery-kb5020102-23db9799-102c-4378-9754-2f19f6c7858a) |5.13.0-1009-azure </br> 5.13.0-1012-azure </br> 5.13.0-1013-azure </br> 5.13.0-1014-azure </br> 5.13.0-1017-azure </br> 5.13.0-1021-azure </br> 5.13.0-1022-azure </br> 5.13.0-1023-azure </br> 5.13.0-1025-azure </br> 5.13.0-1028-azure </br> 5.13.0-1029-azure </br> 5.13.0-1031-azure </br> 5.13.0-21-generic </br> 5.13.0-22-generic </br> 5.13.0-23-generic </br> 5.13.0-25-generic </br> 5.13.0-27-generic </br> 5.13.0-28-generic </br> 5.13.0-30-generic </br> 5.13.0-35-generic </br> 5.13.0-37-generic </br> 5.13.0-39-generic </br> 5.13.0-40-generic </br> 5.13.0-41-generic </br> 5.13.0-44-generic </br> 5.13.0-48-generic </br> 5.13.0-51-generic </br> 5.13.0-52-generic </br> 5.15.0-1007-azure </br> 5.15.0-1008-azure </br> 5.15.0-1013-azure </br> 5.15.0-1014-azure </br> 5.15.0-1017-azure </br> 5.15.0-1019-azure </br> 5.15.0-1020-azure </br> 5.15.0-33-generic </br> 5.15.0-51-generic </br> 5.15.0-43-generic </br> 5.15.0-46-generic </br> 5.15.0-48-generic </br> 5.4.0-1091-azure </br> 5.4.0-126-generic </br> 5.15.0-1021-azure </br> 5.15.0-1022-azure </br> 5.15.0-50-generic </br> 5.15.0-52-generic </br> 5.4.0-1094-azure </br> 5.4.0-128-generic </br> 5.4.0-131-generic |
20.04 LTS |[9.50](https://support.microsoft.com/en-us/topic/update-rollup-63-for-azure-site-recovery-kb5017421-992e63af-aa94-4ea6-8d1b-2dd89a9cc70b) | 5.4.0-1080-azure </br> 5.4.0-1083-azure </br> 5.4.0-1085-azure </br> 5.4.0-1086-azure </br> 5.4.0-1089-azure </br> 5.4.0-1090-azure </br> 5.4.0-113-generic </br> 5.4.0-117-generic </br> 5.4.0-120-generic </br> 5.4.0-121-generic </br> 5.4.0-122-generic </br> 5.4.0-124-generic </br> 5.4.0-125-generic |
20.04 LTS |[9.49](https://support.microsoft.com/en-us/topic/update-rollup-62-for-azure-site-recovery-e7aff36f-b6ad-4705-901c-f662c00c402b) | No new 20.04 LTS kernels supported in this release. |
20.04 LTS |[9.48](https://support.microsoft.com/topic/update-rollup-61-for-azure-site-recovery-kb5012960-a1cc029b-03ad-446f-9365-a00b41025d39) | 5.4.0-1074-azure </br>  5.4.0-107-generic </br> 5.4.0-1077-azure </br> 5.4.0-1078-azure </br> 5.4.0-109-generic </br> 5.4.0-110-generic </br> 5.11.0-1007-azure </br> 5.11.0-1012-azure </br> 5.11.0-1013-azure </br> 5.11.0-1015-azure </br> 5.11.0-1017-azure </br> 5.11.0-1019-azure </br> 5.11.0-1020-azure </br> 5.11.0-1021-azure </br> 5.11.0-1022-azure </br> 5.11.0-1023-azure </br> 5.11.0-1025-azure </br> 5.11.0-1027-azure </br> 5.11.0-1028-azure </br> 5.11.0-22-generic </br> 5.11.0-25-generic </br> 5.11.0-27-generic </br> 5.11.0-34-generic </br> 5.11.0-36-generic </br> 5.11.0-37-generic </br> 5.11.0-38-generic </br> 5.11.0-40-generic </br> 5.11.0-41-generic </br> 5.11.0-43-generic </br> 5.11.0-44-generic </br> 5.11.0-46-generic </br> 5.4.0-1077-azure </br> 5.4.0-1078-azure </br> 5.8.0-1033-azure </br> 5.8.0-1036-azure </br> 5.8.0-1039-azure </br> 5.8.0-1040-azure </br> 5.8.0-1041-azure </br> 5.8.0-1042-azure </br> 5.8.0-1043-azure </br> 5.8.0-23-generic </br> 5.8.0-25-generic </br> 5.8.0-28-generic </br> 5.8.0-29-generic </br> 5.8.0-31-generic </br> 5.8.0-33-generic </br> 5.8.0-34-generic </br> 5.8.0-36-generic </br> 5.8.0-38-generic </br> 5.8.0-40-generic </br> 5.8.0-41-generic </br> 5.8.0-43-generic </br> 5.8.0-44-generic </br> 5.8.0-45-generic </br> 5.8.0-48-generic </br> 5.8.0-49-generic </br> 5.8.0-50-generic </br> 5.8.0-53-generic </br> 5.8.0-55-generic </br> 5.8.0-59-generic </br> 5.8.0-63-generic |
20.04 LTS |[9.47](https://support.microsoft.com/topic/update-rollup-60-for-azure-site-recovery-k5011122-883a93a7-57df-4b26-a1c4-847efb34a9e8) | 5.4.0-1065-azure </br> 5.4.0-92-generic </br> 5.4.0-1067-azure </br> 5.4.0-1068-azure </br> 5.4.0-94-generic </br> 5.4.0-96-generic </br> 5.4.0-97-generic </br> 5.4.0-99-generic </br> 5.4.0-100-generic </br> 5.4.0-1069-azure </br> 5.4.0-1070-azure |


> [!Note]
> To support latest Linux kernels within 15 days of release, Azure Site Recovery rolls out hot fix patch on top of latest mobility agent version. This fix is rolled out in between two major version releases. To update to latest version of mobility agent (including hot fix patch) follow steps mentioned in [this article](service-updates-how-to.md#azure-vm-disaster-recovery-to-azure). This patch is currently rolled out for mobility agents used in Azure to Azure DR scenario.

#### Supported Debian kernel versions for Azure virtual machines

**Release** | **Mobility service version** | **Kernel version** |
--- | --- | --- |
Debian 7 | [9.51](https://support.microsoft.com/en-us/topic/update-rollup-64-for-azure-site-recovery-kb5020102-23db9799-102c-4378-9754-2f19f6c7858a) | No new Debian 7 kernels supported in this release. |
Debian 7 | [9.50](https://support.microsoft.com/en-us/topic/update-rollup-63-for-azure-site-recovery-kb5017421-992e63af-aa94-4ea6-8d1b-2dd89a9cc70b) | No new Debian 7 kernels supported in this release. |
Debian 7 | [9.49](https://support.microsoft.com/en-us/topic/update-rollup-62-for-azure-site-recovery-e7aff36f-b6ad-4705-901c-f662c00c402b) | No new Debian 7 kernels supported in this release. |
Debian 7 | [9.48](https://support.microsoft.com/topic/update-rollup-61-for-azure-site-recovery-kb5012960-a1cc029b-03ad-446f-9365-a00b41025d39) | No new Debian 7 kernels supported in this release. |
Debian 7 | [9.47](https://support.microsoft.com/topic/update-rollup-60-for-azure-site-recovery-k5011122-883a93a7-57df-4b26-a1c4-847efb34a9e8) | No new Debian 7 kernels supported in this release. |
|||
Debian 8 | [9.51](https://support.microsoft.com/en-us/topic/update-rollup-64-for-azure-site-recovery-kb5020102-23db9799-102c-4378-9754-2f19f6c7858a) | No new Debian 8 kernels supported in this release. |
Debian 8 | [9.50](https://support.microsoft.com/en-us/topic/update-rollup-63-for-azure-site-recovery-kb5017421-992e63af-aa94-4ea6-8d1b-2dd89a9cc70b) | No new Debian 8 kernels supported in this release. |
Debian 8 | [9.49](https://support.microsoft.com/en-us/topic/update-rollup-62-for-azure-site-recovery-e7aff36f-b6ad-4705-901c-f662c00c402b) | No new Debian 8 kernels supported in this release. |
Debian 8 | [9.48](https://support.microsoft.com/topic/update-rollup-61-for-azure-site-recovery-kb5012960-a1cc029b-03ad-446f-9365-a00b41025d39) | No new Debian 8 kernels supported in this release. |
Debian 8 | [9.47](https://support.microsoft.com/topic/update-rollup-60-for-azure-site-recovery-k5011122-883a93a7-57df-4b26-a1c4-847efb34a9e8) | No new Debian 8 kernels supported in this release. |
|||
Debian 9.1 | [9.51](https://support.microsoft.com/en-us/topic/update-rollup-64-for-azure-site-recovery-kb5020102-23db9799-102c-4378-9754-2f19f6c7858a) | No new Debian 9.1 kernels supported in this release. |
Debian 9.1 | [9.50](https://support.microsoft.com/en-us/topic/update-rollup-63-for-azure-site-recovery-kb5017421-992e63af-aa94-4ea6-8d1b-2dd89a9cc70b) | No new Debian 9.1 kernels supported in this release. |
Debian 9.1 | [9.49](https://support.microsoft.com/en-us/topic/update-rollup-62-for-azure-site-recovery-e7aff36f-b6ad-4705-901c-f662c00c402b) | 4.9.0-19-amd64
Debian 9.1 | [9.48](https://support.microsoft.com/topic/update-rollup-61-for-azure-site-recovery-kb5012960-a1cc029b-03ad-446f-9365-a00b41025d39) | 4.9.0-18-amd64 </br> 4.19.0-0.bpo.19-amd64 </br> 4.19.0-0.bpo.17-cloud-amd64 to 4.19.0-0.bpo.19-cloud-amd64
Debian 9.1 | [9.47](https://support.microsoft.com/topic/update-rollup-60-for-azure-site-recovery-k5011122-883a93a7-57df-4b26-a1c4-847efb34a9e8) | 4.9.0-16-amd64, 4.9.0-17-amd64
|||
Debian 10 | [9.51](https://support.microsoft.com/en-us/topic/update-rollup-64-for-azure-site-recovery-kb5020102-23db9799-102c-4378-9754-2f19f6c7858a) | 4.19.0-22-amd64 </br> 4.19.0-22-cloud-amd64 </br> 5.10.0-0.deb10.19-amd64 </br> 5.10.0-0.deb10.19-cloud-amd64 |
Debian 10 | [9.50](https://support.microsoft.com/en-us/topic/update-rollup-63-for-azure-site-recovery-kb5017421-992e63af-aa94-4ea6-8d1b-2dd89a9cc70b) | No new Debian 10 kernels supported in this release.
Debian 10 | [9.49](https://support.microsoft.com/en-us/topic/update-rollup-62-for-azure-site-recovery-e7aff36f-b6ad-4705-901c-f662c00c402b) | 4.19.0-21-amd64 </br> 4.19.0-21-cloud-amd64 </br> 5.10.0-0.bpo.15-amd64 </br> 5.10.0-0.bpo.15-cloud-amd64
Debian 10 | [9.48](https://support.microsoft.com/topic/update-rollup-61-for-azure-site-recovery-kb5012960-a1cc029b-03ad-446f-9365-a00b41025d39) | 4.19.0-20-amd64 </br> 4.19.0-20-cloud-amd64 </br> 5.8.0-0.bpo.2-amd64, 5.8.0-0.bpo.2-cloud-amd64, 5.9.0-0.bpo.2-amd64, 5.9.0-0.bpo.2-cloud-amd64, 5.9.0-0.bpo.5-amd64, 5.9.0-0.bpo.5-cloud-amd64, 5.10.0-0.bpo.7-amd64, 5.10.0-0.bpo.7-cloud-amd64, 5.10.0-0.bpo.9-amd64, 5.10.0-0.bpo.9-cloud-amd64, 5.10.0-0.bpo.11-amd64, 5.10.0-0.bpo.11-cloud-amd64, 5.10.0-0.bpo.12-amd64, 5.10.0-0.bpo.12-cloud-amd64 |
Debian 10 | [9.47](https://support.microsoft.com/topic/update-rollup-60-for-azure-site-recovery-k5011122-883a93a7-57df-4b26-a1c4-847efb34a9e8) | No new Debian 10 kernels supported in this release.

> [!Note]
> To support latest Linux kernels within 15 days of release, Azure Site Recovery rolls out hot fix patch on top of latest mobility agent version. This fix is rolled out in between two major version releases. To update to latest version of mobility agent (including hot fix patch) follow steps mentioned in [this article](service-updates-how-to.md#azure-vm-disaster-recovery-to-azure). This patch is currently rolled out for mobility agents used in Azure to Azure DR scenario.

#### Supported SUSE Linux Enterprise Server 12 kernel versions for Azure virtual machines

**Release** | **Mobility service version** | **Kernel version** |
--- | --- | --- |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4, SP5) | [9.51](https://support.microsoft.com/en-us/topic/update-rollup-64-for-azure-site-recovery-kb5020102-23db9799-102c-4378-9754-2f19f6c7858a) | All [stock SUSE 12 SP1,SP2,SP3,SP4,SP5 kernels](https://www.suse.com/support/kb/doc/?id=000019587) are supported. </br></br> 4.12.14-16.106-azure:5 </br> 4.12.14-16.112-azure |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4, SP5) | [9.50](https://support.microsoft.com/en-us/topic/update-rollup-63-for-azure-site-recovery-kb5017421-992e63af-aa94-4ea6-8d1b-2dd89a9cc70b) | All [stock SUSE 12 SP1,SP2,SP3,SP4,SP5 kernels](https://www.suse.com/support/kb/doc/?id=000019587) are supported. </br></br> No new SLES 12 Azure kernels supported in this release. |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4, SP5) | [9.49](https://support.microsoft.com/en-us/topic/update-rollup-62-for-azure-site-recovery-e7aff36f-b6ad-4705-901c-f662c00c402b) | All [stock SUSE 12 SP1,SP2,SP3,SP4,SP5 kernels](https://www.suse.com/support/kb/doc/?id=000019587) are supported. </br></br>4.12.14-16.100-azure:5 </br> 4.12.14-16.103-azure:5  |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4, SP5) | [9.48](https://support.microsoft.com/topic/update-rollup-61-for-azure-site-recovery-kb5012960-a1cc029b-03ad-446f-9365-a00b41025d39) | 4.12.14-16.94-azure:5 </br> 4.12.14-16.97-azure:5 </br> 4.12.14-122.110-default:5 </br> 4.12.14-122.113-default:5 </br> 4.12.14-122.116-default:5 </br> 4.12.14-122.121-default:5 |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4, SP5) | [9.47](https://support.microsoft.com/topic/update-rollup-60-for-azure-site-recovery-k5011122-883a93a7-57df-4b26-a1c4-847efb34a9e8) | All [stock SUSE 12 SP1,SP2,SP3,SP4,SP5 kernels](https://www.suse.com/support/kb/doc/?id=000019587) are supported.</br></br> 4.12.14-16.85-azure:5 </br> 4.12.14-122.106-default:5 </br> 4.12.14-16.88-azure:5 </br> 4.12.14-122.110-default:5 |


#### Supported SUSE Linux Enterprise Server 15 kernel versions for Azure virtual machines

**Release** | **Mobility service version** | **Kernel version** |
--- | --- | --- |
SUSE Linux Enterprise Server 15 (SP1, SP2, SP3) | [9.51](https://support.microsoft.com/en-us/topic/update-rollup-64-for-azure-site-recovery-kb5020102-23db9799-102c-4378-9754-2f19f6c7858a) | By default, all [stock SUSE 15, SP1, SP2, SP3 kernels](https://www.suse.com/support/kb/doc/?id=000019587) are supported. </br></br> 5.3.18-150300.38.80-azure |
SUSE Linux Enterprise Server 15 (SP1, SP2, SP3) | [9.50](https://support.microsoft.com/en-us/topic/update-rollup-63-for-azure-site-recovery-kb5017421-992e63af-aa94-4ea6-8d1b-2dd89a9cc70b) | By default, all [stock SUSE 15, SP1, SP2, SP3 kernels](https://www.suse.com/support/kb/doc/?id=000019587) are supported. </br></br> 5.3.18-150300.38.75-azure:3 |
SUSE Linux Enterprise Server 15, SP1, SP2, SP3 | [9.49](https://support.microsoft.com/en-us/topic/update-rollup-62-for-azure-site-recovery-e7aff36f-b6ad-4705-901c-f662c00c402b) | By default, all [stock SUSE 15, SP1, SP2, SP3 kernels](https://www.suse.com/support/kb/doc/?id=000019587) are supported. </br></br>5.3.18-150300.38.69-azure:3 </br>
SUSE Linux Enterprise Server 15, SP1, SP2, SP3 | [9.48](https://support.microsoft.com/topic/update-rollup-61-for-azure-site-recovery-kb5012960-a1cc029b-03ad-446f-9365-a00b41025d39) | By default, all [stock SUSE 15, SP1, SP2, SP3 kernels](https://www.suse.com/support/kb/doc/?id=000019587) are supported.</br></br>  5.3.18-150300.38.37-azure:3 </br> 5.3.18-150300.38.40-azure:3 </br> 5.3.18-150300.38.47-azure:3 </br> 5.3.18-150300.38.50-azure:3 </br> 5.3.18-150300.38.53-azure:3 </br> 5.3.18-150300.38.56-azure:3 </br> 5.3.18-150300.38.59-azure:3 </br> 5.3.18-150300.38.62-azure:3 </br> 5.3.18-36-azure:3 </br> 5.3.18-38.11-azure:3 </br> 5.3.18-38.14-azure:3 </br> 5.3.18-38.17-azure:3 </br> 5.3.18-38.22-azure:3 </br> 5.3.18-38.25-azure:3 </br> 5.3.18-38.28-azure:3 </br> 5.3.18-38.31-azure:3 </br> 5.3.18-38.34-azure:3 </br> 5.3.18-38.3-azure:3 </br> 5.3.18-38.8-azure:3 |
SUSE Linux Enterprise Server 15, SP1, SP2 | [9.47](https://support.microsoft.com/topic/update-rollup-60-for-azure-site-recovery-k5011122-883a93a7-57df-4b26-a1c4-847efb34a9e8) | By default, all [stock SUSE 15, SP1, SP2 kernels](https://www.suse.com/support/kb/doc/?id=000019587) are supported.</br></br> 5.3.18-38.31-azure </br> 5.3.18-38.8-azure </br> 5.3.18-57-default </br> 5.3.18-59.10-default </br> 5.3.18-59.13-default </br> 5.3.18-59.16-default </br> 5.3.18-59.19-default </br> 5.3.18-59.24-default </br> 5.3.18-59.27-default </br> 5.3.18-59.30-default </br> 5.3.18-59.34-default </br> 5.3.18-59.37-default </br> 5.3.18-59.5-default </br> 5.3.18-38.34-azure:3 </br> 5.3.18-150300.59.43-default:3 </br> 5.3.18-150300.59.46-default:3 </br> 5.3.18-59.40-default:3 </br>


> [!Note] 
> To support latest Linux kernels within 15 days of release, Azure Site Recovery rolls out hot fix patch on top of latest mobility agent version. This fix is rolled out in between two major version releases. To update to latest version of mobility agent (including hot fix patch) follow steps mentioned in [this article](service-updates-how-to.md#azure-vm-disaster-recovery-to-azure). This patch is currently rolled out for mobility agents used in Azure to Azure DR scenario.

## Replicated machines - Linux file system/guest storage

* File systems: ext3, ext4, XFS, BTRFS
* Volume manager: LVM2

> [!NOTE]
> Multipath software is not supported.


## Replicated machines - compute settings

**Setting** | **Support** | **Details**
--- | --- | ---
Size | Any Azure VM size with at least two CPU cores and 1-GB RAM | Verify [Azure virtual machine sizes](../virtual-machines/sizes.md).
RAM | Azure Site Recovery driver consumes 6% of RAM.
Availability sets | Supported | If you enable replication for an Azure VM with the default options, an availability set is created automatically, based on the source region settings. You can modify these settings.
Availability zones | Supported |
Dedicated Hosts | Not supported |
Hybrid Use Benefit (HUB) | Supported | If the source VM has a HUB license enabled, a test failover or failed over VM also uses the HUB license.
VMSS Flex | Availability scenario - supported. Scalability scenario - not supported. |
Azure gallery images - Microsoft published | Supported | Supported if the VM runs on a supported operating system.
Azure Gallery images - Third party published | Supported | Supported if the VM runs on a supported operating system.
Custom images - Third party published | Supported | Supported if the VM runs on a supported operating system.
VMs migrated using Site Recovery | Supported | If a VMware VM or physical machine was migrated to Azure using Site Recovery, you need to uninstall the older version of Mobility service running on the machine, and restart the machine before replicating it to another Azure region.
Azure RBAC policies | Not supported | Azure role-based access control (Azure RBAC) policies on VMs are not replicated to the failover VM in target region.
Extensions | Not supported | Extensions are not replicated to the failover VM in target region. It needs to be installed manually after failover.
Proximity Placement Groups | Supported | Virtual machines located inside a Proximity Placement Group can be protected using Site Recovery.
Tags  | Supported | User-generated tags applied on source virtual machines are carried over to target virtual machines post-test failover or failover. Tags on the VM(s) are replicated once every 24 hours for as long as the VM(s) is/are present in the target region.


## Replicated machines - disk actions

**Action** | **Details**
-- | ---
Resize disk on replicated VM | Resizing up on the source VM is supported. Resizing down on the source VM is not supported. Resizing should be performed before failover. No need to disable/re-enable replication.<br/><br/> If you change the source VM after failover, the changes aren't captured.<br/><br/> If you change the disk size on the Azure VM after failover, changes aren't captured by Site Recovery, and failback will be to the original VM size.<br/><br/> If resizing to >=4 TB, please note Azure guidance on disk caching [here](../virtual-machines/premium-storage-performance.md). 
Add a disk to a replicated VM | Supported
Offline changes to protected disks | Disconnecting disks and making offline modifications to them require triggering a full resync.
Disk caching | Disk Caching is not supported for disks 4 TiB and larger. If multiple disks are attached to your VM, each disk that is smaller than 4 TiB will support caching. Changing the cache setting of an Azure disk detaches and re-attaches the target disk. If it is the operating system disk, the VM is restarted. Stop all applications/services that might be affected by this disruption before changing the disk cache setting. Not following those recommendations could lead to data corruption.

## Replicated machines - storage

This table summarized support for the Azure VM OS disk, data disk, and temporary disk.

- It's important to observe the VM disk limits and targets for [managed disks](../virtual-machines/disks-scalability-targets.md) to avoid any performance issues.
- If you deploy with the default settings, Site Recovery automatically creates disks and storage accounts based on the source settings.
- If you customize, ensure you follow the guidelines.

**Component** | **Support** | **Details**
--- | --- | ---
OS disk maximum size | 4096 GB | [Learn more](../virtual-machines/managed-disks-overview.md) about VM disks.
Temporary disk | Not supported | The temporary disk is always excluded from replication.<br/><br/> Don't store any persistent data on the temporary disk. [Learn more](../virtual-machines/managed-disks-overview.md).
Data disk maximum size | 32 TB for managed disks<br></br>4     TB for unmanaged disks|
Data disk minimum size | No restriction for unmanaged disks. 1 GB for managed disks |
Data disk maximum number | Up to 64, in accordance with support for a specific Azure VM size | [Learn more](../virtual-machines/sizes.md) about VM sizes.
Data disk maximum size per storage account (for unmanaged disks) | 35 TB | This is an upper limit for cumulative size of page blobs created in a premium Storage Account
Data disk change rate | Maximum of 20 MBps per disk for premium storage. Maximum of 2 MBps per disk for Standard storage. | If the average data change rate on the disk is continuously higher than the maximum, replication won't catch up.<br/><br/>  However, if the maximum is exceeded sporadically, replication can catch up, but you might see slightly delayed recovery points.
Data disk - standard storage account | Supported |
Data disk - premium storage account | Supported | If a VM has disks spread across premium and standard storage accounts, you can select a different target storage account for each disk, to ensure you have the same storage configuration in the target region.
Managed disk - standard | Supported in Azure regions in which Azure Site Recovery is supported. |
Managed disk - premium | Supported in Azure regions in which Azure Site Recovery is supported. |
Disk subscription limits | Up to 3000 protected disks per Subscription | Ensure that the Source or Target subscription does not have more than 3000 Azure Site Recovery-protected Disks (Both Data and OS).
Standard SSD | Supported |
Redundancy | LRS and GRS are supported.<br/><br/> ZRS isn't supported.
Cool and hot storage | Not supported | VM disks aren't supported on cool and hot storage
Storage Spaces | Supported |
NVMe storage interface | Not supported
Encryption at host | Not Supported | The VM will get protected, but the failed over VM will not have Encryption at host enabled. [See detailed information](../virtual-machines/disks-enable-host-based-encryption-portal.md) to create a VM with end-to-end encryption using Encryption at host.
Encryption at rest (SSE) | Supported | SSE is the default setting on storage accounts.
Encryption at rest (CMK) | Supported | Both Software and HSM keys are supported for managed disks
Double Encryption at rest | Supported | Learn more on supported regions for [Windows](../virtual-machines/disk-encryption.md) and [Linux](../virtual-machines/disk-encryption.md)
FIPS encryption | Not supported
Azure Disk Encryption (ADE) for Windows OS | Supported for VMs with managed disks. | VMs using unmanaged disks are not supported. <br/><br/> HSM-protected keys are not supported. <br/><br/> Encryption of individual volumes on a single disk is not supported. |
Azure Disk Encryption (ADE) for Linux OS | Supported for VMs with managed disks. | VMs using unmanaged disks are not supported. <br/><br/> HSM-protected keys are not supported. <br/><br/> Encryption of individual volumes on a single disk is not supported. <br><br> Known issue with enabling replication. [Learn more.](./azure-to-azure-troubleshoot-errors.md#enable-protection-failed-as-the-installer-is-unable-to-find-the-root-disk-error-code-151137) |
SAS key rotation | Not Supported | If the SAS key for storage accounts is rotated, customer needs to disable and re-enable replication. |
Host Caching | Supported
Hot add    | Supported | Enabling replication for a data disk that you add to a replicated Azure VM is supported for VMs that use managed disks. <br/><br/> Only one disk can be hot added to an Azure VM at a time. Parallel addition of multiple disks is not supported. |
Hot remove disk    | Not supported | If you  remove data disk on the VM, you need to disable replication and enable replication again for the VM.
Exclude disk | Support. You must use [PowerShell](azure-to-azure-exclude-disks.md) to configure. |    Temporary disks are excluded by default.
Storage Spaces Direct  | Supported for crash consistent recovery points. Application consistent recovery points are not supported. |
Scale-out File Server  | Supported for crash consistent recovery points. Application consistent recovery points are not supported. |
DRBD | Disks that are part of a DRBD setup are not supported. |
LRS | Supported |
GRS | Supported |
RA-GRS | Supported |
ZRS | Not supported | 
Cool and Hot Storage | Not supported | Virtual machine disks are not supported on cool and hot storage
Azure Storage firewalls for virtual networks  | Supported | If restrict virtual network access to storage accounts, enable [Allow trusted Microsoft services](../storage/common/storage-network-security.md#exceptions).
General purpose V2 storage accounts (Both Hot and Cool tier) | Supported | Transaction costs increase substantially compared to General purpose V1 storage accounts
Generation 2 (UEFI boot) | Supported
NVMe disks | Not supported
Azure Shared Disks | Not supported
Ultra Disks | Not supported
Secure transfer option | Supported
Write accelerator enabled disks | Not supported
Tags  | Supported | User-generated tags are replicated every 24 hours.
Soft delete | Not supported | Soft delete is not supported because once it is enabled on a storage account, it increases cost. ASR performs very frequent creates/deletes of log files while replicating causing costs to increase.
iSCSI disks | Not supported | ASR may be used to migrate or failover iSCSI disks into Azure. However, iSCSI disks are not supported for Azure to Azure replication and failover/failback.

>[!IMPORTANT]
> To avoid performance issues, make sure that you follow VM disk scalability and performance targets for [managed disks](../virtual-machines/disks-scalability-targets.md). If you use default settings, Site Recovery creates the required disks and storage accounts, based on the source configuration. If you customize and select your own settings,follow the disk scalability and performance targets for your source VMs.

## Limits and data change rates

The following table summarizes Site Recovery limits.

- These limits are based on our tests, but obviously don't cover all possible application I/O combinations.
- Actual results can vary based on your app I/O mix.
- There are two limits to consider, per disk data churn and per virtual machine data churn.
- The current limit for per virtual machine data churn is 54 MB/s, regardless of size.

**Storage target** | **Average source disk I/O** |**Average source disk data churn** | **Total source disk data churn per day**
---|---|---|---
Standard storage | 8 KB    | 2 MB/s | 168 GB per disk
Premium P10 or P15 disk | 8 KB    | 2 MB/s | 168 GB per disk
Premium P10 or P15 disk | 16 KB | 4 MB/s |    336 GB per disk
Premium P10 or P15 disk | 32 KB or greater | 8 MB/s | 672 GB per disk
Premium P20 or P30 or P40 or P50 disk | 8 KB    | 5 MB/s | 421 GB per disk
Premium P20 or P30 or P40 or P50 disk | 16 KB or greater |20 MB/s | 1684 GB per disk

## Replicated machines - networking
**Setting** | **Support** | **Details**
--- | --- | ---
NIC | Maximum number supported for a specific Azure VM size | NICs are created when the VM is created during failover.<br/><br/> The number of NICs on the failover VM depends on the number of NICs on the source VM when replication was enabled. If you add or remove a NIC after enabling replication, it doesn't impact the number of NICs on the replicated VM after failover. <br/><br/> The order of NICs after failover is not guaranteed to be the same as the original order. <br/><br/> You can rename NICs in the target region based on your organization's naming conventions. NIC renaming is supported using PowerShell.
Internet Load Balancer | Not supported | You can set up public/internet load balancers in the primary region. However, public/internet load balancers are not supported by Azure Site Recovery in the DR region.
Internal Load balancer | Supported | Associate the preconfigured load balancer using an Azure Automation script in a recovery plan.
Public IP address | Supported | Associate an existing public IP address with the NIC. Or, create a public IP address and associate it with the NIC using an Azure Automation script in a recovery plan.
NSG on NIC | Supported | Associate the NSG with the NIC using an Azure Automation script in a recovery plan.
NSG on subnet | Supported | Associate the NSG with the subnet using an Azure Automation script in a recovery plan.
Reserved (static) IP address | Supported | If the NIC on the source VM has a static IP address, and the target subnet has the same IP address available, it's assigned to the failed over VM.<br/><br/> If the target subnet doesn't have the same IP address available, one of the available IP addresses in the subnet is reserved for the VM.<br/><br/> You can also specify a fixed IP address and subnet in **Replicated items** > **Settings** > **Network** > **Network interfaces**.
Dynamic IP address | Supported | If the NIC on the source has dynamic IP addressing, the NIC on the failed over VM is also dynamic by default.<br/><br/> You can modify this to a fixed IP address if required.
Multiple IP addresses | Supported | When you fail over a VM that has a NIC with multiple IP addresses, only the primary IP address of the NIC in the source region is kept by default. To failover Secondary IP Configurations, go to the **Network** blade and configure them.
Traffic Manager     | Supported | You can preconfigure Traffic Manager so that traffic is routed to the endpoint in the source region on a regular basis, and to the endpoint in the target region in case of failover.
Azure DNS | Supported |
Custom DNS    | Supported |
Unauthenticated proxy | Supported | [Learn more](./azure-to-azure-about-networking.md)
Authenticated Proxy | Not supported | If the VM is using an authenticated proxy for outbound connectivity, it cannot be replicated using Azure Site Recovery.
VPN site-to-site connection to on-premises<br/><br/>(with or without ExpressRoute)| Supported | Ensure that the UDRs and NSGs are configured in such a way that the Site Recovery traffic is not routed to on-premises. [Learn more](./azure-to-azure-about-networking.md)
VNET to VNET connection    | Supported | [Learn more](./azure-to-azure-about-networking.md)
Virtual Network Service Endpoints | Supported | If you are restricting the virtual network access to storage accounts, ensure that the trusted Microsoft services are allowed access to the storage account.
Accelerated networking | Supported | Accelerated networking can be enabled on the recovery VM only if it is enabled on the source VM also. [Learn more](azure-vm-disaster-recovery-with-accelerated-networking.md).
Palo Alto Network Appliance | Not supported | With third-party appliances, there are often restrictions imposed by the provider inside the Virtual Machine. Azure Site Recovery needs agent, extensions, and outbound connectivity to be available. But the appliance does not let any outbound activity to be configured inside the Virtual Machine.
IPv6  | Not supported | Mixed configurations that include both IPv4 and IPv6 are also not supported. Free up the subnet of the IPv6 range before any Site Recovery operation.
Private link access to Site Recovery service | Supported | [Learn more](azure-to-azure-how-to-enable-replication-private-endpoints.md)
Tags  | Supported | User-generated tags on NICs are replicated every 24 hours.



## Next steps

- Read [networking guidance](./azure-to-azure-about-networking.md)  for replicating Azure VMs.
- Deploy disaster recovery by [replicating Azure VMs](./azure-to-azure-quickstart.md).
