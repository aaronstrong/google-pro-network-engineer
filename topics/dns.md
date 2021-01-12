# DNS

## [Overview](https://cloud.google.com/dns/docs/overview)
1. Managed
1. DNS Security
1. Migration
1. DNS Forwarding

## Managed DNS Zones
There are 3 types of zones for name resolutions.
1. [Internal DNS](https://cloud.google.com/compute/docs/internal-dns)

    Provides host name resolutions inside of a VPC that is managed by Google from the metadata server 169.254.169.254. These host name records cannot be turned off.

    The format for a hostname is `[INSTANCE_NAME].[ZONE].c.[PROJECT_ID].internal`.

1. [Private Zone](https://cloud.google.com/dns/docs/best-practices#best_practices_for_private_zones)

    Private zones host DNS records that are visible only inside your organization. These records have a higher priority than Internal DNS records.

    ![](https://cloud.google.com/dns/images/private_zones_hosted_in_shared_vpc_network.svg)

    * [Use Forwarding Zones to query on-premises servers](https://cloud.google.com/dns/docs/best-practices#use_forwarding_zones_to_query_on-premises_servers)
    * [Use DNS server policies to allow queries from on-premises](https://cloud.google.com/dns/docs/best-practices#use_dns_server_policies_to_allow_queries_from_on-premises)
    * [Open Google Cloud and on-premises firewalls to allow DNS Traffic](https://cloud.google.com/dns/docs/best-practices#open-google-cloud-and-on-premises-firewalls)

1. Public Zone

    A public zone is visiblie to the internet. Usually purchased and registered through a registrar like Godaddy.

    You must purchase a domain name from a registrar like Godaddy. Then update the registrar's name servers to point to the name servers in Google Cloud DNS. The registrar will then forward requests.

    ![](./registrar-to-clouddns.png)

    <b>[Migrating Public Zones](https://cloud.google.com/dns/docs/migrating)</b>

    Cloud DNS supports the migration of an existing DNS domain from another DNS provider to Cloud DNS.

    1. Create a managed zone in Cloud DNS that will contain your DNS records. When you create a zone, the new zone isn't used until you update your registration.
    2. Export your DNS config from your provider. This can be either in `BIND` or in `YAML` format.
    > :start: Good testing material.
    3. Import the record set
    ```gcloud
    gcloud dns record-sets import -z=EXAMPLE_ZONE_NAME --zone-file-format path-to-file
    ```
    4. Update registrar's name server records. Sign into the registrar provider and change the authoritative name servers to point to the name servers.
    5. Wait for the changes and verify

    <b>Resolution Order</b>
    
    1. Cloud Dns
    1. Private zones
    1. DNS policy forwarding rule
    1. Private DNS forwarding zones
    1. Private and peer zones
    1. Compute engine internal DNS
    1. Queries public zones


## DNS Forwarding and Peering

## Cloud CDN

## Cache Keys and Cache Invalidation

## Signed URLs