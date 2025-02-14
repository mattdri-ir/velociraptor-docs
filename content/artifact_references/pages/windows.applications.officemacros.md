---
title: Windows.Applications.OfficeMacros
hidden: true
tags: [Client Artifact]
---

Office macros are a favourite initial infection vector. Many users
click through the warning dialogs.

This artifact scans through the given directory glob for common
office files. We then try to extract any embedded macros by parsing
the OLE file structure.

If a macro calls an external program (e.g. Powershell) this is very
suspicious!


```yaml
name: Windows.Applications.OfficeMacros
description: |
  Office macros are a favourite initial infection vector. Many users
  click through the warning dialogs.

  This artifact scans through the given directory glob for common
  office files. We then try to extract any embedded macros by parsing
  the OLE file structure.

  If a macro calls an external program (e.g. Powershell) this is very
  suspicious!

parameters:
  - name: officeExtensions
    default: "*.{xls,xlsm,doc,docx,ppt,pptm}"
  - name: officeFileSearchGlob
    default: C:\Users\**\
    description: The directory to search for office documents.

sources:
  - query: |
        SELECT * FROM foreach(
           row={
              SELECT FullPath FROM glob(globs=officeFileSearchGlob + officeExtensions)
           },
           query={
               SELECT * from olevba(file=FullPath)
           })

```
