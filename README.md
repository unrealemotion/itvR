While we understand the importance of uninterrupted service, occasional single-server failures can unfortunately occur. With Azure's VM availability SLA of 99.9% uptime for a single server, we worked to minimize the impact. The disconnection lasted less than 30 minutes, from 9:19 PM, when the initial network issue was detected, to 9:49 PM, when services were fully restored. Fortunately, the failover cluster helped reduce downtime as services seamlessly continued from the other cluster node, allowing us to limit the disruption as much as possible.