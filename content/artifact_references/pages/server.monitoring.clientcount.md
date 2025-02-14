---
title: Server.Monitoring.ClientCount
hidden: true
tags: [Server Event Artifact]
---

An artifact that sends an email every hour of the current state of
the deployment.


```yaml
name: Server.Monitoring.ClientCount

description: |
   An artifact that sends an email every hour of the current state of
   the deployment.

type: SERVER_EVENT

parameters:
   - name: EmailAddress
     default: admin@example.com
   - name: CCAddress
     default:
   - name: Subject
     default: "Deployment statistics for Velociraptor"
   - name: Period
     default: "3600"

sources:
  - query: |
      LET metrics = SELECT * FROM Artifact.Server.Monitor.VeloMetrics()

      SELECT * FROM foreach(
        row={
            SELECT * FROM clock(period=atoi(string=Period))
        },
        query={
             SELECT * FROM mail(
                to=EmailAddress,
                cc=CCAddress,
                subject=Subject,
                period=60,
                body=format(format='Total clients currently connected %v',
                     args=[metrics.client_comms_current_connections])
            )
        })

```
