---
title: Alert processing rules for Azure Monitor alerts
description: Understand Azure Monitor alert processing rules and how to configure and manage them.
ms.topic: how-to
ms.custom: devx-track-azurepowershell
ms.date: 7/30/2024
ms.reviewer: 
---

# Alert processing rules

<a name="configuring-an-action-rule"></a>
<a name="suppression-of-alerts"></a>

> [!NOTE]
> Alert processing rules were previously known as 'action rules'. For backward compatibility, the Azure resource type of these rules is still **Microsoft.AlertsManagement/actionRules** .

Alert processing rules allow you to apply processing on fired alerts. Alert processing rules are different from alert rules. Alert rules generate new alerts that notify you when something happens, while alert processing rules modify the fired alerts as they're being fired to change the usual alert behavior.

You can use alert processing rules to add [action groups](./action-groups.md) or remove (suppress) action groups from your fired alerts. You can apply alert processing rules to different resource scopes, from a single resource, or to an entire subscription, as long as they are within the same subscription as the alert processing rule. You can also use them to apply various filters or have the rule work on a predefined schedule.

Some common use cases for alert processing rules are described here.

## Suppress notifications during planned maintenance

Many customers set up a planned maintenance time for their resources, either on a one-time basis or on a regular schedule. The planned maintenance might cover a single resource, like a virtual machine, or multiple resources, like all virtual machines in a resource group. So, you might want to stop receiving alert notifications for those resources during the maintenance window. In other cases, you might prefer to not receive alert notifications outside of your business hours. Alert processing rules allow you to achieve that.

You could suppress alert notifications by disabling the alert rules themselves at the beginning of the maintenance window, and reenable them after the maintenance is over. In that case, the alerts won't fire in the first place. That approach has several limitations:

* This approach is only practical if the scope of the alert rule is exactly the scope of the resources under maintenance. For example, a single alert rule might cover multiple resources, but only a few of those resources are going through maintenance. So, if you disable the alert rule, you won't be alerted when the remaining resources covered by that rule run into issues.
   * You might have many alert rules that cover the resource. Updating all of them is time consuming and error prone.
   * You might have some alerts that aren't created by an alert rule at all, like alerts from Azure Backup.  

In all these cases, an alert processing rule provides an easy way to suppress notifications.

## Management at scale

Most customers tend to define a few action groups that are used repeatedly in their alert rules. For example, they might want to call a specific action group whenever any high-severity alert is fired. As their number of alert rules grows, manually making sure that each alert rule has the right set of action groups is becoming harder.

Alert processing rules allow you to specify that logic in a single rule, instead of having to set it consistently in all your alert rules. They also cover alert types that aren't generated by an alert rule.

## Add action groups to all alert types

Azure Monitor alert rules let you select which action groups will be triggered when their alerts are fired. However, not all Azure alert sources let you specify action groups. Some examples of such alerts include [Azure Backup alerts](/azure/backup/backup-azure-monitoring-built-in-monitor), [VM Insights guest health alerts](../vm/vminsights-health-alerts.md), [Azure Stack Edge](/azure/databox-online/azure-stack-edge-gpu-manage-device-event-alert-notifications), and [Azure Stack Hub](/azure-stack/operator/azure-stack-monitor-health?view=azs-2408).

For those alert types, you can use alert processing rules to add action groups.

> [!NOTE]
> Alert processing rules don't affect [Azure Service Health](../../service-health/service-health-overview.md) alerts.

## Scope and filters for alert processing rules
<a name="filter-criteria"></a>

This section describes the scope and filters for alert processing rules.

Each alert processing rule has a scope. A scope is a list of one or more specific Azure resources, a specific resource group, or an entire subscription. The alert processing rule applies to alerts that fired on resources within that scope. You can't create an alert processing rule on a resource from a different subscription.   

You can also define filters to narrow down which specific subset of alerts are affected within the scope. The available filters are described in the following table.  

| Filter | Description|
|:---|:---|
Alert context (payload)  |  The rule applies only to alerts that contain any of the filter's strings within the [alert context](./alerts-common-schema.md) section of the alert. This section includes fields specific to each alert type. |
Alert rule ID |  The rule applies only to alerts from a specific alert rule. The value should be the full resource ID, for example, `/subscriptions/SUB1/resourceGroups/RG1/providers/microsoft.insights/metricalerts/MY-API-LATENCY`.  To locate the alert rule ID, open a specific alert rule in the portal, select **Properties**, and copy the **Resource ID** value. You can also locate it by listing your alert rules from PowerShell or the Azure CLI. |
Alert rule name |  The rule applies only to alerts with this alert rule name. It can also be useful with a **Contains** operator. |
Description |  The rule applies only to alerts that contain the specified string within the alert rule description field. |
Monitor condition |  The rule applies only to alerts with the specified monitor condition, either **Fired** or **Resolved**. |
Monitor service |  The rule applies only to alerts from any of the specified monitoring services that are sending the signal. Different services are available depending on the type of signal. For example: </br>**- Platform**: For metric signals, the monitor service is the metric namespace. Platform means the metrics are provided by the resource provider, namely 'Azure'.</br>**- Azure.ApplicationInsights**: Customer-reported metrics, sent by the Application Insights SDK.</br>**- Azure.VM.Windows.GuestMetrics**: VM guest metrics, collected by an extension running on the VM. Can include built-in operating system perf counters, and custom perf counters.</br>**- _\<Custom namespace\>_**: A custom metric namespace, containing custom metrics sent with the Azure Monitor Metrics API.</br>**- Log Analytics**: The service that provides the 'Custom log search' and 'Log (saved query)' signals.</br>**- Activity Log – Administrative**: The service that provides the 'Administrative' activity log events.</br>**- Activity Log – Policy**: The service that provides the 'Policy' activity log events.</br>**- Activity Log – Autoscale** The service that provides the 'Autoscale' activity log events.</br>**- Activity Log – Security**: The service that provides the 'Security' activity log events.</br>**- Resource health**: The service that provides the resource-level health status.</br>**- Service health**: The service that provides the subscription-level health status.|
Resource |  The rule applies only to alerts from the specified Azure resource. For example, you can use this filter with **Does not equal** to exclude one or more resources when the rule's scope is a subscription. | 
Resource group |  The rule applies only to alerts from the specified resource groups. For example, you can use this filter with **Does not equal** to exclude one or more resource groups when the rule's scope is a subscription. | 
Resource type |  The rule applies only to alerts on resources from the specified resource types, such as virtual machines. You can use **Equals** to match one or more specific resources. You can also use **Contains** to match a resource type and all its child resources. For example, use `resource type contains "MICROSOFT.SQL/SERVERS"` to match both SQL servers and all their child resources, like databases.
Severity |  The rule applies only to alerts with the selected severities. |


#### Alert processing rule filters

* If you define multiple filters in a rule, all the rules apply. There's a logical AND between all filters.  
  For example, if you set both `resource type = "Virtual Machines"` and `severity = "Sev0"`, then the rule applies only for `Sev0` alerts on virtual machines in the scope.
* Each filter can include up to five values. There's a logical OR between the values.  
  For example, if your set description contains "this, that" (in the field there's no need to write the apostrophes), then the rule applies only to alerts whose description contains either "this" or "that".
* Notice that you dont have any spaces (before, after or between) the string that is matched it effects the matching of the filter.

### What should this rule do?

Choose one of the following actions:

* **Suppression**: This action removes all the action groups from the affected fired alerts. So, the fired alerts won't invoke any of their action groups, not even at the end of the maintenance window. Those fired alerts will still be visible when you list your alerts in the portal, Azure Resource Graph, API, or PowerShell. The suppression action has a higher priority over the **Apply action groups** action. If a single fired alert is affected by different alert processing rules of both types, the action groups of that alert will be suppressed.
* **Apply action groups**: This action adds one or more action groups to the affected fired alerts.

### When should this rule apply?

You can control when the rule applies. The rule is always active, by default. You can select a one-time window for this rule to apply, or you can have a recurring window, such as a weekly recurrence.

## Configure an alert processing rule

### [Portal](#tab/portal)

You can access alert processing rules by going to the **Alerts** home page in Azure Monitor. Then you can select **Alert processing rules** to see and manage your existing rules. You can also select **Create** > **Alert processing rules** to open the new alert processing rule wizard.

:::image type="content" source="media/alerts-processing-rules/alerts-page-processing-rules.png" alt-text="Screenshot that shows how to access alert processing rules from the Azure Monitor landing page.":::

Let's review the new alert processing rule wizard.

1. On the **Scope** tab, you select which fired alerts are covered by this rule. Pick the **scope** of resources whose alerts will be covered. You can choose multiple resources and resource groups, or an entire subscription. You can  also optionally add filters, as previously described.

   :::image type="content" source="media/alerts-processing-rules/alert-processing-rule-scope.png" alt-text="Screenshot that shows the Scope tab of the alert processing rules wizard.":::

1. On the **Rule settings** tab, you select which action to apply on the affected alerts. Choose between **Suppress notifications** or **Apply action group**. If you choose **Apply action group**, you can select existing action groups by selecting **Add action groups**. You can also create a new action group.

   :::image type="content" source="media/alerts-processing-rules/alert-processing-rule-settings.png" alt-text="Screenshot that shows the Rule settings tab of the alert processing rules wizard.":::

1. On the **Scheduling** tab, you select an optional schedule for the rule. By default, the rule works all the time, unless you disable it. You can set it to work **On a specific time**, or you can set up a **Recurring** schedule.
  
   Let's see an example of a schedule for a one-time, overnight, planned maintenance. It starts in the evening and continues until the next morning, in a specific time zone.

   :::image type="content" source="media/alerts-processing-rules/alert-processing-rule-scheduling-one-time.png" alt-text="Screenshot that shows the Scheduling tab of the alert processing rules wizard with a one-time rule.":::

   An example of a more complex schedule covers an "outside of business hours" case. It has a recurring schedule with two recurrences. One recurrence is daily from the afternoon until the morning. The other recurrence is weekly and covers full days for Saturday and Sunday.

   :::image type="content" source="media/alerts-processing-rules/alert-processing-rule-scheduling-recurring.png" alt-text="Screenshot that shows the Scheduling tab of the alert processing rules wizard with a recurring rule.":::

1. On the **Details** tab, you give this rule a name, pick where it is stored, and optionally add a description for your reference.

1. On the **Tags** tab, you can optionally add tags to the rule.

1. On the **Review + create** tab, you can review and create the alert processing rule.

### [Azure CLI](#tab/azure-cli)

You can use the Azure CLI to work with alert processing rules. For detailed documentation and examples, see the `az monitor alert-processing-rules` [page in the Azure CLI docs](/cli/azure/monitor/alert-processing-rule).

### Prepare your environment

1. Install the Azure CLI.

   Follow the [installation instructions for the Azure CLI](/cli/azure/install-azure-cli).

   Alternatively, you can use Azure Cloud Shell, which is an interactive shell environment that you use through your browser. To start:

   - Open [Azure Cloud Shell](https://shell.azure.com).
   - Select the **Cloud Shell** button on the menu bar in the upper-right corner in the [Azure portal](https://portal.azure.com).

1. Sign in.

   If you're using a local installation of the CLI, sign in by using the `az login` [command](/cli/azure/reference-index#az-login). Follow the steps displayed in your terminal to complete the authentication process.

    ```azurecli
    az login
    ```

1. Install the `alertsmanagement` extension.

   To use the `az monitor alert-processing-rule` commands, install the `alertsmanagement` preview extension.

   ```azurecli
   az extension add --name alertsmanagement
   ```

   The following output is expected.

   ```output
   The installed extension 'alertsmanagement' is in preview.
   ```
   
   To learn more about Azure CLI extensions, see [Use extensions with the Azure CLI](/cli/azure/azure-cli-extensions-overview?).

### Create an alert processing rule with the Azure CLI

Use the `az monitor alert-processing-rule create` command to create alert processing rules.  
For example, to create a rule that adds an action group to all alerts in a subscription, run:

```azurecli
az monitor alert-processing-rule create \
  --name 'AddActionGroupToSubscription' \
  --rule-type AddActionGroups \
  --scopes "/subscriptions/SUB1" \
  --action-groups "/subscriptions/SUB1/resourcegroups/RG1/providers/microsoft.insights/actiongroups/AG1" \
  --resource-group RG1 \
  --description "Add action group AG1 to all alerts in the subscription"
```

The [CLI documentation](/cli/azure/monitor/alert-processing-rule#az-monitor-alert-processing-rule-create) includes more examples and an explanation of each parameter.

### [PowerShell](#tab/powershell)

You can use PowerShell to work with alert processing rules. For detailed documentation and examples, see the `*-AzAlertProcessingRule` commands [in the PowerShell docs](/powershell/module/az.alertsmanagement).

### Create an alert processing rule using PowerShell

Use the `Set-AzAlertProcessingRule` command to create alert processing rules. For example, to create a rule that adds an action group to all alerts in a subscription, run:

```powershell
Set-AzAlertProcessingRule `
  -Name AddActionGroupToSubscription `
  -AlertProcessingRuleType AddActionGroups `
  -Scope /subscriptions/SUB1 `
  -ActionGroupId /subscriptions/SUB1/resourcegroups/RG1/providers/microsoft.insights/actiongroups/AG1 `
  -ResourceGroupName RG1 `
  -Description "Add action group AG1 to all alerts in the subscription"
```

The [PowerShell documentation](/cli/azure/monitor/alert-processing-rule#az-monitor-alert-processing-rule-create) includes more examples and an explanation of each parameter.

* * *

## Manage alert processing rules

### [Portal](#tab/portal)

You can view and manage your alert processing rules from the list view:

:::image type="content" source="media/alerts-processing-rules/alert-processing-rules-list-view.png" alt-text="Screenshot that shows the list view of alert processing rules.":::

From here, you can enable, disable, or delete alert processing rules at scale by selecting the checkboxes next to them. Selecting an alert processing rule opens it for editing. You can enable or disable the rule on the **Details** tab.

### [Azure CLI](#tab/azure-cli)

You can view and manage your alert processing rules by using the [az monitor alert-processing-rules](/cli/azure/monitor/alert-processing-rule) commands from Azure CLI.

Before you manage alert processing rules with the Azure CLI, prepare your environment by using the instructions provided in [Configure an alert processing rule](#configure-an-alert-processing-rule).

```azurecli
# List all alert processing rules for a subscription
az monitor alert-processing-rule list

# Get details of an alert processing rule
az monitor alert-processing-rule show --resource-group RG1 --name MyRule

# Update an alert processing rule
az monitor alert-processing-rule update --resource-group RG1 --name MyRule --enabled true

# Delete an alert processing rule
az monitor alert-processing-rules delete --resource-group RG1 --name MyRule
```

### [PowerShell](#tab/powershell)

You can view and manage your alert processing rules by using the [\*-AzAlertProcessingRule](/powershell/module/az.alertsmanagement) commands from the Azure CLI.

Before you manage alert processing rules with the Azure CLI, prepare your environment by following the instructions in [Configure an alert processing rule](#configure-an-alert-processing-rule).

```powershell
# List all alert processing rules for a subscription
Get-AzAlertProcessingRule

# Get details of an alert processing rule
Get-AzAlertProcessingRule -ResourceGroupName RG1 -Name MyRule | Format-List

# Update an alert processing rule
Update-AzAlertProcessingRule -ResourceGroupName RG1 -Name MyRule -Enabled False

# Delete an alert processing rule
Remove-AzAlertProcessingRule -ResourceGroupName RG1 -Name MyRule
```

* * *

## Next steps

[Learn more about alerts in Azure](./alerts-overview.md)
