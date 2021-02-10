# GCP Lab to create an HA-VPN

First create the VPCs. In this lab we're creating a hub-and-spoke like design.
### Hub and Spoke

```bash
VPC_HUB=hub-vpc
SUBNET_HUB_NAME=hub-subnet-01
SUBNET_HUB_RANGE=192.168.0.0/24

VPC_SPOKE1=spoke1-vpc
SUBNET_SPOKE1_NAME=spoke1-subnet-01
SUBNET_SPOKE1_RANGE=192.168.1.0/24

VPC_SPOKE2=spoke2-vpc
SUBNET_SPOKE2_NAME=spoke2-subnet-01
SUBNET_SPOKE2_RANGE=192.168.2.0/24

ROUTING_MODE=global
SUBNET_MODE=custom
REGION=us-central1

# HUB
gcloud compute networks create $VPC_HUB \
--bgp-routing-mode=$ROUTING_MODE \
--subnet-mode=$SUBNET_MODE

gcloud compute networks subnets create $SUBNET_HUB_NAME \
--network=$VPC_HUB \
--range=$SUBNET_HUB_RANGE \
--region=$REGION

# SPOKE 1
gcloud compute networks create $VPC_SPOKE1 \
--bgp-routing-mode=$ROUTING_MODE \
--subnet-mode=$SUBNET_MODE

gcloud compute networks subnets create $SUBNET_SPOKE1_NAME \
--network=$VPC_SPOKE1 \
--range=$SUBNET_SPOKE1_RANGE \
--region=$REGION

# SPOKE 2
gcloud compute networks create $VPC_SPOKE2 \
--bgp-routing-mode=$ROUTING_MODE \
--subnet-mode=$SUBNET_MODE

gcloud compute networks subnets create $SUBNET_SPOKE2_NAME \
--network=$VPC_SPOKE2 \
--range=$SUBNET_SPOKE2_RANGE \
--region=$REGION
```


## Create firewall rules to allow communications

Create the firewall rules to allow for SSH access and to allow ICMP between our VPCs.

```bash
# HUB
gcloud compute firewall-rules create allow-ssh-ingress-hub \
--network hub-vpc \
--allow tcp:22,icmp \
--source-ranges 192.168.0.0/16,0.0.0.0/0

# SPOKE1
gcloud compute firewall-rules create allow-ssh-ingress-spoke1 \
--network spoke1-vpc \
--allow tcp:22,icmp \
--source-ranges 192.168.0.0/16,0.0.0.0/0

# SPOKE2
gcloud compute firewall-rules create allow-ssh-ingress-spoke2 \
--network spoke2-vpc \
--allow tcp:22,icmp \
--source-ranges 192.168.0.0/16,0.0.0.0/0
```

# Create VMs

Need something to ping against, so create two instances. One in the HUB VPC, and another in our spoke VPC.

```gcloud
gcloud compute instances create instance-hub \
--machine-type=f1-micro \
--zone=us-central1-b \
--subnet=hub-subnet-01 \
--preemptible \
--no-restart-on-failure \
--maintenance-policy=terminate \
--image-project=ubuntu-os-cloud \
--image=ubuntu-2004-focal-v20210119a \
--boot-disk-size=10GB \
--network-tier=standard

gcloud compute instances create instance-spoke2 \
--machine-type=f1-micro \
--zone=us-central1-b \
--subnet=spoke2-subnet-01 \
--preemptible \
--no-restart-on-failure \
--maintenance-policy=terminate \
--image-project=ubuntu-os-cloud \
--image=ubuntu-2004-focal-v20210119a \
--boot-disk-size=10GB \
--network-tier=standard

```

## Create HA VPN
Now create our HA VPN. You can use the below link to create it if you want.

[Main Link](https://cloud.google.com/network-connectivity/docs/vpn/how-to/creating-ha-vpn2)

```bash
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

## Cleanup

```gcloud
gcloud compute vpn-tunnels delete spoke-tunnel-if0 --region=us-central1

gcloud compute vpn-tunnels delete hub-tunnel-if0 --region=us-central1

gcloud compute routers delete cr-2 --region=us-central1

gcloud compute routers delete cr-1 --region=us-central1

gcloud compute instances delete instance-spoke2 --zone=us-central1-b

gcloud compute instances delete instance-1 --zone=us-central1-b

gcloud compute instances delete instance-2 --zone=us-central1-b
```