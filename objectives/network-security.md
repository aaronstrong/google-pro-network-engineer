# 3 Configuring network services

## 3.1 Configuring load balancing. Considerations include

1. Creating backend services

* [Backend Services Overview](https://cloud.google.com/load-balancing/docs/backend-service)
* [Using gcloud](https://cloud.google.com/sdk/gcloud/reference/compute/backend-services/create)
1. [Firewall and security rules](https://cloud.google.com/vpc/docs/firewalls)
1. HTTP(S) load balancer: including changing URL maps, backend groups, health checks, CDN, and SSL certs
* [External HTTP(S) Load Balancing Overview](https://cloud.google.com/load-balancing/docs/https)
* [Multi-region, content based HTTPS Load Balancer](https://cloud.google.com/load-balancing/docs/https/setting-up-https)
* [Setting up external HTTPS Load Balancer](https://cloud.google.com/load-balancing/docs/https/setting-up-https)
1. TCP and SSL proxy load balancers
* [TCP Proxy Load Balancer Overview](https://cloud.google.com/load-balancing/docs/tcp)
* [SSL Proxy Load Balancer](https://cloud.google.com/load-balancing/docs/ssl)
1. Network load balancer
* [External HTTP(S) Load Balancing Overview](https://cloud.google.com/load-balancing/docs/network)
* [Cloud Load Balancing Overview](https://cloud.google.com/load-balancing/docs/load-balancing-overview)
![](https://cloud.google.com/load-balancing/images/choose-lb-5.svg)
* [Setting up network Load Balancer](https://cloud.google.com/load-balancing/docs/network/setting-up-network)
1. Internal load balancer
* [Internal Load Balancer Overview](https://cloud.google.com/load-balancing/docs/internal)
1. Session affinity
* [Session Affinity](https://cloud.google.com/load-balancing/docs/backend-service#session_affinity)
1. [Capacity scaling](https://cloud.google.com/compute/docs/load-balancing-and-autoscaling#http_load_balancing_signals)

## 3.2 Configuring Cloud CDN. Considerations include:

1. Enabling and disabling Cloud CDN
* [CDN with a bucket](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket)
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