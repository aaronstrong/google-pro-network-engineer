# Labs

## VPC

Create a custom VPC with global mode
```gcloud
gcloud compute networks create custom-vpc \--bgp-routing-mode=global \
--subnet-mode=custom
```
---

Create a subnetwork

```gcloud
gcloud compute networks subnets create subnet-01 \
--network=custom-vpc \
--range=192.168.0.0\24 \
--region=us-central1 \
--secondary-range pods=10.0.0.0/16,services=10.17.0.0/20
```

---
Create a firewall rule
```gcloud
gcloud compute firewall-rules create allow-ssh-ingress \
--network custom-vpc \
--allow tcp:22,icmp \
--source-ranges 0.0.0.0/0
```
---
Create an instance with ephemeral public ip
```gcloud
gcloud compute instances create instance-1 \
--machine-type=g1-small \
--zone=us-central1-b \
--subnet=subnet-01
--preemptible --no-restart-on-failure \
--maintenance-policy=terminate \
--image-project=ubuntu-os-cloud \
--image=ubuntu-2004-focal-v20210119a \
--boot-disk-size=10GB \
--subnet=subnet-01 \
--network-tier=standard
```
---
Create a private instance
```gcloud
gcloud compute instances create private-instance-1 \
--machine-type=g1-small \
--zone=us-central1-b \
--subnet=subnet-01 \
--no-address \
--preemptible --no-restart-on-failure \
--maintenance-policy=terminate \
--image-project=ubuntu-os-cloud \
--image=ubuntu-2004-focal-v20210119a \
--boot-disk-size=10GB \
--subnet=subnet-01 \
--network-tier=standard
```
---
Compute Instance with [OS Login](https://cloud.google.com/compute/docs/instances/managing-instance-access#gcloud)
```gcloud
gcloud compute instances create oslogin-instance-1 \
--machine-type=g1-small \
--zone=us-central1-b \
--subnet=subnet-01 \
--preemptible --no-restart-on-failure \
--maintenance-policy=terminate \
--image-project=ubuntu-os-cloud \
--image=ubuntu-2004-focal-v20210119a \
--boot-disk-size=10GB \
--subnet=subnet-01 \
--network-tier=standard \
--metadata enable-oslogin=TRUE
```
---
### Hub and Spoke

```gcloud
gcloud compute networks create hub-vpc \
--bgp-routing-mode=global \
--subnet-mode=custom

gcloud compute networks subnets create hub-subnet-01 \
--network=hub-vpc \
--range=192.168.0.0/24 \
--region=us-central1

gcloud compute networks create spoke1-vpc \
--bgp-routing-mode=global \
--subnet-mode=custom

gcloud compute networks subnets create spoke1-subnet-01 \
--network=spoke1-vpc \
--range=192.168.1.0/24 \
--region=us-central1

gcloud compute networks create spoke2-vpc \
--bgp-routing-mode=global \
--subnet-mode=custom

gcloud compute networks subnets create spoke2-subnet-01 \
--network=spoke2-vpc \
--range=192.168.2.0/24 \
--region=us-central1
```

Create a VPC peer between hub and spoke1 VPC
```gcloud
gcloud compute networks peerings create hub-spoke1-peer --network=hub-vpc --peer-network=spoke1-vpc --export-custom-routes

gcloud compute networks peerings create spoke1-hub-peer --network=spoke1-vpc --peer-network=hub-vpc --export-custom-routes
```

Create firewall rules to allow communications
```gcloud
gcloud compute firewall-rules create allow-ssh-ingress-hub \
--network hub-vpc \
--allow tcp:22,icmp \
--source-ranges 192.168.1.0/24,0.0.0.0/0

gcloud compute firewall-rules create allow-ssh-ingress-spoke1 \
--network spoke1-vpc \
--allow tcp:22,icmp \
--source-ranges 192.168.0.0/24,192.168.2.0/24,0.0.0.0/0

gcloud compute firewall-rules create allow-ssh-ingress-spoke2 \
--network spoke2-vpc \
--allow tcp:22,icmp \
--source-ranges 192.168.0.0/24,192.168.1.0/24,0.0.0.0/0

```

Create VMs
```gcloud
gcloud compute instances create instance-hub \
--machine-type=f1-micro \
--zone=us-central1-b \
--subnet=hub-subnet-01 \
--preemptible --no-restart-on-failure \
--maintenance-policy=terminate \
--image-project=ubuntu-os-cloud \
--image=ubuntu-2004-focal-v20210119a \
--boot-disk-size=10GB \
--network-tier=standard

gcloud compute instances create instance-spoke2 \
--machine-type=f1-micro \
--zone=us-central1-b \
--subnet=spoke2-subnet-01 \
--preemptible --no-restart-on-failure \
--maintenance-policy=terminate \
--image-project=ubuntu-os-cloud \
--image=ubuntu-2004-focal-v20210119a \
--boot-disk-size=10GB \
--network-tier=standard

```

Create HA VPN Gateway
```gcloud
gcloud compute vpn-gateways create hub-gateway \
--network hub-vpc \
--region us-central1

gcloud compute vpn-gateways create spoke2-gateway \
--network spoke2-vpc \
--region us-central1
```

Create Cloud Router
```gcloud
gcloud compute routers create cr-1 \
--region us-central1 \
--network hub-vpc \
--asn 64512

gcloud compute routers create cr-2 \
--region us-central1 \
--network spoke2-vpc \
--asn 64513
```

Create VPN Tunnels, first on the hub
```gcloud
gcloud compute vpn-tunnels create hub-tunnel1\
--peer-gcp-gateway spoke2-gateway \
--region us-central1 \
--ike-version 1 \
--shared-secret mysharedSecret! \
--router cr-1 \
--vpn-gateway hub-gateway \
--interface 0

gcloud compute vpn-tunnels create hub-tunnel2 \
--peer-gcp-gateway spoke2-gateway \
--region us-central1 \
--ike-version 1 \
--shared-secret mysharedSecret! \
--router cr-1 \
--vpn-gateway hub-gateway \
--interface 1
```

Create tunnels on spoke
```gcloud
gcloud compute vpn-tunnels create spoke-tunnel1 \
--peer-gcp-gateway hub-gateway \
--region us-central1 \
--ike-version 1 \
--shared-secret mysharedSecret! \
--router cr-2 \
--vpn-gateway spoke2-gateway \
--interface 0

gcloud compute vpn-tunnels create spoke-tunnel2 \
--peer-gcp-gateway hub-gateway \
--region us-central1 \
--ike-version 1 \
--shared-secret mysharedSecret! \
--router cr-2 \
--vpn-gateway spoke2-gateway \
--interface 1
```
BGP interface
```gcloud
gcloud compute routers add-interface cr-1 \
   --interface-name cr1-int-0 \
   --ip-address 169.254.0.1 \
   --mask-length 30 \
   --vpn-tunnel hub-tunnel1 \
   --region us-central1

gcloud compute routers add-bgp-peer cr-1 \
   --peer-name spoke1-peer \
   --interface cr1-int-0 \
   --peer-ip-address 169.254.0.2 \
   --peer-asn 64513 \
   --region us-central1
```


## Create HA VPN
[Main Link](https://cloud.google.com/network-connectivity/docs/vpn/how-to/creating-ha-vpn2)

```gcloud
gcloud compute vpn-gateways create hub-gateway1 \
--network hub-vpc \
--region us-central1

gcloud compute vpn-gateways create spoke2-gateway1 \
--network spoke2-vpc \
--region us-central1

gcloud compute routers create cr-1 \
--region us-central1 \
--network hub-vpc \
--asn 65001

gcloud compute routers create cr-2 \
--region us-central1 \
--network spoke2-vpc \
--asn 65002

gcloud compute vpn-tunnels create hub-tunnel-if0 \
--peer-gcp-gateway spoke2-gateway1 \
--region us-central1 \
--ike-version 1 \
--shared-secret SHARED_SECRET! \
--router cr-1 \
--vpn-gateway hub-gateway1 \
--interface 0

gcloud compute routers add-interface cr-1 \
   --interface-name bgp-1 \
   --ip-address 169.254.0.1 \
   --mask-length 30 \
   --vpn-tunnel hub-tunnel-if0 \
   --region us-central1

gcloud compute routers add-bgp-peer cr-1 \
   --peer-name spoke2-peer \
   --interface bgp-1 \
   --peer-ip-address 169.254.0.2 \
   --peer-asn 65002 \
   --region us-central1

gcloud compute vpn-tunnels create spoke-tunnel-if0 \
--peer-gcp-gateway hub-gateway1 \
--region us-central1 \
--ike-version 1 \
--shared-secret SHARED_SECRET! \
--router cr-2 \
--vpn-gateway spoke2-gateway1 \
--interface 0

gcloud compute routers add-interface cr-2 \
   --interface-name bgp-1 \
   --ip-address 169.254.0.2 \
   --mask-length 30 \
   --vpn-tunnel spoke-tunnel-if0 \
   --region us-central1

gcloud compute routers add-bgp-peer cr-2 \
   --peer-name hub-peer-1 \
   --interface bgp-1 \
   --peer-ip-address 169.254.0.1 \
   --peer-asn 65001 \
   --region us-central1

```


## DELETE

```gcloud
gcloud compute vpn-tunnels delete spoke-tunnel-if0 --region=us-central1

gcloud compute vpn-tunnels delete hub-tunnel-if0 --region=us-central1

gcloud compute routers delete cr-2 --region=us-central1

gcloud compute routers delete cr-1 --region=us-central1

gcloud compute instances delete instance-spoke2 --zone=us-central1-b

gcloud compute instances delete instance-1 --zone=us-central1-b

gcloud compute instances delete instance-2 --zone=us-central1-b
```