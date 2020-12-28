# 2. Implementing a GCP Virtual Private Cloud (VPC)

## 2.1 Configuring VPCs.

1. Configuring GCP VPC resources (CIDR range, subnets, firewall rules, etc.)

 [GCP VPC Core Concepts](https://cloud.google.com/vpc/docs/concepts)

  * [IP addresses](https://cloud.google.com/compute/docs/ip-addresses)
  
    - Many Google Cloud resources can have internal IP addresses and external IP addresses. Instances use these addresses to communicate with other Google Cloud resources and external systems.
    - Each instance can have only 1 primary internal IP, one ore more secondary IP, and one external IP address
    - Instances in the same VPC can use the internal IP addresses
    - To communicate with the Internet, must use the instances' external IP or a proxy of some kind.
    - The external and internal IP can be static or ephemeral
    
    * External IP addresses
    
     - Static external IP addresses are assigned to the project.
     - Static external IP remain attached to stopped instances until removed.
     - Ephemeral external IP addresses remain attached until VM is stopped and restarted
     
   * Internal IP addresses
