---
title: Azure service health notifications
description: Service health notifications allow you to view service health messages published by Microsoft Azure.
ms.topic: article
ms.date: 06/12/2025

---

# Service health notifications

Service health notifications created through Azure, contain information about the resources under your subscription. These notifications are a subclass of activity log events, and can also be found in the activity log. Service health notifications can be informational or actionable, depending on the class.

There are various classes of service health notifications:  

- **Action required:** Azure might notice something unusual happens on your account, and work with you to remedy the issue. Azure sends you a notification, either detailing the actions you need to take or how to contact Azure engineering or support.  
- **Incident:** An event that impacts service is currently affecting one or more of the resources in your subscription.  
- **Maintenance:** A planned maintenance activity that might affect one or more of the resources under your subscription.  
- **Information:** Potential optimizations that might help improve your resource use. 
- **Security:** Urgent security-related information regarding your solutions that run on Azure.

Each service health notification includes details on the scope and impact to your resources. Details include:

Property name | Description
-------- | -----------
channels | One of the following values: **Admin** or **Operation**.
correlationId | Usually a GUID in the string format. Events that belong to the same action usually share the same correlationId.
eventDataId | The unique identifier of an event.
eventName | The title of an event.
level | The level of an event
resourceProviderName | The name of the resource provider for the impacted resource.
resourceType| The type of resource of the impacted resource.
subStatus | Usually the HTTP status code of the corresponding REST call, but can also include other strings describing a substatus. For example: OK (HTTP Status Code: 200), Created (HTTP Status Code: 201), Accepted (HTTP Status Code: 202), No Content (HTTP Status Code: 204), Bad Request (HTTP Status Code: 400), Not Found (HTTP Status Code: 404), Conflict (HTTP Status Code: 409), Internal Server Error (HTTP Status Code: 500), Service Unavailable (HTTP Status Code: 503), and Gateway Timeout (HTTP Status Code: 504).
eventTimestamp | Timestamp when the event was generated, and the Azure service processing the request corresponding to the event.
submissionTimestamp | Timestamp when the event became available for querying.
subscriptionId | The Azure subscription in which this event was logged.
status | String describing the status of the operation. Values are: **Active**, and **Resolved**.
operationName | The name of the operation.
category | This property is always **ServiceHealth**.
resourceId | The Resource ID of the impacted resource.
Properties.title | The localized title for this communication. English is the default.
Properties.communication | The localized details of the communication with HTML markup. English is the default.
Properties.incidentType | One of the following values: **ActionRequired**, **Informational**, **Incident**, **Maintenance**, or **Security**.
Properties.trackingId | The incident this event is associated with. Use this tracking ID to correlate the events related to an incident.
Properties.impactedServices | An escaped JSON blob that describes the services and regions impacted by the incident. The property includes a list of services, each of which has a **ServiceName**, and a list of impacted regions, each of which has a **RegionName**.
Properties.defaultLanguageTitle | The communication in English.
Properties.defaultLanguageContent | The communication in English as either HTML markup or plain text.
Properties.stage | The possible values for **Incident**, and **Security** are **Active,** **Resolved, or **RCA**. For **ActionRequired** or **Informational** the only value is **Active.** For **Maintenance** they are: **Active**, **Planned**, **InProgress**, **Canceled**, **Rescheduled**, **Resolved**, or **Complete**.
Properties.communicationId | The communication this event is associated with.

### Details on service health level information

**Action Required** (properties.incidentType == ActionRequired)
- Informational - Administrator action is required to prevent impact to existing services.
    
**Maintenance** (properties.incidentType == Maintenance)
- Warning - Emergency maintenance
- Informational - Standard planned maintenance

**Information** (properties.incidentType == Informational)
- Informational - Administrator might be required to prevent impact to existing services.

**Security** (properties.incidentType == Security)
- Warning - Security advisory that affects existing services and might require administrator action.
- Informational - Security advisory that affects existing services.

**Service Issues** (properties.incidentType == Incident)
- Error - Widespread issues accessing multiple services across multiple regions are impacting a broad set of customers.
- Warning - Issues accessing specific services and/or specific regions are impacting a subset of customers.
- Informational - Issues impacting management operations and/or latency, not impacting service availability.
