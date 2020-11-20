# Designing, planning and prototyping a GCP network

## 1.1 Designing the overall network architecture. Considerations include:

* Failover and disaster recovery strategy
* Options for high availability
* DNS strategy (e.g., on-premises, Cloud DNS, GSLB)
* Meeting business requirements
* Choosing the appropriate load balancing options
* Optimizing for latency (e.g., MTU size, caches, CDN)
* Understanding how quotas are applied per project and per VPC
* Hybrid connectivity (e.g., Google private access for hybrid connectivity)
* Container networking
* IAM and security
* SaaS, PaaS, and IaaS services
* Microsegmentation for security purposes (e.g., using metadata, tags)

## 1.2 Designing a Virtual Private Cloud (VPC). Considerations include:

Here is the [Best Practices and reference architecture for VPC design](https://cloud.google.com/solutions/best-practices-vpc-design).

Here is the detailed [VPC network overview](https://cloud.google.com/vpc/docs/vpc).

A Virtual Private Cloud (VPC) network is a virtual version of a physical network, implemented inside of Google's production network. A VPC provides the following:
* Connectivity for VMs including GKE and App Engine flexible instances and other products built on top of Compute Engine VMs
* Offers natvie TCP/UPD Load Balancing and proxy systems for Internal HTTP(S) Load balancing.
* Connects to on-premises networks using Cloud VPN tunnels and Cloud interconnect attachments.

Project can contain multiple VPC networks.

[VPC Specifications](https://cloud.google.com/vpc/docs/vpc#specifications)

---
* VPC networks, including their associated routes and firewall rules are global resources. They are <i>not</i> associated with any particular region or zone.
* Subnets are regional resources. Each subnet defines a range of IP addresses.
* Traffic to and from instances can be controlled with network :fire: firewall rules. Rules are implemented on the VMs themselves, so traffic can only be controlled and logged as it leaves or arrives at a VM.
* VPC networks can be securely connected in hybrid environments by using Cloud VPN or Cloud Interconnect.
* VPC networks only support IPv4 unicast traffic. They do <b>not</b> support broadcast, multicast or IPv6 traffic <i>within</i> the network.

### Networks and subnets

Each VPC network consists of one or more useful IP range partitions called subnets. Each subnet is associated with a region. VPC networks do not have any IP address ranges associated with them. IP ranges are defined for the subnets.

A network must have at least one subnet before you can use it. Auto mode VPC networks create subnets in each region automatically. Custom mode VPC networks start with no subnets, giving you full control over subnet creation. You can create more than one subnet per region

### Subnet Creation Mode

Two types of VPC networks, determined by their subnet creation mode:
* Auto mode, one subnet from each region is automatically created within it. These are automatically created subnets use a set of predefined IP ranges that fit within the `10.128.0.0/9` CIDR block.

* Custom mode, no subnets are automatically created. This type of network provides you with complete control over its subnets and IP ranges. You decide which subnets to create in regions that you choose by using IP ranges you specify.
* You can switch from auto mode to custom mode. One-way conversion; custom mode VPC networks cannot be changed to auto mode VPC networks.
> :star: <b>Important:</b> Production networks should be planned in advance and it's recommended to use custom mode VPC networks.

Default network

---

Each new project starts with a default network, unless you disable it with an Org policy called `compute.skipDefaultNetworkCreation` constraint.

Subnet ranges

---

When you create a subnet, you must define its primary IP address range. The primary internal addresses for the following resources come from the subnet's primary range: VM instances, internal load balancers, and internal protocol forwarding. 

Reserved IP addresses in a subnet

---

Every subnet has four reserved IP addresses in its primary IP range. No reserved IP addresses in the secondary IP ranges.

| Reserved IP addres | Description | Example
| -------------------|-------------|--------
| Network            | First address in the primary range for the subnet | 10.1.2.0 in 10.1.2.0/24
| Default gateway    | Second address in the primary range for the subnet | 10.1.2.1 in 10.1.2.0/24
| Second-to-last address | Second-to-last address in the primary IP range for the subnet that is reserved by Google Cloud or potential future use | 10.1.2.254 in 10.1.2.0/24
| Broadcast | Last address in the primary IP range for the subnet | 10.1.2.255 in 10.1.2.0/24

### Routes and Firewall Rules

Routes

---

Routes in Google Cloud are divided into two categories: system-generated and custom.

Every new network starts with two types of system-generated routes:
* the default route defines a path for traffic to leave the VPC network. It provides a general Internet access to VMs that meet the internet access requirements. It also provides the typical path for Private Google Access.
* A subnet route is create for each of the IP ranges associated with a subnet. Every subnet has at least one subnet route for its primary IP range. Additional subnet routes are created for a subnet if you add secondary IP ranges to it. Subnet routes define paths for traffic to reach VMs that use the subnets. You cannot remove subnet reoutes manually
> :start: <b>Note:</b> If your VPC network is connected to an on-premises network by using Cloud VPN or Cloud Interconnect, check that subnet ranges do not conflict with on-premises IP addresses. [Subnet routes are prioritized first](https://cloud.google.com/vpc/docs/routes#routeselection) so traffic to the destination range remains in your VPC network, even though it might have been intended for the on-premises network.

Dynamic Routing Mode

---

Each VPC network has an associated <i>dynamic routing mode</i> that controls the behavior of all its Cloud Routers. Cloud Routers share routes to your VPC network and learn custom dynamic routes from connected networks when you connect your VPC to another network using a Cloud VPN or Cloud Interconnect.
* <i>Regional dynamic routing</i> is the default. This mode, route to on-premises resources learned bya  given Cloud Router in the VPC netowrk only apply to the subnets in the same region as the Cloud Router. Unless modified by custom advertisements, each Cloud Router only shares the routes to subnets in its region with its on-premises counterpart.
* <i>Global dynamic routing</i> changes the behavior of all Cloud Routers in the network such that the routes to on-premises resources that they learne are available in all sunbets in the VPC network, regardless of region. Unless modified by custom advertisements, each Cloud Router shares routes to all subnets in the VPC, regardless of Region. Unless modified, each Cloud Router shares routes to all subnets in the VPC network with its on-premises counterpart.

You can change the dynamic routing mode from regional to global and vica-versa without restrictions.



* CIDR range for subnets
* IP addressing (e.g., static, ephemeral, private)
* Standalone or shared
* Multiple vs. single
* Multi-zone and multi-region
* Peering
* Firewall (e.g., service account–based, tag-based)
* Routes
* Differences between Google Cloud Networking and other cloud platforms

## 1.3 Designing a hybrid network. Considerations include:

* Using interconnect (e.g., dedicated vs. partner)
* Peering options (e.g., direct vs. carrier)
* IPsec VPN
* Cloud Router
* Failover and disaster recovery strategy (e.g., building high availability with BGP using cloud router)
* Shared vs. standalone VPC interconnect access
* Cross-organizational access
* Bandwidth

## 1.4 Designing a container IP addressing plan for Google Kubernetes Engine 