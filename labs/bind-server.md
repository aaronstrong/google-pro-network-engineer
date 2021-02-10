# [Creating a BIND Server](https://medium.com/faun/dns-forwarding-zone-forwarding-and-dns-policy-in-gcp-640a34b15bca)

The lab must be deployed in a single VPC. There are a few differnet target types when using [Cloud DNS Forwarding](https://cloud.google.com/dns/docs/zones#firewall-rules) that you need to be aware of.

This lab makes use of the Type 1 target type, but if you're deploying in a hybrid model with an authoritative name server on-premises, you'll want to deploy the Type 2 target type. [Use Google Cloud DNS Best Practice Guide.](https://cloud.google.com/dns/docs/best-practices#hybrid-architecture-using-single-shared-vpc-network)


## Step 1

Setup the network. Create 2 VPCs in the same projec. You will also need to create firewall rules to allow inbound access for DNS.

### Create VPC

```gcloud
ROUTING_MODE=global
SUBNET_MODE=custom
REGION=us-central1

# HUB
VPC_HUB=hub-vpc
SUBNET_HUB_NAME=hub-subnet-01
SUBNET_HUB_RANGE=192.168.0.0/24

gcloud compute networks create $VPC_HUB \
--bgp-routing-mode=$ROUTING_MODE \
--subnet-mode=$SUBNET_MODE

gcloud compute networks subnets create $SUBNET_HUB_NAME \
--network=$VPC_HUB \
--range=$SUBNET_HUB_RANGE \
--region=$REGION

# Spoke 1
VPC_SPOKE1=spoke1-vpc
SUBNET_SPOKE1_NAME=spoke1-subnet-01
SUBNET_SPOKE1_RANGE=192.168.1.0/24

gcloud compute networks create $VPC_SPOKE1 \
--bgp-routing-mode=$ROUTING_MODE \
--subnet-mode=$SUBNET_MODE

gcloud compute networks subnets create $SUBNET_SPOKE1_NAME \
--network=$VPC_SPOKE1 \
--range=$SUBNET_SPOKE1_RANGE \
--region=$REGION
```

### Create Firewall Rule
Fireall rule to allow DNS
> Note: `35.19.192.0/19` is the range coming from Cloud DNS
```gcloud
VPC_HUB=hub-vpc
gcloud compute firewall-rules create allow-dns \
--network $VPC_HUB \
--allow tcp:53,udp:53 \
--source-ranges 192.168.0.0/16,172.16.0.0/12,10.0.0.0/8,35.199.192.0/19
```

Firewall rule to allow SSH into our instances

```bash
# HUB
VPC_HUB=hub-vpc
gcloud compute firewall-rules create allow-ssh-$VPC_HUB \
--network $VPC_HUB \
--allow tcp:22 \
--source-ranges 0.0.0.0/0

# SPOKE1
VPC_SPOKE1=spoke1-vpc
gcloud compute firewall-rules create allow-ssh-$VPC_SPOKE1 \
--network $VPC_SPOKE1 \
--allow tcp:22 \
--source-ranges 0.0.0.0/0
```

## Step 2

Create the BIND server in the `HUB` vpc and then configure the BIND server. Next, deploy a client machine in the `Spoke 1` VPC.

### Create a BIND Server
> Note: I'm using instance types of `f1-micro` and `preemptible` to keep my costs low. This is a lab.
```gcloud
gcloud beta compute instances create my-bind-server \
--zone=us-central1-a \
--machine-type=n1-standard-1 \
--subnet=hub-subnet-01 \
--network-tier=STANDARD \
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
sudo apt-get install -y bind9 bind9utils bind9-doc dnsutils
cd /etc/bind
sudo cp db.local forward.mylab.com
sudo cp db.local reverse.mylab.com
EOF'
```

Create a client to use the BIND server.

```gcloud
gcloud compute instances create instance-client1 \
--machine-type=f1-micro \
--zone=us-central1-b \
--subnet=hub-subnet-01 \
--preemptible \
--metadata=startup-script='#! /bin/bash
sudo apt-get update
sudo apt-get install -y dnsutils
EOF'  \
--no-restart-on-failure \
--maintenance-policy=terminate \
--image-project=ubuntu-os-cloud \
--image=ubuntu-2004-focal-v20210119a \
--boot-disk-size=10GB \
--network-tier=standard
```

Create a third client in the `Spoke 1` VPC

```gcloud
gcloud compute instances create remote-instance1 \
--machine-type=f1-micro \
--zone=us-central1-b \
--subnet=spoke1-subnet-01 \
--preemptible \
--metadata=startup-script='#! /bin/bash
sudo apt-get update
sudo apt-get install -y dnsutils
EOF'  \
--no-restart-on-failure \
--maintenance-policy=terminate \
--image-project=ubuntu-os-cloud \
--image=ubuntu-2004-focal-v20210119a \
--boot-disk-size=10GB \
--network-tier=standard
```

### Update the BIND server config

SSH into the BIND server and and setup the DNS.

```bash
cd /etc/bind
sudo vim named.conf.local
```

```bash
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";
zone    "mylab.com" IN {
        type master;
        file    "/etc/bind/forward.mylab.com";
};

zone    "0.168.192.in-addr.arpa" IN {
        type master;
        file    "/etc/bind/reverse.mylab.com";
};
```

### Update the forwarder
```bash
sudo vim forward.mylab.com
```

> Update the values accordingly

```bash
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     primary.mylab.com. root.mylab.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      primary.mylab.com.
@       IN      A       127.0.0.1
@       IN      AAAA    ::1

primary IN      A       192.168.0.3
test    IN      A       192.168.0.3
```
### Update the reverse DNS file
```bash
sudo vim reverse.mylab.com
```
```bash
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     mylab.com. root.mylab.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      primary.mylab.com.
primary IN      A       192.168.0.3
40      IN      PTR     primary.mylab.com.
```

Restart the BIND server
```bash
sudo service bind9 restart
```
Check the status of the BIND service
```bash
sudo service bind9 status
```
It should say `(running)`

```vim
● bind9.service - BIND Domain Name Server
   Loaded: loaded (/lib/systemd/system/bind9.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2021-02-07 00:03:11 UTC; 7s ago
     Docs: man:named(8)
  Process: 3321 ExecStop=/usr/sbin/rndc stop (code=exited, status=0/SUCCESS)
 Main PID: 3324 (named)
    Tasks: 4 (limit: 4369)
   CGroup: /system.slice/bind9.service
           └─3324 /usr/sbin/named -f -u bind
```

Check the local DNS entry resolves
```bash
dig A test.mylab.com @localhost
```
Result needs to have an `Answer` result.

```bash
; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> A test.mylab.com @localhost
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39691
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: b736a78bc1ae8df3c288006e6016238bdba81a8fd87a9943 (good)
;; QUESTION SECTION:
;test.mylab.com.             IN      A
;; ANSWER SECTION:
test.mylab.com.      604800  IN      A       192.168.0.3
;; AUTHORITY SECTION:
mylab.com.           604800  IN      NS      primary.mylab.com.
;; ADDITIONAL SECTION:
primary.mylab.com.   604800  IN      A       192.168.0.3
;; Query time: 0 msec
;; SERVER: ::1#53(::1)
;; WHEN: Sun Jan 31 03:27:07 UTC 2021
;; MSG SIZE  rcvd: 128
```

## Create Cloud DNS [Zones](https://cloud.google.com/dns/docs/zones)


```gcloud
PROJECT_ID=<PROJECT_ID>
BIND_IP=192.168.0.3
gcloud config set project $PROJECT_ID
# Enable Cloud DNS API
gcloud services enable dns.googleapis.com

# Setup Cloud DNS
gcloud dns managed-zones create mylab-com \
--description=DESCRIPTION \
--dns-name=mylab.com \
--networks=hub-vpc \
--private-forwarding-targets=$BIND_IP \
--visibility=private
```

### Test Cloud DNS

Remote into the `instance-client1` instance in the same VPC as the BIND server. Because we configured VPC Forwarding in Cloud DNS, all client machines will forward their requests to the BIND server.

From the client machine
```bash
dig test.mylab.com
```
We can see we have an answer response with the correct IP plus it's coming from `127.0.0.53`.

```bash
$ dig test.mylab.com

; <<>> DiG 9.16.1-Ubuntu <<>> test.mylab.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63464
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;test.mylab.com.                        IN      A

;; ANSWER SECTION:
test.mylab.com.         6931    IN      A       192.168.0.3

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Sun Feb 07 00:23:24 UTC 2021
;; MSG SIZE  rcvd: 59
```

### Cloud DNS Peering

Create a Cloud DNS peer between the `Spoke 1` VPC and the `HUB` VPC.

```gcloud
gcloud dns managed-zones create dns-peer \
  --description="dns peering" \
  --dns-name=mylab.com \
  --networks=spoke1-vpc \
  --target-network=hub-vpc \
  --target-project=playground-s-11-0ca700be \
  --visibility=private
```
Test from the remote-nstance in the `Spoke 1` VPC.

```bash
@remote-instance1:$ dig test.mylab.com

; <<>> DiG 9.16.1-Ubuntu <<>> test.mylab.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 65314
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;test.mylab.com.                        IN      A

;; ANSWER SECTION:
test.mylab.com.         7036    IN      A       192.168.0.3

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Sun Feb 07 00:51:41 UTC 2021
;; MSG SIZE  rcvd: 59
```

## List inbound forwarder entry points

When an inbound server policy applies to a VPC network, Cloud DNS creates a set of regional [internal IP addresses](https://cloud.google.com/dns/docs/policies#list-in-entrypoints) that serve as destinations to which your on-premises systems or name resolvers can send DNS requests. These addresses serve as entry points to the name resolution order of your VPC network.

Each inbound forwarder accepts and receives queries from Cloud VPN tunnels or Cloud Interconnect attachments (VLANs) in the same region as the regional internal IP address.

```gcloud
gcloud compute addresses list \
--filter='purpose = "DNS_RESOLVER"' \
--format='csv(address, region, subnetwork)'
```

### Client to use DNS
Update a client machine to use the BIND DNS

```bash
cd /etc/resolv.conf
sudo vim /etc/resolv.conf

>nameserver 192.168.0.3
```

## Deploying Microsoft Active Directory with Cloud DNS

Found this great [article](https://cloud.google.com/solutions/deploying-microsoft-active-directory-domain-controllers-with-advanced-networking-configuration-on-gcp#launch_the_test_instance_us-central1) that talks about deploying Microsoft Active Directory Domain Controllers in GCP using Cloud DNS.


## Troubleshooting

I found this great [troubleshooting](https://cloud.google.com/dns/docs/troubleshooting) guide when I initially had problems getting the lab configured.

My favorite command to trace incoming packets:
```bash
sudo tcpdump port 53 and udp -vv
```