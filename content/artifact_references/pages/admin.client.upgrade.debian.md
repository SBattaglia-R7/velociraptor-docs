---
title: Admin.Client.Upgrade.Debian
hidden: true
tags: [Client Artifact]
---

Remotely push new client updates to Debian hosts.

NOTE: This artifact requires that you supply a client Debian package using the
tools interface or using the "debian client" command. Simply click on the tool
in the GUI and upload a package.


<pre><code class="language-yaml">
name: Admin.Client.Upgrade.Debian
description: |
  Remotely push new client updates to Debian hosts.

  NOTE: This artifact requires that you supply a client Debian package using the
  tools interface or using the "debian client" command. Simply click on the tool
  in the GUI and upload a package.

tools:
  - name: VelociraptorDebian

parameters:
  - name: SleepDuration
    default: "600"
    type: int
    description: |
      The package is typically large and we do not want to
      overwhelm the server so we stagger the download over this many
      seconds.

sources:
  - precondition:
      SELECT OS From info() where OS = 'Linux'

    query:  |
      // Force the file to be copied to the real temp directory since
      // we are just about to remove the Tools directory.
      LET bin &lt;= SELECT copy(filename=OSPath,
          dest=expand(path="/tmp/") + basename(path=OSPath)) AS Dest
      FROM Artifact.Generic.Utils.FetchBinary(
         ToolName="VelociraptorLinux", IsExecutable=FALSE,
         SleepDuration=SleepDuration)

      // Call the binary and return all its output in a single row.
      // If we fail to download the binary we do not run the command.
      SELECT * FROM foreach(row=bin,
      query={
         SELECT * FROM execve(
              argv=["dpkg", "-i", Dest],
              length=10000000)
      })

</code></pre>

