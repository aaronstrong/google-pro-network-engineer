# 3 Configure Network Services

## 3.1 Configuring load balancing. Considerations include:

1. [Creating backend services](https://cloud.google.com/load-balancing/docs/backend-service)

    There are 6 backend load balancing services. A backend service is either <i>global</i> or <i>regional</i> in scope.
    * External HTTP(S) Load Balancing
    * Internal HTTP(S) Load Balancing
    * SSL Proxy Load Balancing
    * TCP Proxy Load Balancing
    * Internal TCP/UDP Load Balancing
    * External TCP/UDP Network Load Balancing

    [gcloud command to create backend-services](https://cloud.google.com/sdk/gcloud/reference/compute/backend-services/create)
    ```gcloud
    gcloud compute backend-services create
    ```
1. [Firewall and security rules](https://cloud.google.com/vpc/docs/firewalls)

    VPC firewall rules let you allow or deny connections to or from your virtual machine (VM) instances based on a configuration that you specify. Every VPC network functions as a distributed firewall. While firewall rules are defined at the network level, connections are allowed or denied on a per-instance basis.

    In addition to firewall rules that you create, Google Cloud has other rules that can affect incoming (ingress) or outgoing (egress) connections:

    * GCP doesn't allow egress traffic on TCP port 25
    * GCP always allows traffic between VM and metadata server at `169.254.169.254`
    * Every network has 2 implied firewall rules that permit outgoing connections and block incoming connections.
    * The default network has pre-populated rules.

    <b>Specifications</b>

    * Each firewall rule applies to ingress or egres connections, not both.
    * Only supporst IPv4.
    * Actions are `allow` or `deny`
    * VPC firewall is stateful. When a connection is allowed through the firewall in either direction, return traffic matching this connection is also allowed.
    * When creating a rule, you must select a VPC network even though the rule is enforced at the instance level.

    <b>Implied Rules</b>

    Every VPC network has 2 implied firewall rules that exist, cannot be removed but can be overriden with a higher priority and are not shown in the console.

    1. <b>Implied allow egress rule</b>. An `allow` egress rule with the lowest priority (`65535`) with destination of `0.0.0.0/0` that lets any instance send traffic to any destination.
    1. <b>Implied deny ingress rule</b>. A `deny` ingress rule with the lowest priority (`65535`) with source of `0.0.0.0/0` that protects all instances by blocking all incoming connections.

    <b>Pre-populated rules in default network</b>

    Default network has 4 pre-populated rules that can be removed or overriden with a higher priority.

    1. `default-allow-internal`
    Allows ingress connections for all protocols and ports among instances in the network with priority of `65534`.
    1. `default-allow-ssh`
    Allows ingress connections on TCP destination port 22 from any source to any instance in the network. This rule has a priority of `65534`. 
    1. `default-allow-rdp` Allows ingress connections on TCP destination port 3389 from any source to any instance in the network. This rule has a priority of `65534`.
    1. `default-allow-imcp` Allows ingress ICMP traffic from any source to any instance in the network. This rule has a priority of `65534`, and it enables tools such as `ping`.

    <b>Always allowed traffic</b>

    GCP runs a metadata server at `169.254.169.254` that provides the following services:

    * DHCP
    * DNS resolution
    * NTP
    * Instance metadata
    > :star: Note: Firewall rules cannot block traffic that an instance sends to one of its own IP addresses because that traffic never leaves the VM itself. These addresses include its primary internal IP address, any alias IP address, and loopback addresses. Also, if the instance participates as a backend for an internal load balancer, the load balancer's IP address is also assigned to it.
1. HTTP(S) load balancer: including changing URL maps, backend groups, health checks, CDN, and SSL certs
1. TCP and SSL proxy load balancers
1. Network load balancer
1. Internal load balancer
1. Session affinity
1. Capacity scaling

## 3.2 Configuring Cloud CDN. Considerations include:

1. Enabling and disabling Cloud CDN
1. Using cache keys
1. Cache invalidation
1. Signed URLs

## 3.3 Configuring and maintaining Cloud DNS. Considerations include:

1. Managing zones and records
1. Migrating to Cloud DNS
1. DNS Security (DNSSEC)
1. Global serving with Anycast
1. Cloud DNS
1. Internal DNS
1. Integrating on-premises DNS with GCP

## 3.4 Enabling other network services. Considerations include:

1. Health checks for your instance groups
1. Canary (A/B) releases
1. Distributing backend instances using regional managed instance groups
1. Enabling private API access