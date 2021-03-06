# 2. Implementing a GCP Virtual Private Cloud (VPC)

[GCP VPC Core Concepts](https://cloud.google.com/vpc/docs/concepts)

## 2.1 Configuring VPCs.

1. Configuring GCP VPC resources (CIDR range, subnets, firewall rules, etc.)

    * [IP addresses](https://cloud.google.com/compute/docs/ip-addresses)
    
      - Many Google Cloud resources can have internal IP addresses and external IP addresses. Instances use these addresses to communicate with other Google Cloud resources and external systems.
      - Each instance can have only 1 primary internal IP, one ore more secondary IP, and one external IP address
      - Instances in the same VPC can use the internal IP addresses
      - To communicate with the Internet, must use the instances' external IP or a proxy of some kind.
      - The external and internal IP can be static or ephemeral

      
      External IP addresses
      
      - Static external IP addresses are assigned to the project.
      - Static external IP remain attached to stopped instances until removed.
      - Ephemeral external IP addresses remain attached until VM is stopped and restarted
      
      Internal IP addresses
      - Every VM instance can have one primary internal IP address that is unique to the VPC network
      - Static internal IP addresses are assigned to a project long term until they are explicitly released
      - Ephemeral internal IP addresses remain attached to VM instances and forwarding rules until the instance or forwarding rule is deleted.

      Internal DNS
      - You can send packets to an instance by specifying the fully qualified DNS name (FQDN) of the target instance.
      - Instances using zonal DNS: `[INSTANCE_NAME].[ZONE].c.[PROJECT_ID].internal`
      - Instances using global DNS: `[INSTANCE_NAME].c.[PROJECT_ID].internal`

    * [VPC Overview](https://cloud.google.com/vpc/docs/vpc)

      * Subnet Modes

        * Default Mode
          - Created for every project, unless disabled
          - You can disable with the `skipDefaultNetworkCreation` Org policy
          - Deployed with 4 additional firewall rules
          - Great for POC or understanding VPC in GCP
        * Auto Mode
          - Auto mode VPC networks are built with one subnet per region at creation time and automatically receive new subnets in new regions
          - IP ranges for these subnets fit inside the `10.128.0.0/9` CIDR block
        * Custom Mode
          - Can switch from `auto mode` to `custom mode`, but this is a one way switch
          - Best used for production because you have control of IP space and region placement

      | Default | Auto Mode | Custom (<u>best practice</u>) |
      |---------------|-----------|------------------------|
      | Every project | User create| User Created          |
      | 1 subnet per region /20 | 1 subnet per region /20 in the `10.128.0.0/9` CIDR | User specificed region up to /8|
      | 4 default firewall rules | Optional firewall rules | User defined firewall rules
      | 2 implied fw rules | 2 implied fw rules | 2 implied fw rules |
      | subnet expandes to /16 | subnet expands to /16 | Max subnet of /8 |

    * [Subnet Ranges](https://cloud.google.com/vpc/docs/vpc#valid-ranges)
    
      Click the link for additional ranges, but the main point is any RFC1918 private range are valid ranges
    
      | Range     | Description |
      | ----------|-------------|
      | `10.0.0.0/8` | RFC 1918 |
      | `172.16.0.0/12` | RFC 1918 |
      | `192.168.0.0/16`| RFC 1918 |

    * [Restricted Ranges](https://cloud.google.com/vpc/docs/vpc#restricted-ranges)

      Click on the link for more details around what IP ranges are restricted.

    * [Reserved IP Ranges](https://cloud.google.com/vpc/docs/vpc#reserved_ip_addresses_in_every_subnet)

      There are some IP addresses the GCP reserves

      | Reserved IP Address | Description | Example           |
      | --------------------|------------ | ----------------- |
      | Network | First address in the primary range for the subnet | 10.1.2.0 in 10.1.2.0/24 |
      | Default Gateway | Second address in the primary range for the subnet | 10.1.2.1 in 10.1.2.0/24 |
      | Second-to-last addr | Reserved for future use | 10.1.2.254 in 10.1.2.0/24 |
      Broadcast | Last address in the primary IP range for the subnet | 10.1.2.255 in 10.1.2.0/24 |
      |Secondary Range | The first address in the secondary IP range | 172.16.0.0 in 172.16.0.0/24

      > :star:<b>Note</b>: Google Cloud software-defined networking reserves a virtual gateway IP address for the primary IP ranges of each subnet in a VPC network. However, virtual gateways do not respond to ICMP traffic or decrement IP TTL headers.

    * [Routes and Firewalls](https://cloud.google.com/vpc/docs/vpc#affiliated_resources)

      Every new network starts with two types of system-generated routes:
      1. The `default route` defines a path for traffic to leave the VPC network. It provides general internet access to VMs that meet the internet access requirements. It also provides the typical path for Private Google Access.
      1. A `subnet route` is created for each of the IP ranges associated with a subnet. Every subnet has at least one subnet route for its primary IP range. Additional subnet routes are created for a subnet if you add secondary IP ranges to it. Subnet routes define paths for traffic to reach VMs that use the subnets. You cannot remove subnet routes manually.

      >:star:<b>Note:</b> If your VPC network is connected to an on-premises network by using Cloud VPN or Cloud Interconnect, check that subnet ranges do not conflict with on-premises IP addresses. Subnet routes are prioritized first so traffic to the destination range remains in your VPC network, even though it might have been intended for the on-premises network.
1. [VPC peering](https://cloud.google.com/vpc/docs/vpc-peering)
    * VPC Network Peering enables you to connect VPC networks so that workloads in different VPC networks can communicate internally. Traffic stays within Google's network and doesn't traverse the public internet.

      | Advantage | Description |
      | ---| ----|
      | Latency | Connectivity that uses only internal addresses provides lower latency than connectivity that uses external addresses |
      | Security |  Service owners do not need to have their services exposed to the public Internet and deal with its associated risks. |
      | Cost | Google Cloud charges egress bandwidth pricing for networks using external IPs to communicate even if the traffic is within the same zone. If however, the networks are peered they can use internal IPs to communicate and save on those egress costs. |

    * <b>Key Properties</b>
        * Peered VPC networks remain administratively separate. Routes, firewalls, VPNs, and other traffic management tools are administered and applied separately in each of the VPC networks.
        * Each side of a peering association is set up independently.
        * You can exchange custom routes (static and dynamic routes)
        *  given VPC network can peer with multiple VPC networks (25)
        * IAM Permissions: `Owner`, `Editor`, or `Network Admin`
        * Can peer across projects or organizations
    * <b>Restrictions</b>
        * No overlapping CIDR blocks. Will be denied if they overlap
        * Custom Routes are not shared by default
        * Only directly peered networks can communicate. Transitive peering is not supported.

    * <b>Routing Order</b>

      When an instance sends a packet, Google Cloud attempts to select one route from the set of applicable routes according to the following routing order.
        1. Subnet routes and peering subnet routes are considered first because these routes have the most specific destinations.
        1. If the packet cannot be routed by a subnet route or peering subnet route, Google Cloud looks for a static, dynamic, or peering custom route that has the most specific destination.
        1. If more than one route has the same most specific destinanation...[read here](https://cloud.google.com/vpc/docs/routes#routeselection)
        1. If no applicable destination is found, Google Cloud drops the packet, replying with an ICMP destination or network unreachable error.
    * <b>Importing and Exporting Routes</b>

      When you import custom routes, your VPC network can receive custom routes from the peer network only if that network is exporting them. Similarly, if you export custom routes, the peer network can receive custom routes only if that network is importing them.

      When you import or export custom routes, networks only exchange custom routes with direct peers. For example, if your VPC network is peered with multiple networks, routes that your network imports from one peered network aren't exported to the other peered networks.

    * <b>[VPC Network as a transit Network](https://cloud.google.com/vpc/docs/vpc-peering#transit-network)</b>
      
      <img align="center" src="https://cloud.google.com/vpc/images/peering/network-peering-vpc-transit.svg">

        * This example provides the following reachability:

            * VM instances in `network-a` can reach other VMs in `network-b` and systems in the on-premises network.
            * VM instances in `network-b` can reach other VMs in both `network-a` and `network-c`, as well as systems in the on-premises network.
            * VM instances in `network-c` can reach other VMs in `network-b` and systems in the on-premises network.
            * Because VPC Network Peering isn't transitive, VM instances in `network-a` and `network-c` cannot communicate with each other unless you also peer network `network-a` with `network-c`.

1. [Shared VPC](https://cloud.google.com/vpc/docs/shared-vpc#ip_addresses)
    * Shared VPC allows an organization to connect resources from multiple projects to a common Virtual Private Cloud (VPC) network, so that they can communicate with each other securely and efficiently using internal IPs from that network. When you use Shared VPC, you designate a project as a host project and attach one or more other service projects to it.

    * <b>Concepts</b>
        * A host project contains one or more Shared VPC networks.
        * A service project is any project that has been attached to a host project
        * A project cannot be both a host and a service project simultaneously.
        * A service project can belong to only one host project.

    For clarity, a project that does not participate in Shared VPC is called a `standalone project`. This emphasizes that it is neither a host project nor a service project.

    * <b>IAM</b>  :construction_worker:

        Required Administrative Roles
        | IAM Role | Puprose |
        | ---------| --------|
        | Organization Admin | The nominate the Shared VPC Admins |
        | Shared VPC Admin | Shared VPC Admins have the Compute Shared VPC Admin (compute.xpnAdmin) and Project IAM Admin (resourcemanager.projectIamAdmin) roles for the organization or one or more folders. |
        | Service Project Admin | A Shared VPC Admin defines a Service Project Admin by granting an IAM member the Network User (compute.networkUser) role to either the whole host project or select subnets of its Shared VPC networks. |

    * Basic concept
    <img align="center" src="https://cloud.google.com/vpc/images/shared-vpc/shared-vpc-example-concepts.svg">


1. Configure API Access (private, public, NAT GW, proxy)
    * [Private Google Access](https://cloud.google.com/vpc/docs/configure-private-google-access)
      When a VM instance doesn't have a public IP address, but requires access to Google APIs
      * <b>Requirements</b>
          * Private Google Access enabled on a per-subnet basis
          * If using `private.googleapis.com` or `restricted.googleapis.com` domain names, you'll need to create DNS records for these domains.
          * Appropriate routes must exist. These routes must use the default internet gateway as next hop. If using `private.googleapis.com` or `restricted.googleapis.com` you'll need one route per domain.
          * Egress firewalls :fire: must permit traffic to the IP address ranges used by Google APIs and services. 
      * <b>IAM</b> :construction_worker:
          * `Owner`, `Editor`, or `Network Admin` role can create or update subnets and assign IP addresses.
      * <b>Network Configuration</b>

        | Domain and IP address | Supported Services | Example Usage |
        | ----------------------|--------------------|---------------|
        | `private.googleapis.com` 199.36.153.8/30 | Enables API access to most Google APIs and services regardless of whether they are supported by VPC Service Controls. | Use private.googleapis.com to access Google APIs and services using a set of IP addresses only routable from within Google Cloud. Under circumstances that you don't use VPC Service Controls |
        | `restricted.googleapis.com` 199.36.153.4/30 | Enables API access to Google APIs and services that are supported by VPC Service Controls. | Choose `restricted.googleapis.com` when you <b>only</b> need access to Google APIs and services that <b>are</b> supported by VPC Service Controls |
        >:star:<b>Note</b>:  If you need to restrict users to just the Google APIs and services that support VPC Service Controls, use `restricted.googleapis.com.`

        * <b>[DNS Configuation](https://cloud.google.com/vpc/docs/configure-private-google-access#config-domain)</b>

          If you choose either `private.googleapis.com` or `restricted.googleapis.com`, you need to configure DNS such that VMs in your VPC network resolve requests to `*.googleapis.com`.
    * <b>[Cloud NAT](https://cloud.google.com/nat/docs/overview)</b>

      Lets Google Cloud virtual machine (VM) instances without external IP addresses and private Google Kubernetes Engine (GKE) clusters send outbound packets to the internet.

      * Cloud NAT is distributed, software defined managed service. No appliances or VMs.
      * Implements outbound NAT in conjunction with static routes in your VPC whose next hop is <i>default internet gateway</i>
      * Does NOT implement unsolicited inbound connections from the internet.
      * Does not have any firewall rule requirement. Firewall rules are applied directly to network instances, not NAT gateways
      * Depends on Cloud Router
      * Cloud NAT relies on custom static routes whose next hops are the default internet gateway.
      * The Cloud NAT gateway can be configured to provide NAT for the VM network interface's primary internal IP address, alias IP ranges, or both

      > :star: <b>Note</b>: Even though a Cloud NAT gateway is managed by a Cloud Router, Cloud NAT does not use or depend on the Border Gateway Protocol (BGP).

      <b>[NAT IP addressess and Ports](https://cloud.google.com/nat/docs/ports-and-addresses)</b>
        
        <b>NAT IP Addresses</b>
          - A NAT IP address is a regional external IP address, routable on the internet.
          - 2 methods: `Automatic` and `Manual`
        
        <b>Ports</b>
          - Each NAT IP address on a Cloud NAT gateway offers 64,512 TCP source ports and 64,512 UDP source ports
          - Cloud NAT doesn't use the first 1,024 well-known (privileged) ports.
          - The minimum ports per VM instance that you specify (default is 64)

      <b>Benefits</b>
      | Security | Availability | Scalability | Performance |
      | ---------|--------------|-------------|-------------|
      | VMs don't need external IPs | SDN vs VMs | Automatically scale the # of NAT IP addresses | NAT does not reduce bandwidth per VM |
      

1. [Configure VPC flow logs](https://cloud.google.com/vpc/docs/flow-logs)

    VPC Flow Logs records a sample of network flows sent from and received by VM instances, including instances used as Google Kubernetes Engine nodes. These logs can be used for network monitoring, forensics, real-time security analysis, and expense optimization.

    * <b>Keys</b>   
        - Flow logs introduce no delays or performance penalty
        - Only works with VPC networks, not legacy.
        - Enable/disable per subnet and collects all data from VMs
        - Use filters to generate certain logs

## 2.2 Configuring routing

1. [Configuring internal static/dynamic routing](https://cloud.google.com/network-connectivity/docs/router/concepts/overview#static_versus_dynamic_routing)

    Cloud Router isn't a connectivity option, but a service that works over Cloud VPN or Interconnect connections to provide dynamic routing by using the BGP)for your VPC networks. Cloud Router isn't supported for Direct Peering or Carrier Peering connections.

    Cloud Router is required or recommended in the following cases:
    * Required for Cloud NAT
    * Required for Cloud Interconnect and HA VPN
    * A recommended configuration option for Classic VPN

    With static routes, you must create or maintain a routing table. A topology change on either network requires you to manually update static routes. Also, static routes can't automatically reroute traffic if a link fails. Suiteable for small and stable topologies.

    With Cloud Router, you can use BGP to exchange routing information between networks.  Changes are seamlessly implemented without disrupting traffic. This method of exchanging routes through BGP is dynamic routing. Suitable for any size network.

1. Configuring routing policies using tags and priority

    * When you create a route, you specify a VPC network and can specify tags so that the route is only applicable to traffic sent from the primary internal IP address of the network interface attached to that VPC network for instances with matching network tags.

    * Specify a Priority for the route. A priority is only used to to determine routing order if routes have equivalent destinations. See static route parameters for more details.
1. [Configuring NAT (e.g., Cloud NAT, instance-based NAT)](https://cloud.google.com/nat/docs/using-nat#creating_nat)
    * Use the link to configure a Cloud NAT

## 2.3 Configuring and maintaining Google Kubernetes Engine clusters. Considerations include:

1. VPC-native clusters using alias IPs
1. Clusters with shared VPC
1. Private clusters
1. Cluster network policy
1. Adding authorized networks for cluster master access

## 2.4 Configuring and managing firewall rules. Considerations include:

1. [Target network tags and service accounts](https://cloud.google.com/vpc/docs/add-remove-network-tags)
    * You make a firewall rule applicable to specific instances by using target tags and source tags
    * The default target is all instances in the network, but you can specify instances as targets using either target tags or target service accounts.
    * The target tag defines the Google Cloud VMs to which the rule applies.
    * The source tag for an ingress firewall rule applied on a VPC network.
    * You can use a combination of IP ranges and source tags or a combination of IP ranges and source service accounts.
    * You <b>cannot</b> use both network tags and service accounts in the same rule.
1. [Priority](https://cloud.google.com/vpc/docs/firewalls#priority_order_for_firewall_rules)
    * The firewall rule priority is an integer from 0 to 65535, inclusive. Lower integers indicate higher priorities. If you do not specify a priority when creating a rule, it is assigned a priority of 1000.
    * The relative priority of a firewall rule determines whether it is applicable when evaluated against others. The evaluation logic works as follows:
        1. The highest priority rule applicable to a target for a given type of traffic takes precedence.
        1. The highest priority rule applicable for a given protocol and destination port definition takes precedence, even when the protocol and destination port definition is more general.
        1. A rule with a deny action overrides another with an allow action only if the two rules have the same priority.
1. [Network protocols](https://cloud.google.com/vpc/docs/firewalls#protocols_and_ports)
    * You can specify a protocol or a combination of protocols and their destination ports.
    * If you omit both protocols and ports, the firewall rule is applicable for all traffic on any protocol and any destination port.
    * You can only specify destination ports. Source ports not supported.
1. Ingress and egress rules
1. [Firewall logs](https://cloud.google.com/vpc/docs/firewall-rules-logging)
    * Firewall Rules Logging allows you to audit, verify, and analyze the effects of your firewall rules.
    * Logging is also useful if you need to determine how many connections are affected by a given firewall rule.
    * <b>IAM</b>  :construction_worker:
        | Task | Role |
        |------|------|
        | Create/delete/update rules | `Owner`, `Editor`, `Security Admin` |
        | View Logs | `Owner`, `Editor`, `Viewer`, `Security Admin` |






