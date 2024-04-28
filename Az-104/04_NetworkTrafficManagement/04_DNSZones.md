# DNS

DNS (Domain Name System) is a hierarchical decentralized naming system for computers, services, or any resource connected to the Internet or a private network. It translates easily memorized domain names to the numerical IP addresses needed for locating and identifying computer services and devices with the underlying network protocols. DNS servers maintain a distributed database of domain names and their associated IP addresses and provide name resolution services to clients requesting the translation of domain names to IP addresses.

In Azure, Azure DNS is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure. It allows you to host your DNS domains and manage DNS records using Azure's highly available and scalable infrastructure. Azure DNS supports hosting domains both in the public DNS and private DNS zones.

Here are the key components of Azure DNS:

1. **DNS Zone**: A DNS zone is a portion of the DNS namespace that is managed by Azure DNS. It is a container for hosting DNS records for a specific domain name, such as `example.com`. Azure DNS supports both public and private DNS zones.

2. **Recordset**: A recordset is a collection of DNS records that are managed together within a DNS zone. Each recordset represents a set of DNS records that have the same name, type, and DNS zone. For example, you might have an "A" recordset for your domain pointing to a specific IP address.

Azure DNS supports various types of DNS records, including:

- A (Address) Records: Maps a domain name to an IPv4 address.
- AAAA (IPv6 Address) Records: Maps a domain name to an IPv6 address.
- CNAME (Canonical Name) Records: Alias of one domain name to another.
- MX (Mail Exchange) Records: Specifies mail server responsible for accepting email messages on behalf of a domain.
- TXT (Text) Records: Contains arbitrary text used for various purposes, such as SPF (Sender Policy Framework) records for email authentication.

Using Azure DNS, you can manage these recordsets to control how your domain names are resolved on the internet or within your Azure network. It provides a reliable and scalable DNS hosting solution integrated with other Azure services.

To configure a DNS zone with Azure App Gateway and change the name servers in GoDaddy, you'll perform the following steps:

### Setting up DNS Zone with Azure App Gateway:

1. **Create a DNS Zone in Azure**:
   - Log in to the Azure portal.
   - Navigate to "DNS zones" under the "Networking" section.
   - Click on "Add" to create a new DNS zone.
   - Enter the domain name you want to manage, e.g., `example.com`, and configure other settings as needed.

2. **Add DNS Records**:
   - Inside your DNS zone in Azure, add DNS records as needed. You'll at least need an A record pointing to your App Gateway's public IP address.
   - Set the name (usually `@` for the root domain) and the IP address of your Application Gateway.

3. **Configure Application Gateway**:
   - Set up your Azure Application Gateway with your desired configuration, including listeners, backend pools, routing rules, etc.

### Changing Name Servers in GoDaddy:

1. **Retrieve Azure Name Servers**:
   - In the Azure portal, find the DNS zone you created earlier.
   - Note down the Azure-assigned name servers, usually in the format `ns1-xx.azure-dns.com` through `ns4-xx.azure-dns.com`.

2. **Log in to GoDaddy**:
   - Go to GoDaddy's website and log in to your account.

3. **Access Domain Settings**:
   - Navigate to the domain management section.
   - Find the domain you're configuring and look for the option to manage its DNS or name servers.

4. **Change Name Servers**:
   - Replace the existing GoDaddy name servers with the Azure name servers you noted down earlier.
   - Usually, this involves removing the existing GoDaddy name servers and adding the Azure name servers in their place.

5. **Save Changes**:
   - Save the changes in GoDaddy's dashboard.

### Verify Configuration:

1. **DNS Propagation**:
   - DNS changes may take some time to propagate across the internet. Be patient; it could take anywhere from a few minutes to several hours for the changes to take effect globally.

2. **Testing**:
   - Once DNS propagation is complete, you can test the setup by accessing your web application using the domain name. Ensure that traffic is routed correctly to your application via the Azure Application Gateway.

By following these steps, you'll configure a DNS zone with Azure App Gateway and delegate name servers in GoDaddy to point to Azure DNS.
