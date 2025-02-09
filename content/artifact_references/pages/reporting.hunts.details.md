---
title: Reporting.Hunts.Details
hidden: true
tags: [Server Artifact]
---

Report details about which client ran each hunt, how long it took
and if it has completed.


<pre><code class="language-yaml">
name: Reporting.Hunts.Details
description: |
  Report details about which client ran each hunt, how long it took
  and if it has completed.

type: SERVER

sources:
  - query: |
        LET hunts = SELECT basename(path=hunt_id) as hunt_id,
            create_time,
            hunt_description
        FROM hunts() order by create_time desc limit 6

        LET flows = select hunt_id,
          hunt_description,
          Fqdn,
          ClientId,
          { SELECT os_info.system FROM clients(search=ClientId) } as OS,
          timestamp(epoch=Flow.create_time/1000000) as create_time,
          basename(path=Flow.session_id) as flow_id,
          (Flow.active_time - Flow.create_time) / 1000000 as Duration,
          format(format='%v', args=[Flow.state]) as State
        FROM hunt_flows(hunt_id=hunt_id) order by create_time desc

        SELECT * from foreach(row=hunts, query=flows)

</code></pre>

