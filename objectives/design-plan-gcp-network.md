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

VPC Specifications
---
* VPC networks, including their associated routes and firewall rules are global resources. They are <i>not</i> associated with any particular region or zone.
* Subnets are regional resources. Each subnet defines a range of IP addresses.
* Traffic to and from instances can be controlled with network :fire: firewall rules. Rules are implemented on the VMs themselves, so traffic can only be controlled and logged as it leaves or arrives at a VM.
* VPC networks can be securely connected in hybrid environments by using Cloud VPN or Cloud Interconnect.
* VPC networks only support IPv4 unicast traffic. They do <b>not</b> support broadcast, multicast or IPv6 traffic <i>within</i> the network.


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