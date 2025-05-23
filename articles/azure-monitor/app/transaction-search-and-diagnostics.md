---
title: Transaction Search and Diagnostics
description: This article explains Application Insights end-to-end transaction diagnostics and how to search and filter raw telemetry sent by your web app.
ms.topic: how-to
ms.date: 01/31/2025
ms.reviewer: cogoodson
---

# Transaction Search and Diagnostics

Azure Monitor Application Insights offers Transaction Search for pinpointing specific telemetry items and Transaction Diagnostics for comprehensive end-to-end transaction analysis.

**Transaction Search**: This experience enables users to locate and examine individual telemetry items such as page views, exceptions, and web requests. Additionally, it offers the capability to view log traces and events coded into the application. It identifies performance issues and errors within the application.

**Transaction Diagnostics**: Quickly identify issues in components through comprehensive insight into end-to-end transaction details, including dependencies and exceptions. Access this feature via the Search interface by choosing an item from the search results.

## [Transaction Search](#tab/transaction-search)

Transaction search is a feature of [Application Insights](./app-insights-overview.md) that you use to find and explore individual telemetry items, such as page views, exceptions, or web requests. You can also view log traces and events that you code.

For more complex queries over your data, use [Log Analytics](../logs/log-analytics-tutorial.md).

## Where do you see Search?

You can find **Search** in the Azure portal or Visual Studio.

### In the Azure portal

You can open transaction search from the Application Insights **Overview** tab of your application. You can also select **Search** under **Investigate** on the left menu.

:::image type="content" source="./media/search-and-transaction-diagnostics/view-custom-events.png" lightbox="./media/search-and-transaction-diagnostics/view-custom-events.png" alt-text="Screenshot that shows the Search tab.":::

Go to the **Event types** dropdown menu to see a list of telemetry items such as server requests, page views, and custom events you coded. The top of the **Results** list has a summary chart showing counts of events over time.

Back out of the dropdown menu or select **Refresh** to get new events.

### In Visual Studio

In Visual Studio, there's also an **Application Insights Search** window. It's most useful for displaying telemetry events generated by the application that you're debugging. But it can also show the events collected from your published app at the Azure portal.

Open the **Application Insights Search** window in Visual Studio:

:::image type="content" source="./media/search-and-transaction-diagnostics/32.png" lightbox="./media/search-and-transaction-diagnostics/32.png" alt-text="Screenshot that shows Visual Studio open to Application Insights Search.":::

The **Application Insights Search** window has features similar to the web portal:

:::image type="content" source="./media/search-and-transaction-diagnostics/34.png" lightbox="./media/search-and-transaction-diagnostics/34.png" alt-text="Screenshot that shows Visual Studio Application Insights Search window.":::

The **Track Operation** tab is available when you open a request or a page view. An "operation" is a sequence of events associated with a single request or page view. For example, dependency calls, exceptions, trace logs, and custom events might be part of a single operation. The **Track Operation** tab shows graphically the timing and duration of these events in relation to the request or page view.

## Inspect individual items

Select any telemetry item to see key fields and related items.

:::image type="content" source="./media/search-and-transaction-diagnostics/telemetry-item.png" lightbox="./media/search-and-transaction-diagnostics/telemetry-item.png" alt-text="Screenshot that shows an individual dependency request.":::

The end-to-end transaction details view opens.

## Filter event types

Open the **Event types** dropdown menu and choose the event types you want to see. If you want to restore the filters later, select **Reset**.

The event types are:

* **Trace**: [Diagnostic logs](./asp-net-trace-logs.md) including TrackTrace, log4Net, NLog, and System.Diagnostic.Trace calls.
* **Request**: HTTP requests received by your server application including pages, scripts, images, style files, and data. These events are used to create the request and response overview charts.
* **Page View**: [Telemetry sent by the web client](./javascript.md) used to create page view reports.
* **Custom Event**: If you inserted calls to `TrackEvent()` to [monitor usage](./api-custom-events-metrics.md), you can search them here.
* **Exception**: Uncaught [exceptions in the server](./asp-net-exceptions.md), and the exceptions that you log by using `TrackException()`.
* **Dependency**: [Calls from your server application](./asp-net-dependencies.md) to other services such as REST APIs or databases, and AJAX calls from your [client code](./javascript.md).
* **Availability**: Results of [availability tests](availability-overview.md)

## Filter on property values

You can filter events on the values of their properties. The available properties depend on the event types you selected. Select **Filter** :::image type="content" source="./media/search-and-transaction-diagnostics/filter-icon.png" lightbox="./media/search-and-transaction-diagnostics/filter-icon.png" alt-text="Filter icon"::: to start.

Choosing no values of a particular property has the same effect as choosing all values. It switches off filtering on that property.

Notice that the counts to the right of the filter values show how many occurrences there are in the current filtered set.

## Find events with the same property

To find all the items with the same property value, either enter it in the **Search** box or select the checkbox when you look through properties on the **Filter** tab.

:::image type="content" source="./media/search-and-transaction-diagnostics/filter-property.png" lightbox="./media/search-and-transaction-diagnostics/filter-property.png" alt-text="Screenshot that shows selecting the checkbox of a property on the Filter tab.":::

## Search the data

> [!NOTE]
> To write more complex queries, open [Logs (Analytics)](../logs/log-analytics-tutorial.md) at the top of the **Search** pane.
>

You can search for terms in any of the property values. This capability is useful if you write [custom events](./api-custom-events-metrics.md) with property values.

You might want to set a time range because searches over a shorter range are faster.

:::image type="content" source="./media/search-and-transaction-diagnostics/search-property.png" lightbox="./media/search-and-transaction-diagnostics/search-property.png" alt-text="Screenshot that shows opening a diagnostic search.":::

Search for complete words, not substrings. Use quotation marks to enclose special characters.

| String | *Not* found | Found |
| --- | --- | --- |
| HomeController.About |`home`<br/>`controller`<br/>`out` | `homecontroller`<br/>`about`<br/>`"homecontroller.about"`|
|United States|`Uni`<br/>`ted`|`united`<br/>`states`<br/>`united AND states`<br/>`"united states"`

You can use the following search expressions:

| Sample query | Effect |
| --- | --- |
| `apple` |Find all events in the time range whose fields include the word `apple`. |
| `apple AND banana` <br/>`apple banana` |Find events that contain both words. Use capital `AND`, not `and`. <br/>Short form. |
| `apple OR banana` |Find events that contain either word. Use `OR`, not `or`. |
| `apple NOT banana` |Find events that contain one word but not the other. |

## Sampling

If your app generates significant telemetry and uses ASP.NET SDK version 2.0.0-beta3 or later, it automatically reduces the volume sent to the portal through adaptive sampling. This module sends only a representative fraction of events. It selects or deselects events related to the same request as a group, allowing you to navigate between related events.

Learn about [sampling](./sampling.md).

## Create work item

You can create a bug in GitHub or Azure DevOps with the details from any telemetry item.

Go to the end-to-end transaction detail view by selecting any telemetry item. Then select **Create work item**.

:::image type="content" source="./media/search-and-transaction-diagnostics/work-item.png" lightbox="./media/search-and-transaction-diagnostics/work-item.png" alt-text="Screenshot that shows Create work item.":::

The first time you do this step, you're asked to configure a link to your Azure DevOps organization and project. You can also configure the link on the **Work Items** tab.

## Send more telemetry to Application Insights

In addition to the out-of-the-box telemetry sent by Application Insights SDK, you can:

* Capture log traces from your favorite logging framework in [.NET](./asp-net-trace-logs.md) or [Java](./opentelemetry-add-modify.md?tabs=java). This means you can search through your log traces and correlate them with page views, exceptions, and other events.

* [Write code](./api-custom-events-metrics.md) to send custom events, page views, and exceptions.

Learn how to [send logs and custom telemetry to Application Insights](./asp-net-trace-logs.md).

## [Transaction Diagnostics](#tab/transaction-diagnostics)

The unified diagnostics experience automatically correlates server-side telemetry from across all your Application Insights monitored components into a single view. It doesn't matter if you have multiple resources. Application Insights detects the underlying relationship and allows you to easily diagnose the application component, dependency, or exception that caused a transaction slowdown or failure.

## What is a component?

Components are independently deployable parts of your distributed or microservice application. Developers and operations teams have code-level visibility or access to telemetry generated by these application components.

* Components are different from "observed" external dependencies, such as SQL and event hubs, which your team or organization might not have access to (code or telemetry).
* Components run on any number of server, role, or container instances.
* Components can be separate Application Insights instrumentation keys, even if subscriptions are different. Components also can be different roles that report to a single Application Insights instrumentation key. The new experience shows details across all components, regardless of how they were set up.

> [!NOTE]
> Are you missing the related item links? All the related telemetry is on the left side in the [top](#cross-component-transaction-chart) and [bottom](#all-telemetry-with-this-operation-id) sections.

## Transaction diagnostics experience

This view has four key parts:

- a results list
- a cross-component transaction chart
- a time-sequence list of all telemetry related to this operation
- the details pane for any selected telemetry item

:::image type="content" source="media/search-and-transaction-diagnostics/4-parts-cross-component.png" lightbox="media/search-and-transaction-diagnostics/4-parts-cross-component.png" alt-text="Screenshot that shows the four key parts of the view.":::

## Cross-component transaction chart

This chart provides a timeline with horizontal bars during requests and dependencies across components. Any exceptions that are collected are also marked on the timeline.

- The top row on this chart represents the entry point. It's the incoming request to the first component called in this transaction. The duration is the total time taken for the transaction to complete.
- Any calls to external dependencies are simple noncollapsible rows, with icons that represent the dependency type.
- Calls to other components are collapsible rows. Each row corresponds to a specific operation invoked at the component.
- By default, the request, dependency, or exception that you selected appears to the side. Select any row to see its [details](#details-of-the-selected-telemetry).

> [!NOTE]
> Calls to other components have two rows. One row represents the outbound call (dependency) from the caller component. The other row corresponds to the inbound request at the called component. The leading icon and distinct styling of the duration bars help differentiate between them.

## All telemetry with this Operation ID

This section shows a flat list view in a time sequence of all the telemetry related to this transaction. It also shows the custom events and traces that aren't displayed in the transaction chart. You can filter this list to telemetry generated by a specific component or call. You can select any telemetry item in this list to see corresponding [details on the side](#details-of-the-selected-telemetry).

:::image type="content" source="media/search-and-transaction-diagnostics/all-telemetry-drawer-opened.png" lightbox="media/search-and-transaction-diagnostics/all-telemetry-drawer-opened.png" alt-text="Screenshot that shows the time sequence of all telemetry.":::

## Details of the selected telemetry

This collapsible pane shows the detail of any selected item from the transaction chart or the list. **Show all** lists all the standard attributes that are collected. Any custom attributes are listed separately under the standard set. Select the ellipsis button (...) under the **Call Stack** trace window to get an option to copy the trace. **Open profiler traces** and **Open debug snapshot** show code-level diagnostics in corresponding detail panes.

:::image type="content" source="media/search-and-transaction-diagnostics/exception-detail.png" lightbox="media/search-and-transaction-diagnostics/exception-detail.png" alt-text="Screenshot that shows exception details.":::

## Search results

This collapsible pane shows the other results that meet the filter criteria. Select any result to update the respective details of the preceding three sections. We try to find samples that are most likely to have the details available from all components, even if sampling is in effect in any of them. These samples are shown as suggestions.

:::image type="content" source="media/search-and-transaction-diagnostics/search-results.png" lightbox="media/search-and-transaction-diagnostics/search-results.png" alt-text="Screenshot that shows search results.":::

## .NET Profiler and Snapshot Debugger

[.NET Profiler](./profiler-overview.md) or [Snapshot Debugger](snapshot-debugger.md) help with code-level diagnostics of performance and failure issues. With this experience, you can see .NET Profiler traces or snapshots from any component with a single selection.

If you can't get the .NET Profiler working, contact serviceprofilerhelp\@microsoft.com.

If you can't get Snapshot Debugger working, contact snapshothelp\@microsoft.com.

:::image type="content" source="media/search-and-transaction-diagnostics/profiler-traces.png" lightbox="media/search-and-transaction-diagnostics/profiler-traces.png" alt-text="Screenshot that shows .NET Profiler integration.":::

---

## See also

* To review frequently asked questions (FAQ), see [Transaction Search FAQ](application-insights-faq.yml#transaction-search) and [Transaction Diagnostics FAQ](application-insights-faq.yml#transaction-diagnostics) 
* [Write complex queries in Analytics](../logs/log-analytics-tutorial.md)
* [Send logs and custom telemetry to Application Insights](./asp-net-trace-logs.md)
* [Availability overview](availability-overview.md)
