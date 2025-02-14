---
title: Windows.EventLogs.PowershellModule
hidden: true
tags: [Client Artifact]
---

This Artifact will search and extract Module events (Event ID 4103) from
Powershell-Operational Event Logs.

Powershell is commonly used by attackers across all stages of the attack
lifecycle. Although quite noisy Module logging can provide valuable insight.

There are several parameter's available for search leveraging regex.
  - DateAfter enables search for events after this date.
  - DateBefore enables search for events before this date.
  - ContextRegex enables regex search over ContextInfo text field.
  - PayloadRegex enables a regex search over Payload text field.
  - SearchVSS enables VSS search


```yaml
name: Windows.EventLogs.PowershellModule
description: |
  This Artifact will search and extract Module events (Event ID 4103) from
  Powershell-Operational Event Logs.

  Powershell is commonly used by attackers across all stages of the attack
  lifecycle. Although quite noisy Module logging can provide valuable insight.

  There are several parameter's available for search leveraging regex.
    - DateAfter enables search for events after this date.
    - DateBefore enables search for events before this date.
    - ContextRegex enables regex search over ContextInfo text field.
    - PayloadRegex enables a regex search over Payload text field.
    - SearchVSS enables VSS search


author: Matt Green - @mgreen27

reference:
  - https://attack.mitre.org/techniques/T1059/001/
  - https://www.fireeye.com/blog/threat-research/2016/02/greater_visibilityt.html

parameters:
  - name: EventLog
    default: C:\Windows\system32\winevt\logs\Microsoft-Windows-PowerShell%4Operational.evtx
  - name: DateAfter
    description: "search for events after this date. YYYY-MM-DDTmm:hh:ss Z"
    type: timestamp
  - name: DateBefore
    description: "search for events before this date. YYYY-MM-DDTmm:hh:ss Z"
    type: timestamp
  - name: ContextRegex
    description: "regex search over Payload text field."
    type: regex
  - name: PayloadRegex
    description: "regex search over Payload text field."
    type: regex
  - name: SearchVSS
    description: "Add VSS into query."
    type: bool

sources:
  - query: |
        -- Build time bounds
        LET DateAfterTime <= if(condition=DateAfter,
            then=timestamp(epoch=DateAfter), else=timestamp(epoch="1600-01-01"))
        LET DateBeforeTime <= if(condition=DateBefore,
            then=timestamp(epoch=DateBefore), else=timestamp(epoch="2200-01-01"))

        -- Determine target files
        LET files = SELECT *
          FROM if(condition=SearchVSS,
            then= {
              SELECT *
              FROM Artifact.Windows.Search.VSS(SearchFilesGlob=EventLog)
            },
            else= {
              SELECT *, FullPath as Source
              FROM glob(globs=EventLog)
            })

        -- Main query
        LET hits = SELECT *
          FROM foreach(
            row=files,
            query={
              SELECT
                timestamp(epoch=System.TimeCreated.SystemTime) As EventTime,
                System.EventID.Value as EventID,
                System.Computer as Computer,
                System.Security.UserID as SecurityID,
                EventData.ContextInfo as ContextInfo,
                EventData.Payload as Payload,
                Message,
                System.EventRecordID as EventRecordID,
                System.Level as Level,
                System.Opcode as Opcode,
                System.Task as Task,
                Source
              FROM parse_evtx(filename=FullPath)
              WHERE EventID = 4103
                AND EventTime > DateAfterTime
                AND EventTime < DateBeforeTime
                AND if(condition=ContextRegex,
                    then=ContextInfo=~ContextRegex,else=TRUE)
                AND if(condition=PayloadRegex,
                    then=ContextInfo=~PayloadRegex,else=TRUE)
            })
          ORDER BY Source DESC

        -- Group results for deduplication
        LET grouped = SELECT *
          FROM hits
          GROUP BY EventRecordID

        -- Output results
        SELECT
            EventTime,
            EventID,
            Computer,
            SecurityID,
            ContextInfo,
            Payload,
            Message,
            EventRecordID,
            Level,
            Opcode,
            Task,
            Source
        FROM grouped

```
