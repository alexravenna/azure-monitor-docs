---
title: Example log table queries for WOUserAudits
description:  Example queries for WOUserAudits log table
ms.topic: generated-reference
ms.service: azure-monitor
ms.author: edbaynash
author: EdB-MSFT
ms.date: 04/14/2025

# This file is automatically generated. Changes will be overwritten. Do not change this file directly. 

---

# Queries for the WOUserAudits table

For information on using these queries in the Azure portal, see [Log Analytics tutorial](/azure/azure-monitor/logs/log-analytics-tutorial). For the REST API, see [Query](/rest/api/loganalytics/query).


### Auditing workload orchestration Operations  


Lists of audit workload orchestration operations.  

```query
WOUserAudits
| where Message !startswith_cs "Request" 
| order by EdgeLocation, TimeGenerated desc
| project EdgeLocation, TimeGenerated, User, Message, OperatingResourceId, OperatingResourceK8SId, OperationName
| take 100
```



### Auditing workload orchestration API requests  


Lists of audit workload orchestration api requests.  

```query
WOUserAudits
| where Message startswith_cs "Request" 
| order by EdgeLocation, TimeGenerated desc
| project EdgeLocation, TimeGenerated, User, Message, OperatingResourceId, OperatingResourceK8SId, OperationName
| take 100
```

