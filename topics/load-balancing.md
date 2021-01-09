# Load Balancing

[Load Balancing Overview](https://cloud.google.com/load-balancing/docs/load-balancing-overview)

A load balancer distributes user traffic across multiple instances of your applications.

![](https://cloud.google.com/load-balancing/images/lb-simple-overview.svg)

<b>Summary of Cloud Load Balancers</b>
| Load Balancer Type       | Traffic Type            | Preserve Client IP | Global or Regional                                              | Load Balancing Scheme | Load Balancer destination Ports              | Proxy or pass- through |
|--------------------------|-------------------------|--------------------|-----------------------------------------------------------------|-----------------------|----------------------------------------------|------------------------|
| External HTTP(S)         | HTTP or HTTPS           | No                 | Global                                                          | EXTERNAL              | HTTP on 80 or 8080; HTTPS on 443             | Proxy                  |
| Internal HTTP(S)         | HTTP or HTTPS           | NO                 | Regional                                                        | INTERNAL_MANAGED      | HTTP on 80 or 8080; HTTPS on 443             | Proxy                  |
| SSL Proxy                | TCP with SSL offload    | No                 | Global                                                          | EXTERNAL              | 25,43,110,143,195,443,465,587,700,993,995... | Proxy                  |
| TCP Proxy                | TPC without SSL offload | No                 | Global                                                          | EXTERNAL              | 25,43,110,143,195,443,465,587,700,993,995... | Proxy                  |
| External Network TCP/UDP | TCP or UDP              | Yes                | Regional                                                        | EXTERNAL              | Any                                          | Pass through           |
| Internal TCP/UDP         | TCP or UDP              | Yes                | Regional backends, regional frontends (global access supported) | INTERNAL              | Any                                          | Pass through           |

<b>Types of Cloud Load Balancing</b>

| Internal or External | Regional/Global                       | Network Tiers  | Proxy/Pass through | Traffic Type | LB Type          |
|----------------------|---------------------------------------|----------------|--------------------|--------------|------------------|
| Internal             | Regional                              | Premium Only   | Pass-through       | TCP or UDP   | Internal TCP/UDP |
|                      | Regional                              | Premium Only   | Proxy              | HTTP(S)      | Internal HTTP(S) |
| External             | Regional                              | Premium or Std | Pass-through       | TCP/UDP      | TCP/UDP Network  |
|                      | Global in premium tier                | Premium or Std | Proxy              | TCP          | TCP Proxy        |
|                      | Effectively regional in Standard Tier | Premium or Std | Proxy              | SSL          | SSL Proxy        |
|                      | Effectively regional in Standard Tier | Premium or Std | Proxy              | HTTP(S)      | External HTTP(S) |

<b>Global vs Regional</b>

* Use <b>global load balancing</b> when your backends are distributed across multiple regions, your users need access to the same applications and content, and you want to provide access by using a single anycast IP address. Global load balancing can also provide IPv6 termination.

* Use <b>regional load balancing</b> when your backends are in one region, and you only require IPv4 termination.

<b>External vs Internal</b>

* External load balancers distribute traffic coming from the internet to your Google Cloud Virtual Private Cloud (VPC) network. Global load balancing requires that you use the Premium Tier of Network Service Tiers. For regional load balancing, you can use Standard Tier.

* Internal load balancers distribute traffic to instances inside of Google Cloud.

![](https://cloud.google.com/load-balancing/images/choose-lb-4.svg)

<b>Traffic Type</b>

* For <b>HTTP(S)</b> traffic, use:
    * External HTTP(S) Load Balancing
    * Internal HTTP(S) Load Balancing
* For <b>TCP</b> traffic, use:
    * TCP Proxy Load Balancing
    * Network Load Balancing
    * Internal TCP/UDP Load Balancing
* For <b>UDP</b> traffic, use:
    * Network Load Balancing
    * Internal TCP/UDP Load Balancing

<b>Choosing a load balancer</b>

![](https://cloud.google.com/load-balancing/images/choose-lb.svg)

[Load Balancing Backend Service Overview](https://cloud.google.com/load-balancing/docs/backend-service)

There are 6 backend load balancing services. A backend service is either <i>global</i> or <i>regional</i> in scope.
* External HTTP(S) Load Balancing
* Internal HTTP(S) Load Balancing
* SSL Proxy Load Balancing
* TCP Proxy Load Balancing
* Internal TCP/UDP Load Balancing
* External TCP/UDP Network Load Balancing

## External HTTP(S) Load Balancer

[Overview](https://cloud.google.com/load-balancing/docs/https)

### About

Google Cloud HTTP(S) Load Balancing is a global, proxy-based Layer 7 load balancer that enables you to run and scale your services worldwide behind a single external IP address. External HTTP(S) Load Balancing is implemented on Google Front Ends (GFEs). GFEs are distributed globally and operate together using Google's global network and control plane.

### Use Cases

## TCP Global Load Balancer

### About

### Use Cases

## SSL Global Load Balancer
### About

### Use Cases

## Internal Load Balancer

### About

### Use Cases

## Managed Instance Groups

### About

### Use Cases

## Network endpoint groups

## Load Balancing Components