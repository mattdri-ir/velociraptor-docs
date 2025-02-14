---
title: Linux.Debian.Packages
hidden: true
tags: [Client Artifact]
---

Parse dpkg status file.

```yaml
name: Linux.Debian.Packages
description: Parse dpkg status file.
parameters:
  - name: linuxDpkgStatus
    default: /var/lib/dpkg/status
sources:
  - precondition: |
      SELECT OS From info() where OS = 'linux'
    query: |
        /* First pass - split file into records start with
           Package and end with \n\n.

           Then parse each record using multiple RegExs.
        */
        LET packages = SELECT parse_string_with_regex(
            string=Record,
            regex=['Package:\\s(?P<Package>.+)',
                   'Installed-Size:\\s(?P<InstalledSize>.+)',
                   'Version:\\s(?P<Version>.+)',
                   'Source:\\s(?P<Source>.+)',
                   'Architecture:\\s(?P<Architecture>.+)']) as Record
            FROM parse_records_with_regex(
                   file=linuxDpkgStatus,
                   regex='(?sm)^(?P<Record>Package:.+?)\\n\\n')

        SELECT Record.Package as Package,
               atoi(string=Record.InstalledSize) as InstalledSize,
               Record.Version as Version,
               Record.Source as Source,
               Record.Architecture as Architecture from packages

```
