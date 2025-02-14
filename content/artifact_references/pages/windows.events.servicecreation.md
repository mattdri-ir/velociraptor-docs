---
title: Windows.Events.ServiceCreation
hidden: true
tags: [Client Event Artifact]
---

Monitor for creation of new services.

New services are typically created by installing new software or
kernel drivers. Attackers will sometimes install a new service to
either insert a malicious kernel driver or as a persistence
mechanism.

This event monitor extracts the service creation events from the
event log and records them on the server.


```yaml
name: Windows.Events.ServiceCreation
description: |
  Monitor for creation of new services.

  New services are typically created by installing new software or
  kernel drivers. Attackers will sometimes install a new service to
  either insert a malicious kernel driver or as a persistence
  mechanism.

  This event monitor extracts the service creation events from the
  event log and records them on the server.
type: CLIENT_EVENT

parameters:
  - name: systemLogFile
    default: >-
      C:/Windows/System32/Winevt/Logs/System.evtx

sources:
 - precondition:
     SELECT OS from info() where OS = "windows"

   query: |
        SELECT System.TimeCreated.SystemTime as Timestamp,
               System.EventID.Value as EventID,
               EventData.ImagePath as ImagePath,
               EventData.ServiceName as ServiceName,
               EventData.ServiceType as Type,
               System.Security.UserID as UserSID,
               EventData as _EventData,
               System as _System
        FROM watch_evtx(filename=systemLogFile) WHERE EventID = 7045

```
