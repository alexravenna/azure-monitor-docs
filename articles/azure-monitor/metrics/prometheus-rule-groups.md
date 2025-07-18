---
title: Rule Groups in Azure Monitor Managed Service for Prometheus
description: Description of rule groups in Azure Monitor managed service for Prometheus which alerting and data computation.
ms.topic: how-to
ms.date: 11/09/2024
---

# Azure Monitor managed service for Prometheus rule groups

Rules in Prometheus act on data as the data is collected. They're configured as part of a Prometheus rule group, which is applied to Prometheus metrics in an [Azure Monitor workspace](azure-monitor-workspace-overview.md).

## Rule types

There are two types of Prometheus rules.

| Type | Description |
|:-----|:------------|
| Alert | [Alert rules](https://aka.ms/azureprometheus-promio-alertrules) let you create an Azure Monitor alert based on the results of a Prometheus Query Language (PromQL) query. Alerts fired by Azure Managed Prometheus alert rules are processed and trigger notifications in similar ways to other Azure Monitor alerts. |
| Recording | [Recording rules](https://aka.ms/azureprometheus-promio-recrules) allow you to precompute frequently needed or computationally extensive expressions and store their result as a new set of time series. Time series created by recording rules are ingested back to your Azure Monitor workspace as new Prometheus metrics. |

## Create Prometheus rules

You can create and configure Azure Managed Prometheus rule groups, recording rules, and alert rules by using the Azure resource type `Microsoft.AlertsManagement/prometheusRuleGroups`. The alert rules and recording rules are defined as part of the rule group properties. Prometheus rule groups are defined with a scope of a specific [Azure Monitor workspace](azure-monitor-workspace-overview.md). You can create Prometheus rule groups by using Azure Resource Manager templates (ARM templates), APIs, the Azure CLI, or PowerShell.

Azure Managed Prometheus rule groups follow the structure and terminology of the open-source Prometheus rule groups. Rule names, expression, `for` clause, labels, and annotations are all supported in the Azure version. Note the following key differences between open-source software rule groups and Azure Managed Prometheus:

* Azure Managed Prometheus rule groups are managed as Azure resources and include necessary information for resource management, such as the subscription and resource group where the Azure rule group should reside.
* Azure Managed Prometheus alert rules include dedicated properties that allow alerts to be processed like other Azure Monitor alerts. For example, alert severity, action group association, and alert autoresolve configuration are supported as part of Azure Managed Prometheus alert rules.

> [!NOTE]
> For your Azure Kubernetes Service (AKS) or Azure Arc-enabled Kubernetes clusters, you can use some of the recommended alerts rules. For predefined alert rules, see [this website](../containers/container-insights-metric-alerts.md#enable-prometheus-alert-rules).

### Limit rules to a specific cluster

You can optionally limit the rules in a rule group to query data originating from a single specific cluster by adding a cluster scope to your rule group or by using the rule group `clusterName` property.
Limit rules to a single cluster if your Azure Monitor workspace contains a large amount of data from multiple clusters. In such a case, there's a concern that running a single set of rules on all the data might cause performance or throttling issues. By using the cluster scope, you can create multiple rule groups, each configured with the same rules, with each group covering a different cluster.

To limit your rule group to a cluster scope [by using an ARM template](#create-a-prometheus-rule-group-by-using-an-arm-template), add the Azure resource ID value of your cluster to the rule group `scopes[]` list. *The scopes list must still include the Azure Monitor workspace resource ID.* The following cluster resource types are supported as a cluster scope:

* Azure Kubernetes Service clusters (`Microsoft.ContainerService/managedClusters`)
* Azure Arc-enabled Kubernetes clusters (`Microsoft.kubernetes/connectedClusters`)
* Azure connected appliances (`Microsoft.ResourceConnector/appliances`)

In addition to the cluster ID, you can configure the `clusterName` property of your rule group. The `clusterName` property must match the `cluster` label that's added to your metrics when scraped from a specific cluster. By default, this label is set to the last part (resource name) of your cluster ID. If you changed this label by using the [cluster_alias](../containers/prometheus-metrics-scrape-configuration.md#cluster-alias) setting in your cluster scraping ConfigMap, you must include the updated value in the rule group `clusterName` property. If your scraping uses the default `cluster` label value, the `clusterName` property is optional.

Here's an example of how a rule group is configured to limit query to a specific cluster:

``` json
{
    "name": "sampleRuleGroup",
    "type": "Microsoft.AlertsManagement/prometheusRuleGroups",
    "apiVersion": "2023-03-01",
    "location": "northcentralus",
    "properties": {
         "description": "Sample Prometheus Rule Group limited to a specific cluster",
         "scopes": [
             "/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.monitor/accounts/<azure-monitor-workspace-name>",
             "/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.containerservice/managedclusters/<myClusterName>"
         ],
         "clusterName": "<myCLusterName>",
         "rules": [
             {
                ...
             }
         ]
    }
}        
```

If both the cluster ID scope and `clusterName` property aren't specified for a rule group, the rules in the group query data from all the clusters in the workspace from all clusters.

You can also limit your rule group to a cluster scope by using the [portal UI](#configure-the-rule-group-scope).

### Create or edit a Prometheus rule group in the Azure portal

To create a new rule group from the Azure portal home page:

1. In the [Azure portal](https://portal.azure.com/), select **Monitor** > **Alerts**.

1. Select **Prometheus rule groups**.

    :::image type="content" source="media/prometheus-metrics-rule-groups/create-new-rule-group-alert-screen.png" alt-text="Screenshot that shows how to reach Prometheus rule groups from the Azure Monitor alerts screen.":::

1. Select **+ Create** to open the wizard for rule group creation.

    :::image type="content" source="media/prometheus-metrics-rule-groups/create-new-rule-group-start.png" alt-text="Screenshot that shows steps to create a new Prometheus rule group.":::

To edit a new rule group from the portal home page:

1. In the [Azure portal](https://portal.azure.com/), select **Monitor** > **Alerts**.
1. Select **Prometheus rule groups** to see the list of existing rule groups in your subscription.
1. Select the rule group that you want to go to. Enter edit mode.

#### Configure the rule group scope

On the **Scope** tab:

1. Select the Azure Monitor workspace from a list of workspaces that are available in your subscriptions. The rules in this group query data from this workspace.

1. To limit your rule group to a cluster scope, select the **Specific cluster** option:

    * Select the cluster from the list of clusters that are already connected to the selected Azure Monitor workspace.
    * The default **Cluster name** value is entered for you. Change this value only if you changed your cluster label value by using [cluster_alias](../containers/prometheus-metrics-scrape-configuration.md#cluster-alias).

1. Select **Next** to configure the rule group details.

   :::image type="content" source="media/prometheus-metrics-rule-groups/create-new-rule-group-scope.png" alt-text="Screenshot that shows configuration of Prometheus rule group scope.":::

#### Configure the rule group details

On the **Details** tab:

1. Select the subscription and resource group where the rule group should be stored.
1. Enter the rule group name and description. This name can't be changed after the rule group is created.
1. Select the **Evaluate every** period for the rule group. One minute is the default.
1. Select if the rule group is to be enabled when it's created.
1. Select **Next** to configure the rules in the group.

   :::image type="content" source="media/prometheus-metrics-rule-groups/create-new-rule-group-details.png" alt-text="Screenshot that shows configuration of Prometheus rule group details.":::

#### Configure the rules in the group

On the **Rules** tab, you can see the list of recording rules and alert rules in the group. You can add rules up to the limit of 20 rules in a single group.

Rules are evaluated in the order in which they appear in the group. You can change the order of rules by using the **move up** and **move down** options.

To add a new recording rule:

1. Select **+ Add recording rule** to open the **Create a recording rule** pane.
1. Enter the name of the rule. This name is the name of the metric created by the rule.
1. Enter the PromQL **Expression** value for the rule by using the PromQL-sensitive expression editor box. You can see the results of the expression query visualized in the preview chart. You can modify the preview time range to zoom in or out on the expression result history.
1. Select if the rule is to be enabled when created.
1. You can enter optional **Labels** key/value pairs for the rule. These labels are added to the metric created by the rule.
1. Select **Create** to add the new rule to the rule list.

:::image type="content" source="media/prometheus-metrics-rule-groups/create-new-rule-group-recording.png" alt-text="Screenshot that shows configuration of a Prometheus rule group recording rule.":::

To add a new alert rule:

1. Select **+ Add alert rule** to open the **Create an alert rule** pane.
1. Select the **Severity** value for alerts fired by this rule.
1. Enter the name of the rule. This name is the name of alerts fired by the rule.
1. Enter the PromQL **Expression** value for the rule by using the PromQL-sensitive expression editor box. You can see the results of the expression query visualized in the preview chart. You can modify the preview time range to zoom in or out on the expression result history.
1. Select the **Wait for** value for the period when the alert expression first becomes true and until the alert is fired.
1. You can enter optional **Annotations** key/value pairs for the rule. These annotations are added to alerts fired by the rule.
1. You can enter optional **Labels** key/value pairs for the rule. These labels are added to the alerts fired by the rule.
1. Select the [action groups](../alerts/action-groups.md) that the rule triggers.
1. Select **Automatically resolve alert** to automatically resolve alerts if the rule condition is no longer true during the **Time to auto-resolve** period.
1. Select if the rule is to be enabled when created.
1. Select **Create** to add the new rule to the rule list.

:::image type="content" source="media/prometheus-metrics-rule-groups/create-new-rule-group-alert.png" alt-text="Screenshot that shows configuration of Prometheus rule group alert rule.":::

> [!NOTE]
> For alert rules, the expression query typically returns only time series that fulfill the expression condition. If the preview chart isn't shown and you get the message "The query returned no result," it's likely that the condition wasn't fulfilled in the preview time range.

#### Finish creating the rule group

1. On the **Tags** tab, set any required Azure resource tags to be added to the rule group resource.

    :::image type="content" source="media/prometheus-metrics-rule-groups/create-new-rule-group-tags.png" alt-text="Screenshot that shows the Tags tab when creating a new alert rule.":::

1. On the **Review + create** tab, the rule group is validated and lets you know about any issues. On this tab, you can also select the **View automation template** option and download the template for the group that you're about to create.

1. After validation passes and you review the settings, select **Create**.

    :::image type="content" source="media/prometheus-metrics-rule-groups/create-new-rule-group-review-create.png" alt-text="Screenshot that shows the Review + create tab when you create a new alert rule.":::

1. You can follow up on the rule group deployment to make sure that it finishes successfully or to be notified of any error.

### Create a Prometheus rule group by using an ARM template

You can use an ARM template to create and configure Prometheus rule groups, alert rules, and recording rules. With ARM templates, you can programmatically create and configure rule groups in a consistent and reproducible way across all your environments.

The basic steps are:

1. Use the following template as a JSON file that describes how to create the rule group.
1. Deploy the template by using any deployment method, such as the [Azure portal](/azure/azure-resource-manager/templates/deploy-portal), the [Azure CLI](/azure/azure-resource-manager/templates/deploy-cli), [Azure PowerShell](/azure/azure-resource-manager/templates/deploy-powershell), or [REST APIs](/azure/azure-resource-manager/templates/deploy-rest).

### Template example for a Prometheus rule group

The following sample template creates a Prometheus rule group, including one recording rule and one alert rule. This template creates a resource of type `Microsoft.AlertsManagement/prometheusRuleGroups`. The scope of this group is limited to a single AKS cluster. The rules run in the order in which they appear within a group.

``` json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [
        {
           "name": "sampleRuleGroup",
           "type": "Microsoft.AlertsManagement/prometheusRuleGroups",
           "apiVersion": "2023-03-01",
           "location": "northcentralus",
           "properties": {
                "description": "Sample Prometheus Rule Group",
                "scopes": [
                    "/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.monitor/accounts/<azure-monitor-workspace-name>",
                    "/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.containerservice/managedclusters/<myClusterName>"
                ],
                "enabled": true,
                "clusterName": "<myCLusterName>",
                "interval": "PT1M",
                "rules": [
                    {
                        "record": "instance:node_cpu_utilisation:rate5m",
                        "expression": "1 - avg without (cpu) (sum without (mode)(rate(node_cpu_seconds_total{job=\"node\", mode=~\"idle|iowait|steal\"}[5m])))",
                        "labels": {
                            "workload_type": "job"
                        },
                        "enabled": true
                    },
                    {
                        "alert": "KubeCPUQuotaOvercommit",
                        "expression": "sum(min without(resource) (kube_resourcequota{job=\"kube-state-metrics\", type=\"hard\", resource=~\"(cpu|requests.cpu)\"})) /  sum(kube_node_status_allocatable{resource=\"cpu\", job=\"kube-state-metrics\"}) > 1.5",
                        "for": "PT5M",
                        "labels": {
                            "team": "prod"
                        },
                        "annotations": {
                            "description": "Cluster has overcommitted CPU resource requests for Namespaces.",
                            "runbook_url": "https://github.com/kubernetes-monitoring/kubernetes-mixin/tree/master/runbook.md#alert-name-kubecpuquotaovercommit",
                            "summary": "Cluster has overcommitted CPU resource requests."
                        },
                        "enabled": true,
                        "severity": 3,
                        "resolveConfiguration": {
                            "autoResolved": true,
                            "timeToResolve": "PT10M"
                        },
                        "actions": [
                            {
                               "actionGroupID": "/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.insights/actiongroups/<action-group-name>"
                            }
                        ]
                    }
                ]
            }
        }
    ]
}        
```

The following tables describe each of the properties in the rule definition.

### Rule group

The rule group contains the following properties.

| Name | Required | Type | Description |
|:-----|:---------|:-----|:------------|
| `name` | True | string | Prometheus rule group name. |
| `type` | True | string | `Microsoft.AlertsManagement/prometheusRuleGroups` |
| `apiVersion` | True | string | `2023-03-01` |
| `location` | True | string | Resource location out of supported regions. |
| `properties.description` | False | string | Rule group description. |
| `properties.scopes` | True | string[] | Must include the target Azure Monitor workspace ID. Can optionally include one more cluster ID. |
| `properties.enabled` | False | Boolean | Enable/disable the group. Default is `true`. |
| `properties.clusterName` | False | string | Must match the `cluster` label that's added to metrics scraped from your target cluster. By default, set to the last part (resource name) of the cluster ID that appears in `scopes[]`. |
| `properties.interval` | False | string | Group evaluation interval. Default = `PT1M`. |

### Recording rules

The `rules` section contains the following properties for recording rules.

| Name | Required | Type | Description |
|:-----|:---------|:-----|:------------|
| `record` | True | string | Recording rule name. This name is used for the new time series. |
| `expression` | True | string | PromQL expression to calculate the new time series value. |
| `labels` | True | string | Prometheus rule labels key/value pairs. These labels are added to the recorded time series. |
| `enabled` | False | boolean | Enable/disable group. Default is `true`. |

### Alert rules

The `rules` section contains the following properties for alerting rules.

| Name | Required | Type | Description | Notes |
|:-----|:---------|:-----|:------------|:------|
| `alert` | False | string | Alert rule name. | |
| `expression` | True | string | PromQL expression to evaluate. | |
| `for` | False | string | Alert firing timeout. Values = `PT1M`, `PT5M`, etc. | |
| `labels` | False | object | Labels key/value pairs. | Prometheus alert rule labels. These labels are added to alerts fired by this rule. |
| `rules.annotations` | False | object | Annotations key/value pairs to add to the alert. | |
| `enabled` | False | Boolean | Enable/disable group. Default is `true`. | |
| `rules.severity` | False | integer | Alert severity. 0-4, default is `3` (informational). | |
| `rules.resolveConfigurations.autoResolved` | False | Boolean | When enabled, the alert is automatically resolved when the condition is no longer true. Default = `true`. | |
| `rules.resolveConfigurations.timeToResolve` | False | string | Alert autoresolution timeout. Default = `PT5M`. | |
| `rules.action[].actionGroupId` | false | string | One or more action group resource IDs. Each is activated when an alert is fired. | |

### Convert Prometheus rules file to a Prometheus rule group ARM template

If you have a [Prometheus rules configuration file](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/#configuring-rules) (in YAML format), you can now convert it to an ARM template for an Azure Prometheus rule group by using the [az-prom-rules-converter utility](https://github.com/Azure/prometheus-collector/tree/main/tools/az-prom-rules-converter#az-prom-rules-converter). The rules file can contain the definition of one or more rule groups.

In addition to the rules file, you must provide the utility with other properties that are needed to create the Azure Prometheus rule groups. These properties include subscription, resource group, location, target Azure Monitor workspace, target cluster ID and name, and action groups (used for alert rules). The utility creates a template file that you can deploy directly or within a deployment pipe, which provides some of these properties as parameters.

Properties that you provide to the utility are used for all the rule groups in the template. For example, all the rule groups in the file are created in the same subscription, resource group, and location and use the same Azure Monitor workspace. If an action group is provided as a parameter to the utility, the same action group is used in all the alert rules in the template. If you want to change this default configuration (for example, use different action groups in different rules), you can edit the resulting template according to your needs before you deploy it.

> [!NOTE]
> The `az-prom-convert-utility` tool is provided as a courtesy. We recommend that you review the resulting template and verify that it matches your intended configuration.

### Create a Prometheus rule group by using the Azure CLI

You can use the Azure CLI to create and configure Prometheus rule groups, alert rules, and recording rules. The following code examples use [Azure Cloud Shell](/azure/cloud-shell/overview).

1. In the [portal](https://portal.azure.com/), select **Cloud Shell**. At the prompt, use the commands that follow.

1. To create a Prometheus rule group, use the `az alerts-management prometheus-rule-group create` command. For detailed information about this command, see the [documentation on the Azure CLI commands for creating and managing Prometheus rule groups](/cli/azure/alerts-management/prometheus-rule-group#commands).

#### Example: Create a new Prometheus rule group with rules

```azurecli
 az alerts-management prometheus-rule-group create -n TestPrometheusRuleGroup -g TestResourceGroup -l westus --enabled --description "test" --interval PT10M --scopes "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourcegroups/testrg/providers/microsoft.monitor/accounts/testaccount" --rules [{"record":"test","expression":"test","labels":{"team":"prod"}},{"alert":"Billing_Processing_Very_Slow","expression":"test","enabled":"true","severity":2,"for":"PT5M","labels":{"team":"prod"},"annotations":{"annotationName1":"annotationValue1"},"resolveConfiguration":{"autoResolved":"true","timeToResolve":"PT10M"},"actions":[{"actionGroupId":"/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/testrg/providers/microsoft.insights/actionGroups/test-action-group-name1","actionProperties":{"key11":"value11","key12":"value12"}},{"actionGroupId":"/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/testrg/providers/microsoft.insights/actionGroups/test-action-group-name2","actionProperties":{"key21":"value21","key22":"value22"}}]}]
```

### Create a new Prometheus rule group by using PowerShell

To create a Prometheus rule group by using PowerShell, use the [new-azprometheusrulegroup](/powershell/module/az.alertsmanagement/new-azprometheusrulegroup) cmdlet.

#### Example: Create a Prometheus rule group definition with rules

```powershell
$rule1 = New-AzPrometheusRuleObject -Record "job_type:billing_jobs_duration_seconds:99p5m"
$action = New-AzPrometheusRuleGroupActionObject -ActionGroupId /subscriptions/fffffffff-ffff-ffff-ffff-ffffffffffff/resourceGroups/MyresourceGroup/providers/microsoft.insights/actiongroups/MyActionGroup -ActionProperty @{"key1" = "value1"}
$Timespan = New-TimeSpan -Minutes 15
$rule2 = New-AzPrometheusRuleObject -Alert Billing_Processing_Very_Slow -Expression "job_type:billing_jobs_duration_seconds:99p5m > 30" -Enabled $false -Severity 3 -For $Timespan -Label @{"team"="prod"} -Annotation @{"annotation" = "value"} -ResolveConfigurationAutoResolved $true -ResolveConfigurationTimeToResolve $Timespan -Action $action
$rules = @($rule1, $rule2)
$scope = "/subscriptions/fffffffff-ffff-ffff-ffff-ffffffffffff/resourcegroups/MyresourceGroup/providers/microsoft.monitor/accounts/MyAccounts"
New-AzPrometheusRuleGroup -ResourceGroupName MyresourceGroup -RuleGroupName MyRuleGroup -Location eastus -Rule $rules -Scope $scope -Enabled
```

## View Prometheus rule groups

You can view your Prometheus rule groups and their included rules in the Azure portal in one of the following ways:

* On the [portal home page](https://portal.azure.com/), in the search box, look for **Prometheus Rule Groups**.
* On the [portal home page](https://portal.azure.com/), select **Monitor** > **Alerts**, and then select **Prometheus rule groups**.

    :::image type="content" source="media/prometheus-metrics-rule-groups/prometheus-rule-groups-from-alerts.png" alt-text="Screenshot that shows how to view Prometheus rule groups from the alerts screen.":::

* On the page of a specific AKS resource, or a specific Azure Monitor workspace, select **Monitor** > **Alerts**, and then select **Prometheus rule groups** to view a list of rule groups for this specific resource. You can select a rule group from the list to view or edit its details.

## View the resource health states of your Prometheus rule groups

You can now view the [resource health state](../../service-health/resource-health-overview.md) of your Prometheus rule group in the portal. You can detect problems in your rule groups, such as incorrect configuration, or query throttling problems.

1. In the [portal](https://portal.azure.com/), go to the overview of the Prometheus rule group that you want to monitor.

1. On the left pane, under **Help**, select **Resource health**.

    :::image type="content" source="media/prometheus-metrics-rule-groups/prometheus-rule-groups-resource-health.png" alt-text="Screenshot that shows how to view the resource health state of a Prometheus rule group.":::

1. On the **Resource health** pane, you can see the current availability state of the rule group. You can also see a history of recent resource health events, up to the last 30 days.

    :::image type="content" source="media/prometheus-metrics-rule-groups/prometheus-rule-groups-resource-health-history.png" alt-text="Screenshot that shows how to view the resource health history of a Prometheus rule group.":::

    * If the rule group is marked as **Available**, it's working as expected.
    * If the rule group is marked as **Degraded**, one or more rules in the group aren't working as expected. The rule query might be throttled, or other issues might cause the rule evaluation to fail. Expand the status entry for more information on the detected problem, suggestions for mitigation, or further troubleshooting.
    * If the rule group is marked as **Unavailable**, the entire rule group isn't working as expected. There might be a configuration issue (for example, the Azure Monitor workspace can't be detected) or internal service issues. Expand the status entry for more information on the detected problem, suggestions for mitigation, or further troubleshooting.
    * If the rule group is marked as **Unknown**, the entire rule group is disabled or is in an unknown state.

## Disable and enable rule groups

To enable or disable a rule, select the rule group in the Azure portal. Select either **Enable** or **Disable** to change its status.

## Related content

* [Learn more about the Azure alerts](../alerts/alerts-types.md)
* [Prometheus documentation for recording rules](https://aka.ms/azureprometheus-promio-recrules)
* [Prometheus documentation for alerting rules](https://aka.ms/azureprometheus-promio-alertrules)
