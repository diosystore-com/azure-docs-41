---
title: Manage NSG flow logs - Azure REST API
titleSuffix: Azure Network Watcher
description: Learn how to manage network security group flow logs in Azure Network Watcher using REST API.
services: network-watcher
author: halkazwini
ms.service: network-watcher
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 07/13/2021
ms.author: halkazwini
ms.custom: engagement-fy23
---

# Manage network security group flow logs using REST API

> [!div class="op_single_selector"]
> - [Azure portal](network-watcher-nsg-flow-logging-portal.md)
> - [PowerShell](network-watcher-nsg-flow-logging-powershell.md)
> - [Azure CLI](network-watcher-nsg-flow-logging-cli.md)
> - [REST API](network-watcher-nsg-flow-logging-rest.md)

Network Security Group flow logs are a feature of Network Watcher that allows you to view information about ingress and egress IP traffic through a Network Security Group. These flow logs are written in JSON format and show outbound and inbound flows on a per rule basis, the NIC the flow applies to, 5-tuple information about the flow (Source/Destination IP, Source/Destination Port, Protocol), and if the traffic was allowed or denied.

## Before you begin

ARMclient is used to call the REST API using PowerShell. ARMClient is found on chocolatey at [ARMClient on Chocolatey](https://chocolatey.org/packages/ARMClient). The detailed specifications of NSG flow logs REST API can be found [here](/rest/api/network-watcher/flowlogs) 

This scenario assumes you've already followed the steps in [Create a Network Watcher](network-watcher-create.md) to create a Network Watcher.

> [!Important]
> For Network Watcher REST API calls the resource group name in the request URI is the resource group that contains the Network Watcher, not the resources you are performing the diagnostic actions on.

## Scenario

The scenario covered in this article shows you how to enable, disable, and query flow logs using the REST API. To learn more about Network Security Group flow loggings, visit [Network Security Group flow logging - Overview](network-watcher-nsg-flow-logging-overview.md).

In this scenario, you will:

* Enable flow logs (Version 2)
* Disable flow logs
* Query flow logs status

## Log in with ARMClient

Log in to armclient with your Azure credentials.

```powershell
armclient login
```

## Register Insights provider

In order for flow logging to work successfully, the **Microsoft.Insights** provider must be registered. If you aren't sure if the **Microsoft.Insights** provider is registered, run the following script.

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000"
armclient post "https://management.azure.com//subscriptions/${subscriptionId}/providers/Microsoft.Insights/register?api-version=2016-09-01"
```

## Enable Network Security Group flow logs

The command to enable flow logs version 2 is shown in the following example. For version 1, replace the 'version' field with '1':

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000"
$targetUri = "" # example /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName/providers/Microsoft.Network/networkSecurityGroups/{nsgName}"
$storageId = "/subscriptions/00000000-0000-0000-0000-000000000000/{resourceGroupName/providers/Microsoft.Storage/storageAccounts/{saName}"
$resourceGroupName = "NetworkWatcherRG"
$networkWatcherName = "NetworkWatcher_westcentralus"
$requestBody = @"
{
    'targetResourceId': '${targetUri}',
    'properties': {
    'storageId': '${storageId}',
    'enabled': 'true',
    'retentionPolicy' : {
			days: 5,
			enabled: true
		},
    'format': {
        'type': 'JSON',
        'version': 2
    }
	}
}
"@

armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/configureFlowLog?api-version=2016-12-01" $requestBody
```

The response returned from the preceding example is as follows:

```json
{
  "targetResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}",
  "properties": {
    "storageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{saName}",
    "enabled": true,
    "retentionPolicy": {
      "days": 5,
      "enabled": true
    },
    "format": {
    "type": "JSON",
    "version": 2
    }
  }
}
```
> [!NOTE]
> - The API [Network Watchers - Set Flow Log Configuration](/rest/api/network-watcher/network-watchers/set-flow-log-configuration) used above is old and may soon be deprecated.
> - It is recommended to use the new [Flow Logs - Create Or Update](/rest/api/network-watcher/flow-logs/create-or-update) rest API instead.

## Disable Network Security Group flow logs

Use the following example to disable flow logs. The call is the same as enabling flow logs, except **false** is set for the enabled property.

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000"
$targetUri = "" # example /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName/providers/Microsoft.Network/networkSecurityGroups/{nsgName}"
$storageId = "/subscriptions/00000000-0000-0000-0000-000000000000/{resourceGroupName/providers/Microsoft.Storage/storageAccounts/{saName}"
$resourceGroupName = "NetworkWatcherRG"
$networkWatcherName = "NetworkWatcher_westcentralus"
$requestBody = @"
{
    'targetResourceId': '${targetUri}',
    'properties': {
    'storageId': '${storageId}',
    'enabled': 'false',
    'retentionPolicy' : {
			days: 5,
			enabled: true
		},
    'format': {
        'type': 'JSON',
        'version': 2
    }
	}
}
"@

armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/configureFlowLog?api-version=2016-12-01" $requestBody
```

The response returned from the preceding example is as follows:

```json
{
  "targetResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}",
  "properties": {
    "storageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{saName}",
    "enabled": false,
    "retentionPolicy": {
      "days": 5,
      "enabled": true
    },
    "format": {
    "type": "JSON",
    "version": 2
    }
  }
}
```

> [!NOTE]
> - The api [Network Watchers - Set Flow Log Configuration](/rest/api/network-watcher/network-watchers/set-flow-log-configuration) used above is old and may soon be deprecated.
> - It is recommended to use the new [Flow Logs - Create Or Update](/rest/api/network-watcher/flow-logs/create-or-update) rest api to disable flow logs and the [Flow Logs - Delete](/rest/api/network-watcher/flow-logs/delete) to delete flow logs resource.

## Query flow logs

The following REST call queries the status of flow logs on a Network Security Group.

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000"
$targetUri = "" # example /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName/providers/Microsoft.Network/networkSecurityGroups/{nsgName}"
$resourceGroupName = "NetworkWatcherRG"
$networkWatcherName = "NetworkWatcher_westcentralus"
$requestBody = @"
{
    'targetResourceId': '${targetUri}',
}
"@

armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/queryFlowLogStatus?api-version=2016-12-01" $requestBody
```

The following example shows the response returned:

```json
{
  "targetResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}",
  "properties": {
    "storageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{saName}",
    "enabled": true,
   "retentionPolicy": {
      "days": 5,
      "enabled": true
    },
    "format": {
    "type": "JSON",
    "version": 2
    }
  }
}
```

> [!NOTE]
> - The api [Network Watchers - Get Flow Log Status](/rest/api/network-watcher/network-watchers/get-flow-log-status) used above, requires an additional "reader" permission in the resource group of the network watcher. Also, this api is old and may soon be deprecated.
> - It is recommended to use the new [Flow Logs - Get](/rest/api/network-watcher/flow-logs/get) rest api instead.

## Download a flow log

The storage location of a flow log is defined at creation. A convenient tool to access these flow logs saved to a storage account is Microsoft Azure Storage Explorer, which can be downloaded here:  https://storageexplorer.com/

If a storage account is specified, packet capture files are saved to a storage account at the following location:

```
https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/resourceId=/SUBSCRIPTIONS/{subscriptionID}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json
```

## Next steps

Learn how to [Visualize your NSG flow logs with Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md)

Learn how to [Visualize your NSG flow logs with open source tools](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)