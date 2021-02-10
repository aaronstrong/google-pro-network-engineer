# Monitoring

## VPC Flow Logs :pencil2: :page_with_curl:
[Overview](https://cloud.google.com/vpc/docs/flow-logs)

VPC Flow Logs records a sample of network flows sent from and received by VM instances including GKE nodes. These logs can be used for network monitoring, forensics, real-time security analysis, and expense optimization. Flow logs are aggregated by connection from Compute Engine VMs and exported in real time. By subscribing to Pub/Sub, you can analyze flow logs using real-time streaming APIs.

### :key: Key Properties

* VPC flow logs have no impact on the network when enabled.
* VPC flow logs are enabled per subnet. If enabled, collects log data from all VMs in that subnet.
* Use filters in VPC flow logs to generate certain logs.
* To log flows between Pods in the same GKE node, you must enable Intranode visibility for the cluster.
* VPC flow log sampling, samples each VM's TCP and UDP flows, inbound and outbound.
* VPC flow logs interact with firewalls by:
    * Egress packets are sampled <i>before egress</i> firewall rules. Even if an egress firewall rule denies outbound packets.
    * Ingress packets sampled <i>after ingress</i> firewall rules. If an ingress firewall rule denies inbound packets, those packets are not sampled by VPC Flow Logs.

### Logs collection

Logs are stored in Logging for <b>30 days</b> by default. If you wish to keep longer you must export them to a bucket, Pub/Sub or BigQuery.

#### Log sampling

Google Cloud samples packets that leave and enter a VM to generate flow logs. Not every packet is captured into its own log record. About 1 out of every 10 packets is captured, but this sampling rate might be lower depending on the VM's load. You cannot adjust this rate.

Balance traffic visibility and storage costs by adjusting the following aspects of log collection
* <b>Aggregation level</b>: Sampled packets for a time interval are aggregated into a single log entry. 
    * 5 seconds (default)
    * 30 seconds
    * 1 minute
    * 5 minutes
    * 10 or 15 minutes
* <b>Sample Rate</b>: <u>Before being written to Logging</u>, the number of logs can be sampled to reduce their number. By default, the log entry volume is scaled by 0.5 (50%), which means that half of entries are kept.
* <b>Filtering</b>: By default all logs in the subnet are collected. Set filters to collect certain matched criteria generated.

## Firewall Logs :fire: :page_with_curl:

[Overview](https://cloud.google.com/vpc/docs/firewall-rules-logging)

Firewall Rules Logging lets you audit, verify, and analyze the effects of your firewall rules. You enable Firewall Rules Logging individually for each firewall rule whose connections you need to log. Firewall Rules Logging is an option for any firewall rule, regardless of the action (allow or deny) or direction (ingress or egress) of the rule.

When you enable logging for a firewall rule, Google Cloud creates an entry called a connection record each time the rule allows or denies traffic. You can view these records in Cloud Logging, and you can export logs to any destination that Cloud Logging export supports.

### :key: Specifications

* Firewall Rules logging only records TCP and UDP connections. Use packet mirroring for other protocols.
* You <i>cannot</i> enable Firewall Rule logging for implied firewall rules.
* Log entries are written from perspective of VMs.

## Admin Logs

## Cloud Monitoring (was Stackdriver Monitoring)
