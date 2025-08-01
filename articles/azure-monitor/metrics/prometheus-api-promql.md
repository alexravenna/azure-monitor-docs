---
title: Query Prometheus Metrics by Using the API and PromQL
description: Learn how to use the API to query metrics in an Azure Monitor workspace by using PromQL.
ms.topic: how-to
ms.date: 09/17/2024
ms.reviewer: aul
---

# Query Prometheus metrics by using the API and PromQL

Azure Monitor managed service for Prometheus collects metrics from Azure Kubernetes clusters and stores them in an Azure Monitor workspace. Prometheus Query Language (PromQL) is a functional query language that you can use to query and aggregate time-series data. Use PromQL to query and aggregate metrics stored in an Azure Monitor workspace.

This article describes how to query an Azure Monitor workspace by using PromQL via the REST API. For more information on PromQL, see [Query Prometheus](https://prometheus.io/docs/prometheus/latest/querying/basics/).

## Prerequisites

To query an Azure monitor workspace by using PromQL, you need:

* An Azure Kubernetes cluster or remote Kubernetes cluster.
* Azure Monitor managed service for Prometheus scraping metrics from a Kubernetes cluster.
* An Azure Monitor workspace where Prometheus metrics are being stored.

## Authentication

To query your Azure Monitor workspace, authenticate by using Microsoft Entra ID. The API supports Microsoft Entra authentication by using client credentials. Register a client app with Microsoft Entra ID and request a token.

To set up Microsoft Entra authentication, follow these steps:

1. Register an app with Microsoft Entra ID.
1. Grant access for the app to your Azure Monitor workspace.
1. Request a token.

<a name='register-an-app-with-azure-active-directory'></a>

### Register an app with Microsoft Entra ID

To register an app, follow the steps in [Register an app to request authorization tokens and work with APIs](../logs/api/register-app-for-token.md?tabs=portal).

### Allow your app access to your workspace

Assign the Monitoring Data Reader role to your app so that it can query data from your Azure Monitor workspace.

1. Open your Azure Monitor workspace in the Azure portal.

1. On the **Overview** page, note your query endpoint for use in your REST request.

1. Select **Access control (IAM)**.

1. On the **Access control (IAM**) page, select **Add** > **Add role assignment**.

    :::image type="content" source="media/prometheus-api-promql/access-control.png" lightbox="media/prometheus-api-promql/access-control.png" alt-text="Screenshot that shows the Azure Monitor workspace Overview page.":::

1. On the **Add role assignment** page, search for **Monitoring**.

1. Select **Monitoring Data Reader**, and then select the **Members** tab.

    :::image type="content" source="media/prometheus-api-promql/add-role-assignment.png" lightbox="media/prometheus-api-promql/add-role-assignment.png" alt-text="Screenshot that shows the Add role assignment page.":::

1. Choose **Select members**.

1. Search for the app that you registered and select it.

1. Choose **Select**.

1. Select **Review + assign**.

    :::image type="content" source="media/prometheus-api-promql/select-members.png" lightbox="media/prometheus-api-promql/select-members.png" alt-text="Screenshot that shows the Add role assignment page with the Select members pane open.":::

You created your app registration and assigned it access to query data from your Azure Monitor workspace. You can now generate a token and use it in a query.

### Request a token

Send the following request in the command prompt or by using a client like Insomnia or the PowerShell `Invoke-RestMethod` command.

```shell
curl -X POST 'https://login.microsoftonline.com/<tenant ID>/oauth2/token' \
-H 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=client_credentials' \
--data-urlencode 'client_id=<your apps client ID>' \
--data-urlencode 'client_secret=<your apps client secret>' \
--data-urlencode 'resource=https://prometheus.monitor.azure.com'
```

#### Sample response body

```JSON
{
    "token_type": "Bearer",
    "expires_in": "86399",
    "ext_expires_in": "86399",
    "expires_on": "1672826207",
    "not_before": "1672739507",
    "resource": "https:/prometheus.monitor.azure.com",
    "access_token": "eyJ0eXAiOiJKV1Qi....gpHWoRzeDdVQd2OE3dNsLIvUIxQ"
}
```

Save the access token from the response for use in the following HTTP requests.

## Query endpoint

Find your Azure Monitor workspace's query endpoint on the Azure Monitor workspace **Overview** page.

:::image type="content" source="./media/prometheus-api-promql/find-query-endpoint.png" lightbox="./media/prometheus-api-promql/find-query-endpoint.png" alt-text="Screenshot that shows the query endpoint on the Azure Monitor workspace Overview page.":::

## Supported APIs

The following queries are supported.

### Instant queries

For more information, see [Instant queries](https://prometheus.io/docs/prometheus/latest/querying/api/#instant-queries).

**Path:** `/api/v1/query`

#### Examples

```
POST https://k8s-02-workspace-abcd.eastus.prometheus.monitor.azure.com/api/v1/query
--header 'Authorization: Bearer <access token>'
--header 'Content-Type: application/x-www-form-urlencoded' 
--data-urlencode 'query=sum( \
    container_memory_working_set_bytes \
    * on(namespace,pod) \
    group_left(workload, workload_type) \
    namespace_workload_pod:kube_pod_owner:relabel{ workload_type="deployment"}) by (pod)'
```

```
GET 'https://k8s02-workspace-abcd.eastus.prometheus.monitor.azure.com/api/v1/query?query=container_memory_working_set_bytes' 
--header 'Authorization: Bearer <access token>'
```

### Range queries

For more information, see [Range queries](https://prometheus.io/docs/prometheus/latest/querying/api/#range-queries).

**Path:** `/api/v1/query_range`

#### Examples

```
GET 'https://k8s02-workspace-abcd.eastus.prometheus.monitor.azure.com/api/v1/query_range?query=container_memory_working_set_bytes&start=2023-03-01T00:00:00.000Z&end=2023-03-20T00:00:00.000Z&step=6h'
--header 'Authorization: Bearer <access token>
```

``` 
POST 'https://k8s02-workspace-abcd.eastus.prometheus.monitor.azure.com/api/v1/query_range' 
--header 'Authorization: Bearer <access token>'
--header 'Content-Type: application/x-www-form-urlencoded' 
--data-urlencode 'query=up' 
--data-urlencode 'start=2023-03-01T20:10:30.781Z' 
--data-urlencode 'end=2023-03-20T20:10:30.781Z' 
--data-urlencode 'step=6h'
```

### Series

For more information, see [Series](https://prometheus.io/docs/prometheus/latest/querying/api/#finding-series-by-label-matchers).

**Path:** `/api/v1/series`

#### Examples

```
POST 'https://k8s02-workspace-abcd.eastus.prometheus.monitor.azure.com/api/v1/series' 
--header 'Authorization: Bearer <access token>
--header 'Content-Type: application/x-www-form-urlencoded' 
--data-urlencode 'match[]=kube_pod_info{pod="bestapp-123abc456d-4nmfm"}'
```

```
GET 'https://k8s02-workspace-abcd.eastus.prometheus.monitor.azure.com/api/v1/series?match[]=container_network_receive_bytes_total{namespace="default-1669648428598"}'
```

### Labels

For more information, see [Labels](https://prometheus.io/docs/prometheus/latest/querying/api/#getting-label-names).

**Path:** `/api/v1/labels`

#### Examples

```
GET 'https://k8s02-workspace-abcd.eastus.prometheus.monitor.azure.com/api/v1/labels'
```

```
POST 'https://k8s02-workspace-abcd.eastus.prometheus.monitor.azure.com/api/v1/labels'
```

### Label values

For more information, see [Label values](https://prometheus.io/docs/prometheus/latest/querying/api/#query.ing-label-values).

**Path:** `/api/v1/label/__name__/values`

> [!NOTE]
> The only supported version of this API is `__name__`, and it returns all metric names. No other `/api/v1/label/<label_name>/values` are supported.

#### Example

```
GET 'https://k8s02-workspace-abcd.eastus.prometheus.monitor.azure.com/api/v1/label/__name__/values'
```

For the full specification of open-source software Prometheus APIs, see [Prometheus HTTP API](https://prometheus.io/docs/prometheus/latest/querying/api/#http-api).

## API limitations

The following limitations are in addition to the limitations that are described in the Prometheus specification:

* Any time-series fetch queries (`/series` or `/query` or `/query_range`) must contain a `\_\_name\_\_` label matcher. That is, each query must be scoped to a metric. There can be only one `\_\_name\_\_` label matcher in a query.

* The query `/series` doesn't support regular expression filter.

* Supported time range:

    * The `/query_range` API supports a time range of 32 days. This length of time is the maximum time range allowed, including range selectors specified in the query itself. For example, the query `rate(http_requests_total[1h]` for the last 24 hours means that data is queried for 25 hours. This number comes from the 24-hour range plus the 1 hour specified in the query itself.
    * The `/series` API fetches data for a maximum 12-hour time range. If `endTime` isn't provided, then `endTime = time.now()`. If the time range is greater than 12 hours, `startTime` is set to `endTime – 12h`.

* The start time and end time provided with `/labels` and `/label/__name__/values` are ignored. All retained data in the Azure Monitor workspace is queried.

* Experimental features such as exemplars aren't supported.

For more information on Prometheus metrics limits, see [Prometheus metrics](../fundamentals/service-limits.md#prometheus-metrics).

[!INCLUDE [prometheus-case-sensitivity.md](includes/prometheus-case-sensitivity.md)]

[!INCLUDE [duplicate timeseries](includes/prometheus-duplicate-timeseries.md)]

## Frequently asked questions

This section provides answers to common questions.

[!INCLUDE [prometheus-faq-i-am-missing-some-metrics](includes/prometheus-faq-i-am-missing-some-metrics.md)]

[!INCLUDE [prometheus-faq-i-am-missing-metrics-with-same-name-different-casing](includes/prometheus-faq-i-am-missing-metrics-with-same-name-different-casing.md)]

[!INCLUDE [prometheus-faq-i-see-gaps-in-metric-data](includes/prometheus-faq-i-see-gaps-in-metric-data.md)]

## Related content

- [Azure Monitor workspace overview](azure-monitor-workspace-overview.md)
- [Manage an Azure Monitor workspace](azure-monitor-workspace-manage.md)
- [Overview of Azure Monitor Managed Service for Prometheus](prometheus-metrics-overview.md)
- [Query Prometheus metrics using Azure workbooks](prometheus-workbooks.md)
