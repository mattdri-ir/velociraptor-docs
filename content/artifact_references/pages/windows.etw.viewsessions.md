---
title: Windows.ETW.ViewSessions
hidden: true
tags: [Client Artifact]
---

This artifact enumerates all ETW sessions and optionally kills dangling ones


```yaml
name: Windows.ETW.ViewSessions
description: |
  This artifact enumerates all ETW sessions and optionally kills dangling ones

required_permissions:
  - EXECVE

precondition: SELECT OS From info() where OS = 'windows'
parameters:
  - name: SessionRegex
    default: "Velociraptor"
    type: regex
  - name: KillMatching
    type: bool
    description: If set will kill the relevant sessions.


sources:
  - query: |
      SELECT * FROM foreach(row={
         SELECT Stdout, parse_string_with_regex(string=Stdout, regex="(^[^ ]+)").g1 AS SessionName
         from execve(argv=["logman", "query", "-ets"], sep="\n")
         WHERE Stdout =~ "Running" AND SessionName =~ SessionRegex
      }, query={
         SELECT * FROM if(condition=KillMatching,
         then={
             SELECT SessionName, Stdout FROM execve(argv=["logman", "stop", SessionName, "-ets"])
         }, else={
             SELECT SessionName FROM scope()
         })
      })

```
