# Create a Squid Proxy Server

[Google write up](https://cloud.google.com/vpc/docs/special-configurations)


## Single Instance
```gcloud
gcloud beta compute instances create my-proxy-server \
--zone=us-central1-a \
--machine-type=n1-standard-1 \
--subnet=hub-subnet-01 \
--network-tier=STANDARD \
--maintenance-policy=MIGRATE \
--scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append \
--image=ubuntu-1804-bionic-v20210129 \
--image-project=ubuntu-os-cloud \
--boot-disk-size=10GB \
--boot-disk-type=pd-standard \
--boot-disk-device-name=my-bind-server \
--no-shielded-secure-boot \
--shielded-vtpm \
--shielded-integrity-monitoring \
--reservation-affinity=any \
--preemptible \
--metadata=startup-script='#! /bin/bash
sudo apt-get update
sudo apt-get install -y squid
sudo sed -i 's:#\(http_access allow localnet\):\1:' /etc/squid/squid.conf
sudo sed -i 's:#\(http_access deny to_localhost\):\1:' /etc/squid/squid.conf
sudo sed -i 's:#\(acl localnet src 10.0.0.0/8.*\):\1:' /etc/squid/squid.conf
sudo sed -i 's:#\(acl localnet src 172.16.0.0/12.*\):\1:' /etc/squid/squid.conf
sudo sed -i 's:#\(acl localnet src 192.168.0.0/16.*\):\1:' /etc/squid/squid.conf
sudo service squid start
EOF'
```

### Firewall Rule to allow Squid Port

Need to open up the firewall port to allow ingress requests to hit the firewall port

```gcloud
gcloud compute firewall-rules create allow-squid-ingress-hub \
--network hub-vpc \
--allow tcp:3128 \
--source-ranges 192.168.0.0/16
```

### Client Setup
> Note: Update the IP address. Update to the instance IP.

```bash
export http_proxy=http://192.168.0.2:3128
export https_proxy=https://192.168.0.2:3128
curl www.google.com
```
### Configure Firewall Rules

```bash
gcloud compute firewall-rules create fw-allow-health-check \
    --network=hub-vpc \
    --action=allow \
    --direction=ingress \
    --target-tags=allow-health-check \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --rules=tcp,udp,icmp
```

### Create Unmanaged Instance Groups
```bash
gcloud compute instance-groups unmanaged create ig-a \
--zone=us-central1-a

gcloud compute instance-groups unmanaged create ig-c \
--zone=us-central1-b
```
### Add the VMs to the Instance Groups
```gcloud
gcloud compute instance-groups unmanaged add-instances ig-a \
--zone=us-central1-a \
--instances=gateway-1

gcloud compute instance-groups unmanaged add-instances ig-c \
--zone=us-central1-b \
--instances=gateway-2
```

### [Configure Internal Load Balancer](https://cloud.google.com/load-balancing/docs/internal/setting-up-internal)

Create a new regional HTTP health check to test HTTP connectivity to the VMs on 3128.

```gcloud
gcloud compute health-checks create tcp hc-tcp-3128 \
--region=us-central1 \
--port=3128
```

Create the backend service for HTTP traffic

```gcloud
gcloud compute backend-services create be-ilb \
--load-balancing-scheme=internal \
--protocol=tcp \
--region=us-central1 \
--health-checks=hc-tcp-3128 \
--health-checks-region=us-central1
```

Add the instance groups to the backend service

```gcloud
gcloud compute backend-services add-backend be-ilb \
--region=us-central1 \
--instance-group=ig-a \
--instance-group-zone=us-central1-a
gcloud compute backend-services add-backend be-ilb \
--region=us-central1 \
--instance-group=ig-c \
--instance-group-zone=us-central1-b
```

Create a forwarding rule for the backend service.

```gcloud
gcloud compute forwarding-rules create fr-ilb \
--region=us-central1 \
--load-balancing-scheme=internal \
--network=hub-vpc \
--subnet=hub-subnet-01 \
--ip-protocol=TCP \
--ports=80,3128,8080 \
--backend-service=be-ilb \
--backend-service-region=us-central1
```

### Test Load Balancer
> Note: Update the IP address. Update to the front end IP.

```bash
export http_proxy=http://192.168.0.8:3128
export https_proxy=https://192.168.0.8:3128
curl www.google.com
```