## Getting Started with OCI

3 Different Regions Oracle have: Commercial, Government and Microsoft Azure Interconnect
Each region has 3 different Availability Domains in one Data Center.

Every AD has at least 3 fault domain, a fault domain is a failura isolation boundary within an availability domain. This 3 number is well-suited for hosting quorum-based replicated storage systems and consensus algorithms which are some of the basic primitives or fault tolerant systems.

Off-box Network Virtualization: As the name implies, all the network virtualization has been put out into the network

### Differentiation

| Technical     |
| ------------- |
|**Performance:** Off-box Network Virtualization |
|**Performance:** Bare Metal + Local NVMe storage |
|**Performance:** All SSD Storage |
|**Performance:** No Network, CPU or Memory over-subscription |
|Battle tested (NetSuite and othe SaaS apps run on OCI) |
|DB Options - BM, VM, Exadata, RAC |
|Enterprise Apps support (EBS, JDE..) |

| Business     |
| ------------- |
|Aggressive and predictable pricing - cheaper than AWS |
|Industry's unique SLAs on Performance, Management and Availability |
|BYOL and Universal Cloud Credits |
|Support through one org |

## IAM

![pic1](/images/pic1.png)

### Policy Syntax

Allow <**subject**> to <**verb**> <**resource-type**> in <**location**> where <**conditions**>

| Verb    | Type of access |
| ------------- |---------- |
|**inspect**| Ability to list resources|
|**read**| Inspect + ability to get user-specified metadata/actual resource|
|**use**| read + ability to work with existing resources (the actions vary by resource type)|
|**manage**| Includes all permissions for the resource|


| Aggreaget resource-type| Individual resource type |
| ------------- | ---|
|**all-resources**|
|**database-family**| db-systems, db-nodes, db-homes, databases |
|**instance-family**|instances, instance-images, volume-attachments, console-histories |
|**object-family**| buckets, objects |
|**virtual-network-family**| vcn, subnet, route-tables, security-lists, dhcp-options, and many more resources |
|**volume-family**| volumes, volume-attachments, volume-backups |
|**Cluster-family**| clusters, cluster-node-pool, cluster-work-requests |
|**File-family**| file-systems, mount-targets, export-sets |
|**dns**| dns-zones, dns-records, dns-traffic, ... |

#### Verb + Permissions +  API Operation

![pic2](/images/pic2.png)


## Compartment

## Policy Inheritance and Attachment for Compartments, Read !!!

Three levels of compartments: A, B and C
* Policies that apply to resources in Compartment A and also apply to resources in Compartments B and C
* Allow group NetworkAdmins to manage virtual-network-family in compartment A allows the group NetworkAdmins to manage VCNs in Compartment A, B and C

## IAM-Tags

# Networking - VCN, FastConnect, Load Balancer

## CIDR

![pic3](/images/pic3.png)

192.168.1.0/24 would equate to IP range: 192.168.1.0 - 192.168.1.255

|128|64|32|16|8|4|2|1|
| - | -| -| -|-|-|-|-|
|2<sup>7</sup>|2<sup>6</sup>|2<sup>5</sup>|2<sup>4</sup>|2<sup>3</sup>|2<sup>2</sup>|2<sup>1</sup>|2<sup>0</sup>|

192 is represented as **1 1 0 0 0 0 0 0**

**192 =** 2<sup>7</sup> + 2<sup>6</sup>
**168 =**  2<sup>7</sup> + 2<sup>5</sup> + 2<sup>3</sup>
**1 =** 2<sup>0</sup>

Class A: /8
Class B. /16
Class C: /24

/27 hiç birisine dahil değildir. Yukarıda /27'de 3 birim kırmızı işaretlenmiştir. Bunun anlamı bu 3 bit'in subnetwork tarafından ödünç alındığın gösterir. Ve bu şekilde subnet mantığı kullanılabilir.

224'de, 128 + 64 + 32 denkeminden gelmektedir.

Bir kere oluştuktan sonra primary CIDR (Classes Inter-Domain Routing) değiştirilemez. Full block range yani IP address length 32’dir. Bit tanımı max 28, min ise 16 olabilir.

**Örnek:** 10.0.0.0/28 tanımlarsak, bunun anlamı 28’i subnetler ve available instance’lar tarafından alınır. Geriye kalan 4 bit anlamı da 24 yani 16 ip adresinin assign olduğudur.

Örneğimizi 28 değil de 16 olarak düşünürsek, bu durumda yarısı network ve diğer yarısı network içinde bulunan instance’lar için olduğu anlamını taşır.

28 yanlış bir seçim olacaktır. Bu seçim sadece account websitesi için kullanılacaksa olabilir. Ama bu da sonradan değiştirilemeyeceğinden, ilerleyen zamanlarda soruna neden olabilir.

Sonradan değiştirmenin tek yolu, yeni bir VCN oluşturmak ve migrate etmektir.
24 bit tanım olduğunu düşünelim. 32-24=8. 

2<sup>8</sup> = 256 ip kalacaktır.

10.0.0.0 incelersek, burada 10.0.0.4’den 10.0.0.254’e kadar toplam 256-5=251 ip verilebilir. Aşağıdaki ip’le atanamazlar. Default olarak aşağıdaki şekilde tahsis edileceklerdir.

**10.0.0.0** Base Network
**10.0.0.1** VCN router
**10.0.0.2** DNS related
**10.0.0.3** Reserved for future use
**10.0.0.255** Last IP

**According to the Oracle doc, the first two (.0,.1,.2-actually 3) and the last IP addresses are reserved.**

VCN içerisindeki subnet’ler basic TCP/IP kuralı gereği, overlap (çakışma durumu) olamazlar


## Intro VCN (Virtual Cloud Network)

**10.0.0.0/16 :** Recommended RFC 1918 Range

* Supported OCI VCN size range is from /16 to /30. /8 does not supported.
* Subnet D is a regional subnet

![pic4](/images/pic4.png)

* Subnets can be designated as either
      * Private (instances contain private IP addresses assigned to VNICs)
      * Public (contain both private and public IP addresses assigned to VNICs)
* VNIC is a component that enables a compute instance to connect to a VCN. The VNIC determines how the instance connects with endpoints inside and outside the VCN.

## IP Addresses

**VNIC:** Virtual Network Interface Cards

* Each instance in a subnet has at least one primary private IP addresses
* Instances >= 2 VNICs (additional VNICS called secondary VNICs)
* Each VNIC has one primary private IP; can have additional private IPs called secondary private IPs
* Up to 31 secondary additional private IPs

![pic5](/images/pic5.png)

* When a secondary VNIC is added, new Ethernet device is added and is recognized by the instance OS
      * VM1 - single VNIC instance
      * VM2 - connected to two VNICs from two subnets within the same VCN. Used for **virtual appliance scenarios**
      * VM3 - connected to two VNICs from two subnets from separate VCNs. Used to connect instances to a separate management network for isolated access

### Public IP

![pic6](/images/pic6.png)

**Public IP types:** Ephemeral and Reserved
* **Ephemeral:** temporary and existing for the lifetime of the instance
* **Reserved:** Persistent and existing beyond the lifetime of the instance it's assigned to (can be
unassigned and then reassigned to another instance)
* Ephemeral IP can be assigned to primary private IP only (hence, only 1 per VNIC v/s a max 32 for Reserved IP)

* No charge for using Public IP, including when the Reserved public IP addresses are unassociated

- Public IP assigned to
      - Instance (not recommended in most cases)

- **Oracle provided;** cannot choose/edit, but can view
      - OCI Public Load Balancer, NAT Gateway, DRG - IPSec tunnels, OKE master/worker

- Oracle provided; cannot choose/edit/view
      - Internet Gateway, Autonomous Database


## Routing and Gateways

![pic7](/images/pic7.png)

* You can have only one internet gateway for a VCN.
* After creating an internet gateway, youmust add a route for the gateway in the VCN's Route Table to enable traffic flow

## Route Table

![pic8](/images/pic8.png)

* Route Table is contains rules about how IP packets can travel to different IP addresses out of the VCN.

- Consists of a set of route rules; each rule specifies
      - Destination CIDR block
      - Route Target (the next hop) for the traffic that matches that CIDR

* Destination CIDR 0.0.0.0/0 means all traffic destined for Internet Gateway.

* Each subnet uses a single route table specified at time of subnet creation, but can be edited later.
* The route table is used for direction outside the VCN so that means route table is used only if the destination IP is not within the VCN's CIDR block.
* No route rules are required is order to enbale traffic within the VCN itself. So the following rule is **meanless**

|Destination CIDR | Route Target |
|-----------------|--------------|
|10.0.0.0/16      | local        |

* When you add an internet gateway, NAT gateway, service gateway, dynamic routing gateway or a peering connection, you must update the route table for any subnet that uses these gateways or connections.

### Internet Gateway

* You can have only one internet gateway for a VCN
* After creating an internet gateway, you add a rule in the VCN's route table which says that packets to **0.0.0.0/0 - all** addresses need to got to the internet

### NAT(Network Address Translation) Gateway

![pic10](/images/pic10.png)

* The following rule is means, send all traffic to the **NAT Gateway**

|Destination CIDR | Route Target |
|-----------------|--------------|
|0.0.0.0/16       | NAT Gateway  |

* NAT bir ağda bulunan bilgisayarın, kendi ağı dışında başka bir ağa veya İnternete çıkarken farklı bir IP adresi kullanabilmesi için geliştirilmiş bir İnternet protokolüdür. 

* NAT gateway gives an entire private network access to the internet without assigning each host a public IP address.

* Hosts can initiate outbound connections to the internet and receive responses, but not receive inbound connections initiated from the internet. Use case: updates, patches)

* From VCN to the Internet allowed but From the internet to the VCN is not allowed

* You can have more than one NAT gateway on a VCN, though a given subnet can route traffic to only a single NAT gateway.

![pic9](/images/pic9.png)

### Service Gateway

![pic11](/images/pic11.png)

Any traffic from VCN that is destined for one of the supported OCI (Oracle Cloud Infrastructure) public services uses the instance's private IP address for routing, travels over OCI network fabric, and never traverses the internet. 

* **Use case:** Back up DB Systems in VCN to Object Storage

Service CIDR labels represent all the public CIDRs for a given Oracle service or a group of Oracle services. E.g.

* If you're going to do object storage, you could specify object storage as a regional service.

- You could specify OCI region object storage or you could specify all services. So in this case, in future, if you have other services you want to access, you could actually do that because you have access to all the OCI services through the link to your service gateway.
      - OCI **\<region>** Object Storage 
      - All **\<region>** Services

## Dynamic Routing Gateway

![pic12](/images/pic12.png)

* A virtual router that provides a path for private traffic between VCN and destinations other than the internet like On-Premise Data Center.

* DRG is a standalone object. It must be attached to a VCN. VCN and DRG have a one-to-one relationship. That means single VCN can only have one DRG and one DRG can be attached to a single VCN at a time.

|Scenario| Solution |
|-----------------|--------------|
|Let instances connect to the internet and receive connectşons from it | Internet Gateway  |
|Let instances reach the internet without receiving connections from it | NAT Gateway  |
|Let VCN hosts privately connect to object storage, bypassing the internet | Service Gateway  |
|Make an OCI extend an on-premise network, with easy connectivity in both directions | Dynamic Routing Gateway  |

## Peering

### Local Peering Gateway (within region)

![pic13](/images/pic13.png)

* VCN peering is the process of connecting multiple VCNs in same region.
* Local VCN peering is the process of connecting two VCNs in the same region so that their resources can communicate using private IP addresses.
- Use cases;
      - You can have a management VCN which talks to multiple VCNs.
      - You could have bastion host talking with multiple VCNs.
* A local peering gateway (LPG) is a component on a VCN for routing traffic to a locally peered VCN.
* The two VCNs in the peering relationship **shouldn’t** have overlapping CIDRs.
* **Transitive routing** is not supported.

### Remote Peering Gateway (across region)

![pic14](/images/pic14.png)

* Remote VCN peering is the process of connecting two VCNs in different regions so that their resources can communicate using private IP addresses.
* Requires a remote peering connection (RPC) to be created on the DRGs (Dynamic Routing Gateway). RPC's job is to act as a connection point for a remotely peered VCN.
* The two VCNs in the peering relationship must not have overlapping CIDRs
* **Transitive routing** is not supported.
- Use cases;
      - Disaster recovery

|Scenario| Solution |
|-----------------|--------------|
|Let instances connect to the internet and receive connectşons from it | Internet Gateway  |
|Let instances reach the internet without receiving connections from it | NAT Gateway  |
|Let VCN hosts privately connect to object storage, bypassing the internet | Service Gateway  |
|Make an OCI extend an on-premise network, with easy connectivity in both directions | Dynamic Routing Gateway  |
|Privately connect two VCNs in a region |Local Peering Gateway  |
|Privately connect two VCNs in different regions |Remote Peering Gateway |

## Security VCN

### Security List (SL)

![pic15](/images/pic15.png)

- A common set of firewall rules associated with a subnet and applied to all instances launched inside the subnet.
      - Security list consists of rules that specify the types of traffic allowed in and out of the subnet
      - To use a given security list with a particular subnet, you associate the security list with the subnet either during subnet creation or later.
      - Security list apply to a given instance whether it's talking with another instance in the VCN or a host outside the VCN.
      - You can choose whether a given rule is stateful or stateless

### Network Security Group (NSG)

![pic16](/images/pic16.png)

- A network security group (NSG) provides a virtual firewall for a set of cloud resources that all have the same security posture.
      - NSG consists of set of rules that apply only to a set of VNICs of your choice in a single VCN
      - Currently, compute instances, load balancers and DB instances support NSG
      - When writing rules for an NSG, you can specify an NSG as the source or destination. Contrast this with SL rules, where you specify a CIDR as the source or destination
      - Oracle recommends using NSGs instead of SLs because NSGs let you separate the VCN's subnet architecture from your application security requirements

#### SL + NSG

![pic17](/images/pic17.png)

- We can use security lists alone, network security groups alone, or both together
- **If you have security rules that you want to enforce for all VNICs in a VCN:** the easiest solution is to put the rules in one security list, and then associate that security list with all subnets in the VCN
- **If you choose to use both SLs and NSGs, the set of rules that applies to a given VNIC is the union of these items:**
      - The security rules in the SLs associated with the VNIC's subnet
      - The security rules in all NSGs that the VNIC is in
      - A packet in question is allowed if any rule in any of the relevant lists and groups allows the traffic.

#### Stateful Security Rules

![pic18](/images/pic18.png)

* Default Security List rules are stateful
* Connection Tracking: when an instance receives traffic matching the stateful ingress rule, the response is tracked and automatically allowedregardless of any egress rules; similarly for sending traffic from the host

* Host B'den, Instance A'ya 80 portu üzerinden bir trafik başlamış olsun. Rules stateful olduğundan, Instance A'da bir tanım yapmaya gerek olmadan, Host B'ye responce mümkün olacaktır.

#### Stateless Security Rules

![pic19](/images/pic19.png)

* With stateless rules, response traffic is not automatically allowed
* To allow the response traffic for a stateless ingress rule, you must create a corresponding stateless egress rule
* If you add a stateless rule to a security list, that indicates that you do NOT want to use connection tracking for any traffic that matches that rule 
* Stateless rules are better for scenarios with large numbers of connections (Load Balancing, Big Data)

#### Default VCN components

VCN automatically comes with some default components
- Default Route Table
- Default Security List
- Default set of DHCP options

We can’t delete these default components; however, we can change their contents (e.g. individual route rules). And we can create more of each kind of component inyour cloud network (e.g. additional route tables)

#### Internal DNS

The VCN Private Domain Name System (DNS) enables instances to use hostnames instead of IP addresses to talk to each other
- Options:
      - **Internet and VCN Resolver:** default choice for new VCNs
      - **Custom Resolver:** lets instances resolve the hostnames of hosts in our on-premises network through IPsec VPN FastConnect
- Optionally specify a DNS label when creating VCN/subnets/instances
      - **VCN:** \<VCN DNS label>.oraclevcn.com
      - **Subnet:** \<subnet DNS label>.\<VCN DNS label>.oraclevcn.com
      - **Instance FQDN:** \<hostname>.\<subnet DNS label>.\<VCN DNS label>.oraclevcn.com
- Instance FQDN resolves to the instance's Private IP address
- No automatic creation of FQDN for Public IP addresses (e.g. cannot SSH using \<hostname>.\<subnet DNS label>.\<VCN DNS label>oraclevcn.com)

### Putting it all together

- Subnets can have one Route Table and multiple (5*) Security Lists associated to it
- Route table defines what can be routed out of VCN
- Private subnets are recommended to have individual route tables to control the flow of traffic outside of VCN
- All hosts within a VCN can route to all other hosts in a VCN (no local route rule required)
- Security Lists manage connectivity north-south (incoming/outgoing VCN traffic) and east-west (internal VCN traffic between multiple subnets)
- OCI follows a white-list model (you must manually specify white listed traffic flows); By default, things are locked down
- Instances cannot communicate with other instances in the same subnet, until you permit them to!
- Oracle recommends using NSGs instead of SLs because NSGs let you separate the VCN's subnet architecture from your application security requirements

## Connectivity to On-Premises Networks

#### Connectivity options

- Public Internet
      - Internet Gateway/ NAT Gateway
      - Reserved and Ephemeral IPs
      - Internet Data out Pricing (first 10TB free) 
      
- VPN
      - IPsec authentication and encryption
      - Two main options
            - OCI managed VPN Service (free)
            - Software VPN (running on OCI Compute)
- FastConnect
      - Private Connection
      - Separate from the internet
      - Consistent network experience
      - Port speeds of 1 Gbps and10 Gbps
      - SLA

### VPN Basics

VPN – using a public network to make end to end connection between two private networks in a secure fashion.

![pic20](/images/pic20.png)

- **Tunnel:** a way to deliver packets through the internet to private RFC 1918 addresses
- **Authentication:** provides a mechanism to authenticate who you are
- **Encryption:** packets need to be encrypted, so they cannot be sniffed over the public internet
- **Static routing:** configure a router to send traffic for particular destinations in preconfigured directions
- **Dynamic routing:** use a routing protocol such as BGP to figure out what paths traffic should take

### Dynamic Routing Gateway

![pic21](/images/pic21.png)

- A virtual router that provides a path for private traffic between our VCN and destinations other than the internet.
- We can use it to establish a connection with our on-premises network via IPsec VPN or FastConnect (private, dedicated connectivity)
- After attaching a DRG, we must add a route for the DRG in the VCN's route table to enable traffic flow.
- DRG is a standalone object. We must attach it to a VCN. VCN and DRG have a 1:1 relationship.

### VPN Connect (IPSec) - Workflow

![pic22](/images/pic22.png)

* Create a Virtual Cloud Network (VCN)
* Create a Dynamic Routing Gateway (DRG)
* Attach DRG to your VCN
* Update VCN Router to route traffic to DRG
* Create a CPE Object and add on-premises router Public IP address
* From DRG, Create an IPsec Connection between
* CPE and DRG and provide a Static Route or use BGP routing
* Configure on-premises CPE Router

https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/libreswan.htm

### FastConnect

FastConnect provides a dedicated and private connection with higher bandwidth options, and a more reliable and consistent networking experience when compared to internet-based connections

* Connect to OCI directly or via pre-integrated Network Partners
* Port speeds of 1 Gbps and 10 Gbps increments
* Extend remote datacenters into Oracle **Private peering** or connect to Public resources **Public peering**
* No charges for inbound/outbound data transfer
* Uses BGP (Border Gateway Protocol) protocol

#### Virtual Circuit

* Virtual circuit - isolated network path that runs over one or more physical network connections to provide a single, logical connection between customer's edge router and their DRG
* Each virtual circuit is made up of information shared between the customer, Oracle, and a provider
* Possible to have multiple virtual circuits to isolate traffic from different parts of organization (e.g. one virtual circuit for 10.0.1.0/24; another for 172.16.0.0/16), or to provide redundancy
* FastConnect uses BGP to exchange routing information

### FastConnect Use Scenarios

![pic23](/images/pic23.png)

- Private Peering
      - Extension of the on premise network to the OCI VCN
      - Communication across connection with private IP addresses
- Public Peering
      - To access public OCI services such as Object storage, OCI Console or APIs over dedicated FastConnect connection
      - Doesn’t use DRG (Dynamic Routing Gateway)

#### Public Peering Connection

* You choose which of your organization's public IP prefixes you want to use with the virtual circuit. Each prefix must be /31 or less specific.
* Oracle verifies your organization's ownership of each prefix before sending any traffic for it across the connection.
* When configuring your edge for public peering, make sure to give higher preference to FastConnect over your ISP
* Oracle prefers the most specific route when routing traffic from Oracle Cloud Infrastructure to other destinations that means even if you have a IGW, replies to your verified public prefixes will go over the FastConnect connection.
* You can add or remove public IP prefixes at any time by editing the virtual circuit

#### Private vs Public Peering 

|        | Private FastConnect | Public FastConnect | 
| ------ | ------------------- | ------------------ |
|**Use case**|To manage VCN resources privately | To access OCI’s public service offering|
|**Typical bandwidth** |Higher bandwidth; increments of 1 Gbps, and 10 Gbps ports |Higher bandwidth; increments of 1 Gbps, and 10 Gbps ports |
|**Protocols** |BGP |BGP |
|**Point-to-point IPs** |Customer assigns IPs (/30 or /31) | Oracle assign IPs (/30 or /31)|
|**Prefix-advertisement** |OCI advertises VCN subnet routes |OCI advertises public VCN routes and public Services routes |
|**Prefix-validation** |Not needed |OCI does validation that prefixes are owed by customer or not |
|**Prefix-limit** |2000 |200 |
|**BGP ASN** |Any ASN |Public ASN |

#### Use Cases of Dedicated Connectivity into Cloud

| Where | Why | 
|-------| --- |
|**Latency sensitive enterprise applications**| Applications with relational database especially vulnerable to latency and require predictable performance including backup, replication use cases
|**Big Data & High Performance Computing with data-transfer needs**| Large data transfer (for example batch jobs or real-time queries) require high performance and low latency
|**Sensitive data that cannot traverse the public internet**| Applications that contain sensitive data benefit from an extra level of privacy and isolation
|**Lift-and-shift to Cloud**| Moving Web-App-DB tiers to Oracle Cloud needs dedicated network connectivity

#### IPsec VPN vs FastConnect

| | IPsec VPN | FastConnect |
|-| --------- | ----------- |
| **Use case** | Dev/test and small scale production workloads | Enterprise-class and mission critical workloads, Oracle Apps, Backup, DR | 
| **Supported Services** |  All OCI Services within VCN | All OCI Services within VCN | 
| **Typical bandwidth** |  Typically < 250 Mbps aggregate |  Higher bandwidth; increments of 1 Gbps, and 10 Gbps ports | 
| **Protocols** | IPsec | BGP | 
| **Routing** | Static Routing, Dynamic Routing | Dynamic Routing
| **Connection Resiliency** | active-active | active-active | 
**Encryption** |  Yes, by default | No * (can be achieved using virtual firewall) | 
**Pricing** | Free for the managed service | Billable port hours, No data transfer charge between ADs | 
**SLA** | No SLA | 99.9% Availability SLA | 

### Load Balancer 

A load balancer sits between the clients and the backends performs tasks such as:
* **Service Discovery:** What backends are available in the system? How should the load balancer talk to them?
* **Health Check:** What backends are currently healthy and available to accept requests?
* **Algorithm:** What algorithm should be used to balance individual requests across the healthy backends?

- **Load Balancer benefits**
      - **Fault tolerance and HA:** using health check + LB algorithms, a LB can effectively route around a bad or overloaded backend
      - **Scale:** LB maximizes throughput, minimizes response time, and avoids overload of any single resource
      - **Naming abstraction:** name resolution can be delegated to the LB; backends don’t need public IP addresses

#### OCI Load Balancing Service

* Load Balancer as-a-service, provides scale and HA
* Public and Private Load Balancer options
* Supported Protocols – TCP, HTTP/1.0, HTTP/1.1, HTTP/2, WebSocket
* Supports SSL Termination, End-to-End SSL, SSL Tunneling
* Supports advanced features such as session persistence and content based routing
- Key differentiators
      - Private or Public Load Balancer (with Public IP address)
      - Provisioned bandwidth – 100 Mbps, 400 Mbps, 8 Gbps
      - Single load balancer for TCP (layer 4) and HTTP (layer 7) traffic

#### Public Load Balancer

* Accepts traffic from the internet using a public IP address that serves as the entry point for incoming traffic
* Public Load Balancer is a regional service
* If your region includes multiple availability domains, a public load balancer requires either a regional subnet (recommended) or two availability domain-specific (AD-specific) subnets, each in a separate availability domain.
* Load Balancing service creates a primary load balancer and a standby load balancer, each in a different availability domain
* Supports AD failover in the event of an AD outage in an Oracle Cloud Infrastructure multi-AD region
* Floating Public IP is attached to the primary load balancer, and in the event of an AD outage Floating
Public IP is attached to the standby load balancer
* Service treats the two load balancers as equivalent and you cannot denote one as "primary”

- **Architecture 1: *Regional Subnets - Recommended***

![pic24](/images/pic24.png)

- **Architecture 2: *AD Specific Subnets***

![pic25](/images/pic25.png)

- **Load Balancing Policiec:** Tells the load balancer how to distribute incoming traffic to the backend servers
      - **Round-robin:** Default policy, distributes incoming traffic sequentially to each server in a backends. After each server received a connection, the load balancer repeats the list in the same order.
      - **IP hash:*(Sticky Session)*** This policy ensures that requests from a particular client are always directed to the same backed server. 
      - **Least connection:** Routes incoming non-sticky request traffic to the backend server with the fewest active connections.
* **Backend Server:** Application server responsible for generating content in reply to the incoming TCP or HTTP traffic

- **Health Checks:** A test to confirm the availability of backend servers; supports
      - TCP-level
      - HTTP-level health checks
      - Health check is a test to confirm the availability of backend servers. Health Check is activated for; **backends, backend set, overall Load Balancer**
      - A load balancer IP can have up to 16 listeners (port numbers). Each listener has a backend set that can have 1 to N backend servers
      - Health API provied a 4-state health status (ok, warning, critical, unknown)
      - Health status is updated every three minutes. No finer granularity is available. 
* **Backend Set:** logical entity defined by a list of backend servers, a load balancing policy, and a health check policy
* **Listener:** entity that checks for incoming traffic on the load balancer's IP address

#### Private Load Balancer

* Assigned a private IP address from the subnet hosting the load balancer
* The load balancer can be regional or AD-specific, depending on the scope of the host subnet; highlyavailable within an AD with AD specific subnets or Highly available with regional subnets
* The primary and standby load balancer each require a private IP address from that subnet
* The load balancer is accessible only from within the VCN that contains the associated subnet, or as further restricted by your security list rules

- **Architecture 1: *Regional Subnets***

![pic26](/images/pic26.png)

- **Architecture 2: *AD Specific Subnets***

![pic27](/images/pic27.png)

## Compute

#### Bare Metal, VM and Dedicated Hosts

![pic28](/images/pic28.png)

VM compute instances runs on the same hardware as a Bare Metal instances, leveraging the same cloudoptimized hardware, firmware, software stack, and networking infrastructure.

**Bare Metal (BM):** Direct Hardware Access – customers get the full Bare Metal server (single-tenant model) 

**Virtual Machine (VM):** A hypervisor to virtualize the underlying Bare Metal server into smaller VMs (multi-tenant model) 

**Dedicated VM Hosts (DVH):** Run your VMs instances on dedicated servers that are a single tenant and not shared with other customers

**AMD EPYC Processors:** Amazon EC2 instances featuring AMD EPYC processors provide additional choices to help you optimize both cost and performance for your workloads.

AMD-based instances provide additional options for customers and may offer a better fit for many workloads that do not fully utilize the compute resources. By optimizing the balance between compute resources and utilization, these instances provide up to 10% lower cost than comparable instances.

**Use cases for AMD EPYC based instances** 

* AMD EPYC Bare Metal server (64 cores, 512 GB RAM, 2 x 25 Gbps bandwidth, 75 vNICs) available at $0.03 core/hour, 66% cheaper than other options
* AMD EPYC based instances ideal for maximizing price performance
* Supported for Oracle applications, including E-Business Suite, JD Edwards, and PeopleSoft
* Certified to run Cloudera, Hortonworks, MapR, and Transwarp
* On a 10-TB full TeraSort benchmark, including TeraGen, TeraSort and TeraValidate, the AMD EPYC based instance demonstrated a 40 percent reduction in cost / OCPU vs x86 alternatives with only a
very slight increase in run times
* On a 4-node, 14M cell Fluent CFD simulation of an aircraft wing, the AMD EPYC based instance demonstrated a 30 percent reduction in cost along with a slight reduction in overall run times as compared to an x86 alternative

### Images

**Oracle provided Images**

https://docs.oracle.com/en-us/iaas/Content/Compute/References/images.htm

**Linux Images:**

* User name opc created automatically for instances created from Oracle Linux/CentOS
* User name ubuntu created automatically for instances created from Ubuntu image
* These users have sudo privileges and are configured for remote access over the SSH v2
* Default set of firewall rules allow only SSH access (port 22)
* Provide a startup script using cloud-init 

**Windows Images:**

* User name opc created automatically with an OTP (one time password)
* Include the Windows Update utility to get the latest Windows updates from Microsoft 

**Custom Images:**

* Create a custom image of an instance’s boot disk and use it to launch other instances
* Instances you launch from your custom image include customizations, configuration, and software installed when you created the image
* During the process, instance shuts down and remains unavailable for several minutes. The instance restarts when the process completes
* Custom images do not include the data from any attached block volumes
* A custom image **cannot exceed 300 GB**
* **Windows custom images cannot be exported or downloaded out of the tenancy**

**Image Import/Export:**

Compute service enables you to share custom images across tenancies and regions using image import/export
* Image import/export uses OCI Object Storage service
* You can import Linux and Windows Operating System
- Supports:
      - **Emulation Mode:**
      - Virtual machines I/O devices (disk, network), CPU, and memory are implemented in software
      - Emulated VM can support almost any x86 operating system. These VMs are slow
      - **Paravirtualized:**
      - Virtual Machine includes a driver specifically designed to enable virtualization
      - **Native Mode:** same as Hardware Virtualized Machine (HVM), offers maximum performance with modern OS’s

* ***For more information about custom images:***
https://cloud.oracle.com/iaas/whitepapers/deploying_custom_os_images.pdf 

**Bring your own Image (BYOI):**

![pic29](/images/pic29.png)

The Bring Your Own Image (BYOI) feature enables you to bring your own versions of operating systems to the cloud as long as the underlying hardware supports it. The BYOI can help with the following scenarios:

* Enables lift-and-shift cloud migration projects
* Supports both old and new operating systems
* Encourages experimentation
* Increases infrastructure flexibility 

### Boot Volume 

* A compute instance is launched using OS image stored on a remote boot volume
* Boot volume is created automated and associated with an instance until you terminate the instance
* Boot volumes are encrypted, have faster performance, lower launch times, and higher durability for BM and VM instances
* Compute instance can be scaled to a larger shape by using boot volumes
* You can preserve the boot volume when you terminate a compute instance
* Boot volumes are only terminated when you manually delete them
* Boot volumes cannot be detached from a running instance
* Possible to take a manual backup, assign backup policy or create clone of boot volumes 

#### Custom Boot Volumes

* You have the option of specifying a custom boot volume size
* In order to take advantage of the larger size, you must first extend the root (Linux-based images) or system (Windows-based images) partition.
* Default boot volume size is 46.6 GB on Linux and it can be extended up to 100 GB
* Default boot volume size is 256 GB on Windows and it can be extended up to 500 GB
* The goal of a custom image is to create a gold image. Gold image meaning we have the operating system and all the configurations, customization, etc.

**Custom Images**

| Pros | Cons |
| ---- | ---- |
| You can export a custom image across regions and tenancies |Instance shuts down and remains unavailable for several minutes until the process finished | 
| No cost associated to store your custom images | Limit of 25 custom images per compartment |

**Boot Volume Backup**

| Pros | Cons |
| ---- | ---- |
|It doesn’t require a downtime|Cost associated with the amount of Object Storage used to store your backup|
|Preserve the entire state of your running operating system as a backup|Creating a boot volume backup while instance is running creates a crash-consistent backup|

### Instance Configurations, Pools, Autoscaling

Instance configurations includes; 
- OS image 
- Metadata
- Shape
- vNICs
- Storage
- Subnets

Config files becomes a template and we could spin up multiple instances using that template. It can be put;
- Different Availability Domains.
- Manage all together (stop, start, terminate)
- Attach to a Load Balancer.

**Instance Configurations**

* Clone an instance and save to a configuration file
* Create standardized baseline instance templates
* Easily deploy instances from CLI with a single configuration file
* Automate the provisioning of many instances, its resources and handle the attachments 

**Instance Pools**

* Centrally manage a group of instance workloads that are all configured with a consistent configuration
* Update a large number of instances with a single instance configuration change
* Maintain high availability and distribute instances across availability domains within a region
* Scale out instances on-demand by increasing the instance size of the pool

#### Autoscaling Configurations

* Autoscaling enables you to automatically adjust the number of Compute instances in an instance pool based on performance metrics such as CPU or Memory utilization.
* When an instance pool scales in, instances are terminated in this order: the number of instances is balanced across availability domains, and then balanced across fault domains. Finally, within a fault domain, the oldest instance is terminated first.

![pic30](/images/pic30.png)

#### Instance Metadata and Lifecycle

**Instance Metadata**

* Instance Metadata includes its OCID, name, compartment, shape, region, AD, creation date, state, image, and any custom metadata such as an SSH public key
* Service runs on every instance and is an HTTP endpoint listening on 169.254.169.254
* Get instance metadata by logging in to the instance and using the metadata service
- Oracle provided Linux instances
      - curl http://169.254.169.254/opc/v1/instance/
      - curl http://169.254.169.254/opc/v1/instance/metadata/
      - curl http://169.254.169.254/opc/v1/instance/metadata/\<key-name>/
* Add and update custom metadata for an instance using CLI or SDK

**Instance Life Cycle**

- **Start:** Restarts a stopped instance. After the instance is restarted, the Stop action is enabled
- **Stop:** Shuts down the instance. After the instance is powered off, the Start action is enabled
- **Reboot:** Shuts down the instance, and then restarts it
- **Terminate:** Permanently delete instances that you no longer need.
      - Instance's public and private IP addresses are released and become available for other instances
      - By default, the instance's boot volume is deleted, however you can preserve the boot volume and attach it to a different instance as a data volume, or use it to launch a new instance
- **Resource Billing**
      - Standard shapes, billing pauses in a STOP state
      - Dense I/O shapes, billing continues even in STOP state
      - GPU shapes, billing continues in STOP state
      - HPC shapes, billing continues in STOP state

**Summary**

* OCI Compute Service offers Bare Metal, Virtual Machine and Dedicated Hosts instances
* Bare Metal instances provide direct hardware access and highest level of performance andisolation
* Support a wide variety of shapes with industry leading price/performance
* Supports both x7 and AMD EPYC based instances with industry leading price/performance
* Image options - Oracle-provided images, BYOI, custom images, image import/export
* Advanced features include instance configuration, Pools and Autoscaling

### Oracle Container Engine for Kubernetes

#### Docker and Kubernetes

| Docker Containers | Kubernetes Orchestration |
| ----------------- | ------------------------ |
|Popular, easy to use tooling targeting developer productivity | Production grade container management targeting DevOps and operations, with widespread adoption |
| De facto standard container runtime and image format | Complex but powerful toolset supporting cloud scale applications |
| Used for developer on-boarding and 1st generation application management | Rich operations feature set, autoscaling, rolling upgrades, stateful apps and more.|
|60% of enterprise companies (500+ hosts) use Docker| 40% of Docker users also use orchestrators |
| 15% of all the hosts at these companies run Docker | 80% of these orchestration users prefer Kubernetes |

#### Oracle Container Engine for Kubernetes - OKE

* Managed Kubernetes container service to deploy and run your own container based apps
* Tooling to create, scale, manage & control your own standard Kubernetes clusters instantly
- Too complex, costly and time consuming to build & maintain environments
- Too hard to integrate Kubernetes with a registry and build process for container lifecycle management
- Too difficult to manage and control team access to production clusters
* Enables developers to get started and deploy containers quickly. Gives DevOps teams visibility and control for Kubernetes management.
* Combines production grade container orchestration of open Kubernetes, with control, security, IAM, and high predictable performance of Oracle’s next generation cloud infrastructure
- Kubernetes Challenges
      - Managing Kubernetes Infrastructure, upgrading, security.
      - Container networking & persistent storage
      - Managing Teams & Access
      - CI/CD Integration, automated testing, conditional release

#### Working with OKE and OCIR on OCI

![pic31](/images/pic31.png)

* Pay only for the OCI resources used to run your K8s clusters (VM's, Storage, LB, etc.)

#### Oracle Container Engine (OKE) and Registry

- **Container Native**
	- Standard Docker & Kubernetes
		- Deploy standard & open upstream Docker and Kubernetes versions for compatibility across environments
	- Registry Integration
		- Full Docker v2 compatible	private	registry to store and manage images
	- Container Engine
		- Deploy and operate containers and clusters
	- Full integration to cloud networking and storage
		- Leverage the enterprise class networking, load balancing and persistent storage of Oracle Cloud Infrastructure

- **Developer Friendly** 
	- Streamlined Workflow
		- Use your favorite CI to push containers to the	registry, then Kubernetes to deploy to clusters and manage operations
	- Full REST API
		- Automate the workflow, create and scale clusters through full REST API
	- Built In Cluster Add-Ons
		- Kubernetes Dashboard,DNS & Helm
	- Open Standards
		- Docker Based Runtime
		- Worker Node SSH Access
		- Standard Kubernetes

- **Enterprise Ready**
	- Simplified Cluster Operations
		- Fully managed, highly available registry, master nodes and control plane
		- One-click Quick Create for secure Private Worker Nodes/Subnets
	- Full Bare Metal Performance and Highly Available IaaS
		- Combine Kubernetes with bare metal shapes for raw performance
		- Deploy Kubernetes clusters across multiple Availability Domains for resilient applications
	- Team Based Access Controls
		- Control team access and permissions to clusters

### OCI Registry Service

* A high availability Docker v2 container registry service
* Stores Docker Images in Private or Public Repositories.
* Runs as a fully managed service on Oracle Cloud Infrastructure.

- Without a registry it is hard for Development teams to maintain a consistent set of Docker images for their containerized applications
- Without a managed registry it is hard to enforce access rights and security policies for images
- It is hard to find right images and have them available in the region of deployment

* Full integration with Container Engine for Kubernetes (OKE)
*  Registries are private by default, but can be made public by an admin
* Co-located regionally with Container Engine for low latency Docker image deploys
* Leverages OCI for high performance, low latency and high availability

![pic32](/images/pic32.png)

* Pay only for the OCI resources used to run your K8s clusters (VM’s, storage, LB, etc.) and store your images

#### OCIR Image Retention Policies

* Set up image retention policies to automatically delete images that meet particular selection criteria. Following rules can be applied.
	* images that have not been pulled for a certain number of days
	* images that have not been tagged for a certain number of days
	* images that have not been given particular Docker tags specified as exempt from automatic deletion
* **Hourly process** checks images against the selection criteria and deletes images accordingly.
* A global Image retention policy pre-exists with default selection criteria to retain all images.
* Users can edit global image retention policy or create their own custom policy.
* Policies are regional and applied on repository level.
* Repos can only be part of one image retention policy at a time
* Once the policy is created, first time it can take several hours to take effect known as cooling period to avoid unintentional deletion of images.

## Storage Services

### Object Storage service

* An internet-scale, high-performance storage platform
* Ideal for storing unlimited amount of unstructured data (images, media files, logs, backups)
* Data is managed as objects using an API built on standard HTTP verbs (PUT, GET)
* Regional service, not tied to any specific compute instance
* Offers two distinct storage classes to address the need for performant, frequently accessed "hot" storage, and less frequently accessed "cold" storage
* Supports private access from Oracle Cloud Infrastructure resources in a VCN through a Service Gateway
* Supports advanced features such as cross-region copy, pre-authenticated requests, lifecycle rules and multipart upload

#### Object Storage Scenarios 

* Content Repository - highly available and durable content repository for data, images, logs, and video etc.
* Archive/Backup - use of object storage for preserving data for longer periods of time
* Log Data - application log data for analysis and debugs/troubleshooting
* Large Data Sets - Large data e.g. pharmaceutical trials data, genome data, and Internet of Things (IoT)
* Big Data/Hadoop Support
* Use as a primary data repository for big data enables ~50% improvement in performance
* HDFS connector provides connectivity to various big data analytic engines like Apache Spark and MapReduce

#### Object Storage Service Features

* Strong consistency
	* Object Storage Service always serves the most recent copy of the data when retrieved
* Durability
	* Data stored redundantly across multiple storage servers across multiple ADs
	* Data integrity is actively monitored and corrupt data detected and auto repaired
* Performance
	* Compute and the Object Storage Services are co-located on the same fast network
* Custom metadata
	* Define your own extensive metadata as key-value pairs
* Encryption
	* Employs 256-bit Advanced Encryption Standard (AES-256) to encrypt object data

#### Object Storage Resources 

* Object
	* All data, regardless of content type, is managed as objects (e.g. logs, videos)
	* Each Object is composed of object itself and metadata of the object
* Bucket
	* A logical container for storing objects; Each object is stored in a bucket
* Namespace
	* A logical entity that serves as a top-level container for all buckets and objects
	* Each tenancy is provided one unique namespace that is global, spanning all compartments and regions
	* Bucket names must be unique within your tenancy, but can be repeated across tenancies
	* Within a namespace, buckets and objects exist in flat hierarchy, but you can simulate a directory structure using prefixes and hierarchies 

#### Object Naming

* Service prepends the Object Storage namespace string and bucket name to object name;
  * /n/<object_storage_namespace>/b/<bucket>/o/<object_name>
  * **Example:** https://objectstorage.us-phoenix1.oraclecloud.com/n/gse00014346/b/DatabaseBackup/o/database1.dbf
* Flat hierarchy
* For large number of objects, use prefixes and hierarchies,
  * /n/ansh8tvru7zp/b/event_photos/o/marathon/finish_line.jpg
  * /n/ansh8tvru7zp/b/event_photos/o/marathon/participants/p_21.jpg
* You can use the CLI to perform bulk downloads and bulk deletes of all objects at a specified level of the hierarchy, without affecting objects in levels above or below
* E.g. above, you can use CLI to download or delete all objects at the marathon/ level without downloading or deleting objects at the marathon/participants sublevel

#### Object Storage Tiers 

* **Standard Storage Tier (Hot)**
  * Fast, immediate, and frequent access
  * Object Storage Service always serves the most recent copy of the data when retrieved
  * Data retrieval is instantaneous
  * Standard buckets can’t be downgraded to archive storage 

* **Archive Storage Tier (Cold)**
  * Seldom or rarely accessed data but must be retained and preserved for long periods of time
  * Minimum retention requirement for Archive Storage is 90 days
  * Objects need to be restored before download
  * Archive Bucket can’t be upgraded to Standard storage tier
  * Time To First Byte (TTFB) after Archive Storage restore
request is made: 4 Hours

#### Object Storage Capabilities

* **Pre-Authenticated Requests**
  * Provides a way to let users access a bucket or an object without
having their own credentials
  * Can access via a unique URL.**(p in example means pre-authenticated)**  
  Example: https://objectstorage.us-ashburn1.oraclecloud.com/p/p09Nx-f4UaLCNMMOxGQIpobmMchgHQrSQv4LraSzs/n/intoraclerohit/b/Image/o/kvm
  * Can revoke the links any time (much easier than S3)
* **Public Buckets**
  * At creation, a bucket is considered private and access to the bucket requires authentication and authorization
  * Service supports anonymous, unauthenticated access to a bucket by making a bucket public (read access to the bucket)
  * Changing the type of access doesn't affect existing pre-authenticated requests. Existing pre-authenticated requests still work

#### Cross-region Copy 

* Copy objects to other buckets in the same region and to buckets in other regions
* Must authorize the service to manage objects on your behalf (separate policy for each region).
  * allow service objectstorage-us-ashburn1 to manage object-family in tenancy
* Must specify an existing target bucket
* Bulk copying is not supported
* Objects cannot be copied from Archive storage

#### Object Lifecycle Management 

* Define lifecycle rules to automatically archive or delete objects after a specified number of days
* Must authorize the service to manage objects on your behalf (separate policy/region).
  * allow service objectstorage-us-ashburn-1 to manage object-family in tenancy
* Applied at the bucket or object name prefix level. If
no prefix is specified, the rule will apply to all objects in the bucket
* A rule that deletes an object always takes priority over a rule that would archive that same object
* Enable or disable a rule to make it active or inactive 

#### Managing Multipart Uploads

With multipart uploads, individual parts of an object can be uploaded in parallel to reduce the amount of time you spend uploading. Steps involved;

1. Create object parts
  * Perform a multipart upload to upload objects larger than 100 MiB. Individual parts can be as large as 50 GiB or as small as 10 MB
  * Assign part numbers from 1 to 10,000
2. Initiate an upload
  * Initiate a multipart upload by making a CreateMultipartUpload REST API call
3. Upload object parts
  * Make an UploadPart request for each object part upload
  * If you have network issues, you can restart a failed upload for an individual part. You do not need to restart the entire upload
4. Commit the upload
  * When you have uploaded all object parts, complete the multipart upload by committing it; add a bullet on checksum etc.

#### Summary
* An internet-scale, high-performance storage platform
* Regional service, not tied to any specific compute instance
* Offers two distinct storage classes to address the need for performant, frequently accessed "hot" storage, and less frequently accessed "cold" storage
* Supports private access from Oracle Cloud Infrastructure resources in a VCN through a Service Gateway
* Supports advanced features such as cross-region copy, life cycle management, pre-authenticated requests and multipart uploads 

### Block Volume 

**File Storage:** Manage data as a file hierarchy, 
**Block Storage:** Manage data as fixed-size blocks
**Object Storage:** Manage data as objects.

| | Local NVMe | Block Volume | File Storage | Object Storage | Archive Storage |
|-| ---------- | ------------ | ------------ | -------------- | --------------- |
|**Type**| NVMe SSD based temporary storage | NVMe SSD based block storage | NFSv3 compatible file system | Highly durable Object storage | Long-term archival and backup |
|**Durability**| Non-persistent; survives reboots | Durable (multiple copies in an AD) | Durable (multiple copies in an AD) | Highly durable (multiple copies across ADs) | Highly durable (multiple copies across ADs) 
|**Capacity**| Terabytes+ | Petabytes+ | Exabytes+ | Petabytes+ | Petabytes+
|**Unit Size**| 51.2 TB for BM, 6.4-25.6 TB for VM | 50 GB - 32 TB/vol. 32 vols/instance | Up to 8 Exabyte | 10 TB/object | 10 TB/object |
|**Use cases**| Big Data, OLTP, high performance workloads | Apps that require SAN like features (Oracle DB, VMW, Exchange) | Apps that require shared file system (EBS, HPC) | Unstructured data incl. logs, images, videos | Long term archival and backups (Oracle DB backups) |

#### Local NVMe Storage 

**Local NVMe SSD Devices**

* Some instance shapes in OCI include locally attached NVMe devices
* Local NVMe SSD can be used for workloads that have high storage performance requirements
* Locally attached SSDs are not protected and OCI provides no RAID, snapshots, backups capabilities for these devices
* Customers are responsible for the durability of data on the local SSDs

**NVMe SSD Persisted - Reboot/Pause**

![pic33](/images/pic33.png)

With Oracle Cloud Infrastructure, companies can leverage NVMe for persistent storage to host databases and applications. 

However, other cloud providers typically do not offer such a capability. In cases where NVMe storage was an option with other vendors, it was not persistent. 

This meant that the multi-terabyte database that researchers loaded to this storage was lost when the server stopped.

##### Protecting NVMe SSD Devices 

RAID 1: An exact copy (or mirror) of a set of data on two or more disks pair is functional, data can be retrieved

RAID 10: Stripes data across multiple mirrored pairs. As long as one disk in each mirrored

RAID 6: Block-level striping with two parity blocks distributed across all member disks
 
##### SLA for NVMe Performance

![pic34](/images/pic34.png)

* OCI provides a service-level agreement (SLA) for NVMe performance
* Measured against 4k block sizes with 100% random write workload on Dense IO shapes where the drive is in a steady-state of operation
* Run test on Oracle Linux shapes with 3rd party 
  * Benchmark Suites, https://github.com/cloudharmony/blockstorage

| Shape         |Minimum Supported IOPS |
| ------------- | --------------------- |
|VM.DenseIO1.4  | 200k  |
|VM.DenseIO1.8  | 250k  |
|VM.DenseIO1.16 | 400k  |
|BM.DenseIO1.36 | 2.5MM |
|VM.DenseIO2.8  | 250k  |
|VM.DenseIO2.16 | 400k  |
|VM.DenseIO2.24 | 800k  |
|BM.DenseIO2.52 | 3.0MM |

### Block Volume Intro

#### Block Volume Service 

* Block Volume Service let you store data on block volumes **independently** and beyond the lifespan of compute instances
* Block volumes operates at the raw storage device level and manages data as a set of numbered, fixed-size blocks using a protocol such as iSCSI
* You can create, attach, connect, and move volumes, as needed, to meet your storage and application requirements
* Typical Scenarios
  * Persistent and Durable Storage
  * Expand an Instance's Storage
  * Instance Scaling

##### Block Volume Service (contd.)

* For Bare Metal or 8-core+ VM compute instance, using 4KB blocks. VM perf is limited by VM network bandwidth.**256 KB block size** 

**Capacity:** Configurable: 50 GB to 32 TB (1GB increments)
**Perf-disk type:** NVMe SSD based
**Perf-IOPS:** 60 IOPS/GB - up to 25K IOPS*
**Perf-Throughput/Vol:** 480 KBPS/GB - up to 320 MBPS**
**Perf-Latency (P95):** Sub-millisecond latencies
**Perf-Per-instance Limits:** 
- 32 attachments/instance, up to 1 PB (32 TB/volume x 32 volumes/instance)
- Up to 620K or more IOPS, near line rate throughout.

**Durability:** Multiple replicas across multiple storage servers within the AD 
**Security:** Encrypted at rest and transit 

##### Creating and Attaching a Block Volume

**Paravirtualization** is a light virtualization technique where a VM utilizes hypervisor APIs to access remote storage directly as if it were a local device

**iSCSI** block storage attachment utilizes the internal storage stack in the guest OS and network hardware virtualization to access block volumes. Hypervisor is not involved in the iSCSI attachment process

By default, all Block Volumes are Read/Write.
Block Volume can also be read-only to prevent against accidental modification 

##### Detaching and Deleting Block Volumes

* When an instance no longer requires a block volume, you can disconnect and then detach it from the instance without any loss of data
* When you attach the same volume to another instance or to the same instance, DO NOT FORMAT the disk volume. Otherwise, you will lose all the data on the volume
* When the volume itself is no longer needed, you can delete the block volume
* You cannot undo a delete operation. Any data on a volume will be permanently deleted once the volume is deleted

##### Block Volume Offline Resize

![pic35](/images/pic35.png)

The Oracle Cloud Infrastructure Block Volume service lets you expand the size of block volumes and boot volumes. You have three options to increase the size of your volumes:

* Expand an existing volume in place with offline resizing (cannot resize an attached volume)
* Restore from a volume backup to a larger volume.
* Clone an existing volume to a new, larger volume. 
* You can only increase the size of the volume, you cannot decrease the size

#### Backup and Restoration

* Complete point-in-time snapshot copy of your block volumes
* Encrypted and stored in the Object Storage Service, and can be restored as new volumes to any Availability Domain within the same region (for multi-AD regions)
* Can copy block volume backups from one-region to another

![pic36](/images/pic36.png)

* Backups are done using point-in-time snapshot; therefore, while the backup is being performed in the background asynchronously, your applications can continue to access your data without any
interruption or performance impact
  * For a 2 TB volume being backed up for the first time, ~30 mins
  * For a 50 GB boot volume being backed up for the first time, ~ few mins
* On-demand, one-off block volume backups provide a choice of incremental versus full backup options

**Backup options:**
* On-demand, one-off: point-in-time snapshot
* Automated policy-based: backups automatically on a schedule and retain them based on the selected backup policy. Three backup policies:
  * **Bronze:** monthly incremental backups, retained for twelve months (+full yearly backup, retained for 5 years)
  * **Silver:** weekly incremental backups, retained for four weeks (+ Bronze)
  * **Gold:** daily incremental backups, retained for seven days (+Silver, + Bronze)
* Customized backup policy not available today

#### Clone and Volume Groups

##### Clone

* Cloning allows copying an entire existing block volume to a new volume without needing to go through a backup and restore process
* Clone is a point-in-time direct disk-to-disk deep copy an of entire volume
* The clone operation is immediate, but actual copying of data happens in the background and can take up to 15 minutes for 1 TB volume
* A clone can only be created in the same AD with no need of detaching the source volume before cloning it
* Clones cannot be copied to another region
* A clone can be attached and used as regular volume when its lifecycle state changes from ”PROVISIONING” to "AVAILABLE", usually within seconds
* Clone and backup operations are mutually exclusive
* Number of clones created simultaneously
  * If the source volume is attached: You can create one clone at a time
* If the source volume is detached: You can create up to 10 clones from the same source volume simultaneously

##### Volume Groups

![pic37](/images/pic37.png)

* Group together block and boot volumes from multiple compartments across multiple compute instances in a volume group
* You can use volume groups to create volume group backups and clones that are point-in-time and crash-consistent.
* Manually trigger a full or incremental backup of all the volumes in a volume group leveraging a coordinated snapshot across all the volumes.
* This is ideal for the protection and lifecycle management of enterprise applications, which typically require multiple volumes across multiple compute instances to function effectively
* Volume Group feature is available with no additional charge

#### Boot Volumes

* A compute instance is launched using OS image stored on a remote boot volume
* Boot volume is created automated and associated with an instance until you terminate the instance
* Boot volumes are encrypted, have faster performance, lower launch times, and higher durability for BM and VM instances
* Launch another instance with a boot volume
  * First create a custom image of your boot volume and then using the custom image launch the instance
  * Alternately, you can launch a new instance directly from an unattached boot volume if you don't wish to create a custom image
* Delete boot volume*
  * You can delete an unattached boot volume.
  * You can optionally chose to automatically delete the boot volume when terminating an instance by selecting the checkbox in the delete confirmation dialog.
* OCI does not allow you to delete the boot volume currently attached to an instance.
* Possible to take a manual backup, assign backup policy or create clone of boot volumes

- Attach a Boot Volume to an instance as a block volume for troubleshooting
  - You can attach any boot volume to an instance as block storage in order to debug issues. You will first need to detach a boot volume from its associated compute instance in order to attach it to a different instance.
  - You can follow the steps below to debug your boot volume:
  - 'Stop' the instance you want to debug and click on 'Boot Volume' filter, and then select the 'Detach Boot Volume' button. Alternately, you can terminate your instance which persists your boot volume by default.
  - Navigate to a new running instance you want to use to debug your boot volume, and click the 'Attach Block Volume' button.

##### Custom Boot Volumes

You have the option of specifying a custom boot volume size.

In order to take advantage of the larger size, you must first extend the root (Linux-based images) or system (Windows-based images) partition.

##### Summary

* OCI offers local NVMe SSD storage with SLAs for high-performance workloads
* OCI Block Volume service - persistent, durable, high-performance block service with industry leading price/performance
* Create, attach, connect, and move volumes, as needed, to meet your storage and application requirements
* Block volume service supports backups (on-demand, Policy based) and restoration
* Cloning and Policy based backups offered only by OCI Block Volume service
* Another unique feature, Volume Groups simplifies backups of running enterprise applications that span multiple storage volumes across multiple instances.

#### File Storage Service Intro

**Use Cases:**
* Oracle Applications Lift and Shift
* General Purpose File Systems
* Big Data & Analytics
* HPC Scale Out Apps
* Test / Dev Databases
* MicroServices Containers

##### File Storage service Features

* AD-local service, available in all OCI regions and Availability Domains
* Supports NFS v.3
* Network Lock Management (NLM) for file locking
* Full POSIX semantics
* Data Protection: Snapshots capabilities; 10,000 snapshots per file system
* Security: 128-bit, data-at-rest encryption for all file systems & metadata
* Console management, APIs, CLI, data-path commands, and Terraform
* Create 100 file systems and 2 mount targets per AD per account

##### Mount Target

![pic38](/images/pic38.png)

* NFS endpoint that lives in your subnet of choice; AD-specific
* Mount target has an IP address and DNS name that you can use in your mount command. **Example:** 10.0.0.6
* Requires three private IP addresses in the subnet (don’t use /30 or smaller subnets for the FSS)
* Two of the IP addresses are used during mount target creation; 3rd IP used for HA

- Placing NFS clients and mount target in the same subnet can result in IP conflicts, as users are not shown which private IPs are used for mount target
- Place FSS mount target in its own subnet, where it can consume IPs as it needs

##### File System

* Primary resources for storing files in FSS
* To access your file systems, you create a new (or use an existing) mount target
* 100 File Systems per Mount Target
* AD-specific
* Accessible from OCI VM/BM instances
* Accessible from on-premises through FastConnect/VPN

##### FSS Paths

* **Export Path:** unique path specified when the file system is associated with a mount target during creation
* No two File systems associated with the same mount target can have overlapping export paths (e.g. FS paths like /example and /example/path are not allowed)

```
Mount	target	(NFS	endpoint):	10.0.0.6
Export	Path1:	/example1/path
Export	Path2:	/example2/path
```

* Export path, along with the mount target IP address, is used to mount the file system to an instance
  * sudo mount 10.0.0.6:/example1/path /mnt/mountpointA
  * sudo mount 10.0.0.6:/example2/path /mnt/mountpointB
  * /mnt/mountpointA and /mnt/mountpointB are path to the directory on the NFS client instance on
which the external file systems are mounted

##### Mounting an OCI File System

* Launch OCI instance from console
* Use NFSv3 protocol to mount the FSS volume
* Install nfs-utils (Oracle Linux and CentOS) or nfs-common (Ubuntu) in your Linux system
* Create a directory
* On the FSS console, click on Mount Targets
* Use the Private IP address information to mount the volume using nfs command:

```
opc@node01:~$ sudo mkdir -p /<user’s target directory>
opc@node01:~$ sudo mount <IPaddress>:<path-name> /<user’s target directory>
opc@node01:~$ sudo yum install nfs-utils
opc@node01:~$ sudo mkdir -p /mnt/nfs
opc@node01:~$ sudo mount 10.0.0.3:/fss-shared /mnt/nfs
```

**NOTE:** *Oracle recommend not to pass mount options to achieve best performance with File Storage Service. This approach leaves it to the client and server to negotiate the window size for Read & Write operations.*

#### Security

Four distinct and separate layers of security with its own authorization entities and methods to consider when using FSS

|Security layer|Uses these..|To control actions like these..|
|--------------|------------|-------------------------------|
|IAM Service| OCI users, policies |Creating instances (NFS clients) and FSS VCNs. Creating, listing, and associating file systems and mount targets|
|Security Lists| CIDR blocks| Connecting the NFS client instance to the mount target|
|Export Options| Export options, CIDR blocks| Applying access control per-file system based on source IP CIDR blocks that bridges the Security Lists layer and the NFS v.3 Unix Security layer|
|NFS v3. Unix Security| Unix users| Mounting file systems1, reading the writing files, file access security|

*When mounting file systems, don't use mount options such as nolock, rsize, or wsize. These options cause issues with performance and file locking*

##### Security Lists

![pic39](/images/pic39.png)

Security List can be used as a virtual firewall to prevent NFS clients from mounting an FSS mount target (even in the same subnet). FSS needs;
* Stateful ingress TCP ports 111, 2048 – 2050
* Stateful ingress UDP ports 111 and 2048
* Opening these ports enables traffic from Solaris, Linux, and Windows NFS clients

![pic40](/images/pic40.png)

*For all subnets within VCN (e.g. 10.0.1.0/24) to access File System, change destination CIDR to 10.0.0.0/16; all rules stateful*

##### Export Option

* Security List is all or nothing approach – the client either can or cannot access the mount target, and therefore all file systems associated with it
* In a multi-tenant environment, using NFS export option, you can limit clients' ability to connect to the file system and view or write data
* Export controls how NFS clients access file systems; info stored in an export includes the file system OCID, export path, and client access options
* When you create file system and associated mount target, the NFS export options for that file system are set to allow full access for all NFS clients :
  * **Source:** 0.0.0.0/0 (All)
  * **Require Privileged Source Port:** False
  * **Access:** Read_Write
  * **Identity Squash:** None

![pic41](/images/pic41.png)

* Client X, assigned to 10.0.0.0/24, requires Read/Write access to file system A, but not file system B
* Client Y, assigned to 10.0.1.0/24, requires Read access to file system B, but no access to file system A
* Both file systems A and B are associated to a single mount target

> oci fs export update --export-id <FS_A_export_ID> --exportoptions '[{"source":"10.0.0.0/24 ","require-privileged-sourceport":"true","access":"READ_WRITE","identitysquash":"NONE", "anonymousuid":"65534","anonymousgid":"65534"}]'


> oci fs export update --export-id <FS_B_export_ID> --exportoptions '[{"source":"10.0.1.0/24","require-privileged-sourceport":"true","access":"READ_ONLY","identitysquash":"NONE""anonymousuid":"65534","anonymousgid":"65534"}]'

#####  File Storage Service Snapshots

* Snapshots provide a read-only, space efficient, point-in-time backup of a file system
* Snapshots are created under the root folder of file system, in a hidden directory named .snapshot
* You can take up to 10,000 snapshots per file system
* You can restore a file within the snapshot, or an entire snapshot using the;
  * cp or rsync command 
    * cp -r .snapshot snapshot_name/* destination_directory_name
* If nothing has changed within the target file system and you take a snapshot, it does not consume any additional storage

##### Summary

* OCI File Storage Service provides a fully managed, elastic, durable, distributed, enterprise-grade network file system
* FSS supports NFS v3, snapshots and default data-at-rest encryption
* FSS is highly scalable (Exabytes) and performant
* FSS supports four distinct and separate layers of security with its own authorization entities and methods

### Database

#### OCI Database Service 

* Mission critical, enterprise grade cloud database service with comprehensive offerings to cover all enterprise database needs
  * Exadata, RAC, Bare Metal, VM
* Complete Lifecycle Automation
  * Provisioning, Patching, Backup & Restore
* High Availability and Scalability
  * RAC & Data Guard
  * Dynamic CPU and Storage scaling
* Security
  * Infrastructure (IAM, Security Lists, Audit logs)
  * Database (TDE, Encrypted RMAN backup / Block volume encryption)
* OCI Platform integration
  * Tagging, Limits and Usage integration
* Bring Your Own License (BYOL)

#### Virtual Machine (VM) Database (DB) Systems 

* There are 2 types of DB systems on virtual machines:
  * A 1-node VM DB system consists of one VM.
  * A 2-node VM DB system consists of two VMs clustered with RAC  enabled.
* VM DB systems can have only a single database home, which in turn can have only a single database.
* Amount of memory allocation for the VM DB system depends on the VM shape selected during the provisioning process.
* Size of storage is specified when you launch a VM DB system and you scale up the storage as needed at any time.
* The number of CPU cores on an existing VM DB system cannot be changed.
* If you are launching a DB system with a virtual machine shape, **you have option of selecting an older database version**. Check Display all database versions to include older database versions in the dropdown list of database version choices.
* When a 2-node RAC VM DB system is provisioned, the system assigns each node to a different fault domain by default.
* Data Guard within and across ADs is available for VM DB systems (requires DB Enterprise Edition).

##### VM DB Systems Storage Architecture 

![pic42](/images/pic42.png)

* ASM relies on OCI Block Volume (based on NVMe) for mirroring data
* Block volumes are mounted using iSCSI
* ASM uses external redundancy relying on the triple mirroring of the Block Storage
* Different Block Storage volumes are used for DATA and RECO
* Monitors the disks for hard and soft failures
* These actions ensure highest level availability and performance at all times
* This storage architecture is required for VM RAC DB systems

##### VM DB Systems Storage Architecture – Fast Provisioning Option 

![pic43](/images/pic43.png)

* Linux Logical Volume Manager manages the filesystems used by the database for storing database files, redo logs, etc.
* Block volumes are mounted using iSCSI
* **The available storage value you specify during provisioning determines the maximum total storage available through scaling**
* VM RAC DB Systems cannot be deployed using this option
* Currently supports Oracle Database 18c and 19c releases

https://docs.oracle.com/en-us/iaas/Content/Database/References/fastprovisioningstorage.htm

#### Bare Metal DB Systems 

![pic44](/images/pic44.png)

* Bare Metal DB Systems rely on Bare Metal servers running Oracle Linux Oracle Database ASM for 12c +, ACFS for 11g DB Management Agent Oracle Linux 6.8
* One-node database system:
  * Single Bare Metal server
  * Locally attached 51 TB NVMe storage (raw)
  * Start with 2 cores and scale up/down OCPUs based 52 CPU cores 768 GB RAM 51 TB NVMe raw on requirement
  * Data Guard within and across ADs (requires DB Enterprise Edition)
  * If single node fails, launch another system and restore the databases from current backups

##### Bare Metal DB Systems Storage Architecture 

![pic45](/images/pic45.png)

* ASM manages mirroring of NVMe disks
* Disks are partitioned – one for DATA and one for RECO
* Monitors the disks for hard and soft failures
* Proactively offlines disks that failed, predicted to fail, or are performing poorly & performs corrective actions, if possible
* On disk failure, the DB system automatically creates an internal ticket and notifies internal team to contact the customer
* These actions ensure highest level availability and performance at all times

#### Exadata DB Systems 

![pic46](/images/pic46.png)

* Full Oracle Database with all advanced options
* On fastest and most available database cloud platform
  * Scale-Out Compute, Scale-Out Storage, Infiniband, PCIe flash
  * Complete Isolation of tenants with no overprovisioning
* All Benefits of Public Cloud
  * Fast, Elastic, Web Driven Provisioning
  * Oracle Experts Deploy and Manage Infrastructure

- Oracle manages Exadata infrastructure - servers, storage, networking, firmware, hypervisor, etc.
- You can specify zero cores when you launch Exadata; this provisions & immediately stops Exadata
- You are billed for the Exadata infrastructure for the first month, and then by the hour after that. Each OCPU you add to the system is billed by the hour from the time you add it
- Scaling from ¼ to a ½ rack, or from ½ to a full rack requires that the data associated with database deployment is backed up and restored on a different Exadata DB system  

|Resource| Base System| Quarter Rack| Half Rack| Full Rack|
|--------|------------|-------------|----------|----------|
|        |            |   X6 - X7   |X6 - X7   |   X6 - X7|
|Number of Compute Nodes|2|2|4|8|
|Total Minimum (Default) Number of Enabled CPU Cores|0|22 - 0|44 - 0|88 - 0|
|Total Maximum Number of Enabled CPU Cores|48|84 - 92|168 - 184|336 - 368|
|Total RAM Capacity|720 GB|1440 GB|2880 GB|5760 GB|
|Number of Exadata Storage Servers|3|3|6|12|
|Total Raw Flash Storage Capacity|38.4 TB| 38.4 TB - 76.8 TB| 76.8 TB - 153.6 TB| 153.6 TB - 307.2 TB|
|Total Raw Disk Storage Capacity|252 TB |288 TB - 360 TB| 576 TB - 720 TB| 1152 TB - 1440 TB|
|Total Usable Storage Capacity|74.8 TB| 84 TB - 106 TB| 168 TB - 212 TB| 336 TB - 424 TB|

##### Exadata DB Systems Storage Architecture 

![pic47](/images/pic47.png)

* Backups provisioned on Exadata storage: ~ 40% of the available storage space allocated to DATA disk group and ~60% allocated to the RECO disk group
* Backups not provisioned on Exadata storage: ~ 80% of the available storage space allocated to DATA disk group and ~20% allocated to the RECO disk group
* After the storage is configured, the only way to adjust the allocation without reconfiguring the whole environment is by submitting a service request to Oracle

#### DB Systems – VM, BM, Exadata 

| |Virtual Machine(VM)|Bare Metal(BM)| Exadata|
|-|-------------------|----------|------------|
|Scaling|Storage (number of CPU cores on VM DB cannot be changed)|CPU (amount of available storage cannot be changed|CPU can be scaled within a ¼ , ½ and Full rack. Storage cannot be scaled|
|Multiple Homes/Databases|**No, single DB and Home only****|Yes (one edition, but different versions possible)|Yes|
|Storage|Block Storage|Local NVMe disks|Local spinning disks and NVMe flash cards|
|Real Application Clusters (RAC)|Available (2-node)| Not Available |Available|
|Data Guard| Available| Available| **Available***|

*You can manually configure Data Guard on Exadata DB systems using native Oracle Database utilities and commands. dbcli is not available on Exadata DB systems

**The database can be a container database with multiple pluggable databases, if the edition is High Performance or Extreme Performance.

##### Database Editions and Versions 

| |VM DB Systems|BM DB Systems|Exadata DB Systems| DB Versions |
|-|-------------|-------------|------------------|-------------|
|Standard Edition|Yes| Yes| No| 11.2.0.4, 12.1.0.2, 12.2.0.1, 18.1.0.0, 19.3*|
|High Performance|Yes| Yes| No|11.2.0.4, 12.1.0.2, 12.2.0.1, 18.1.0.0, 19.3*|
|Extreme Performance|Yes| Yes| Yes|11.2.0.4, 12.1.0.2, 12.2.0.1, 18.1.0.0, 19.3*|
|BYOL| Yes|Yes|Yes|11.2.0.4, 12.1.0.2, 12.2.0.1, 18.1.0.0, 19.3*|

*Note that Oracle Database 19c is only available on VM DB and Exadata DB Systems (as of September 2019)*

![pic48](/images/pic48.png)

*Note that all editions include Oracle Database Transparent Data Encryption (TDE)*

#### Managing DB Systems 

You can use the console to perform the following tasks:
* Launch a DB System: You can create a database system
  * **Status check:** You can view the status of your database creation and after that, you can view the runtime status of the database
* Start, stop, or reboot DB Systems
  * Billing continues in stop state for BM DB Systems (but not for VM DB)
* **Scale CPU cores:** scale up the number of enabled CPU cores in the system (BM DB systems only)
* **Scale up Storage:** increase the amount of Block Storage with no impact (VM DB systems only)
* **Terminate:** terminating a DB System permanently deletes it and any databases running on it 

##### Patching DB Systems

* **Automated Applicable Patch Discovery:** Automatic patch discovery and pre-flight checks/tests
* **On demand patching:** N-1 patching (previous patch is available if it hasn’t been applied), pre-check and patching at the click of a button
* **Availability during patching:** For Exadata and RAC shapes, patches are rolling. For single node systems if Active Data Guard is configured this can be leveraged by the patch service.
* **2 step process** – patching is a 2 step process, one for DB System and one for the database. DB System needs to be patched first before the database is patched
* **Identity and Access Controls:** Granular Permissions – its possible to control who can list patches, apply them, etc. 

##### Backup / Restore 

* Managed backup and restore feature for VM/BM DB Systems; Exadata backup process requires creating a backup config file
* Backups stored in Object or Local storage (**recommended:** Object storage for high durability)
* DB System in private subnets can leverage Service Gateway
* Backup options
* Automatic incremental – runs once/day, repeats the cycle every week; retained for 30 days
* On-demand, standalone/ full backups
* Restore a DB 

##### Automatic Backups

* By default, automatic backups are written to Oracle owned object storage (customers will not be able to view the object store backups)
* Default policy cannot be changed at this time
* Automatic backups enabled for the first time after November 20, 2018 on any database will run between midnight and 6:00 AM in the time zone of the DB system's region
* You can optionally specify a 2-hour scheduling window for your database during which the automatic backup process will begin
* These are the preset retention periods for automatic backups: 7 days, 15 days, 30 days, 45 days and 60 days.
* Backup jobs are designed to be automatically re-tried
* Oracle automatically gets notified if a backup job is stuck
* All backups to cloud Object Storage are encrypted
* Link to troubleshooting backup issues 
  * https://docs.us-phoenix1.oraclecloud.com/Content/Database/Troubleshooting/Backup/backupfail.htm

##### High Availability and Scalability

* Robust Infrastructure
* Region with 3 Availability Domains architecture
* Fully redundant and non-blocking Networking Fabric
* 2-way or 3-way mirrored storage for Database
* Redundant Infiniband Fabric (Exadata) for cluster networking
* Database Options to enable HA
* Database RAC Option in VMs and Exadata
* Automated Data Guard within and across ADs
* Dynamic CPU and Storage Scaling

##### Oracle Data Guard

* Robust Infrastructure
* Supported on both Virtual Machine and Bare Metal DB Systems.
* Limited to one Standby database per Primary database on OCI.
* Standby database used for queries, reports, test, or backups (**only for Active Data Guard**)
* Switchover
* Planned role reversal, never any data loss
* No database re-instantiation required
* Used for database upgrades, tech refresh, data center moves, etc.
* Manually invoked via Enterprise Manager, DGMGRL, or SQL*Plus
* Failover
* Unplanned failure of Primary
* Flashback Database used to reinstate original Primary
* Manually invoked via Enterprise Manager, DGMGRL, or SQL*Plus
* May also be done automatically: Fast-Start Failover

#### OCI Security Features Overview for Database Service 

|Security capability|Features|
|-------------------|---------------------------|
|**Instance security isolation**|BM DB Systems|
|**Network security and access control**|VCN, Security Lists, VCN Public and Private subnets, Route Table, Service Gateway|
|**Secure and Highly-available Connectivity**| VPN DRGs, VPN and FastConnect |
|**User authentication & authorization**| IAM Tenancy, Compartments and security policies, console password, API signing key, SSH keys|
|**Data encryption**|DBaaS TDE, RMAN encrypted back-ups, Local storage and Object storage encryption at rest|
|**End-to-end TLS**|LBaaS with TLS1.2, Customer-provided certificates|
|**Auditing** |OCI API audit logs |

#### Pricing

https://www.oracle.com/cloud/price-list.html

#### Summary

* Database service offers mission critical enterprise grade cloud database service with comprehensive offerings – Exadata, RAC, Bare Metal, VMs to cover every enterprise need
* Offers complete lifecycle automation – Provisioning, Patching, Backup, Restore
* Scalability from 1 core VM to Exadata and high-availability options – Data Guard, RAC
* Provides robust Security controls
* Supports BYOL model 

#### Autonomous Database 

![pic49](/images/pic49.png)

| Service | Features | Use Cases |
| ------- | -------- | --------- |
|**Autonomous Database**|World’s Best Fully Self-Driving Database. Oracle Builds and Operates Exadata Infrastructure and Databases. User runs SQL, no Access to OS or Container DB | Cloud elasticity, Machine Learning, Self driving. Instant Provisioning, Always online operation. All workloads, JSON Documents, Graphs, and more|
|**Oracle Database Cloud Services**|World’s Best Automated Database Cloud. Oracle Builds and Operates Infrastructure. User Operates Databases Using Provided Lifecycle Automation. User Has Full Control, including DBA and Root Access.|Availability, Flexible Version and Features. Small to Large DB deployment. Single Instance or RAC, Automated Backup, Patching, Customer controls. 
|**Exadata**|World’s Best Database Platform. Oracle Builds, Optimizes, and Automates Infrastructure. All In-Database Automation Features Included.| Private/Public Cloud on-premise, Consolidation. Highest Performance, Scalability for Mission Critical Workload.
|**Oracle Database**|World’s Best Database Runs Anywhere. User Builds and Operates Databases and Infrastructure. | Small to Big Database transactional need as well DWH needs. Customer Data Center, DIY model|

##### Autonomous Optimizations - Specialized by Workload 

|Autonomous Data Warehouse|Autonomous Transaction Processing |
|-------------------------|----------------------------------|
|Columnar Format|Row Format|
|Creates Data Summaries|Creates Indexes|
|Memory Speeds Joins, Aggs|Memory for Caching to Avoid IO|
|Statistics updated in real-time while preventing plan regressions | Statistics updated in real-time while preventing plan regressions |

##### Autonomous Database - Choice of Cloud Deployment 

![pic50](/images/pic50.png)

| | DBaaS VM or Bare Metal |Exadata Cloud Service or Cloud @Customer |Autonomous Serverless|Autonomous Dedicated |
|-| ---------------------- |-----------------------------------------|---------------------|-------------------- |
|**Management**| Customer| Customer| Oracle| Oracle|
|**Private Network**| Yes| Yes| No| Yes|
|**Single/Multi Tenant**| Single/Multi| Single/Multi| Singl|e Single/Multi|
|**Software Updates**| Customer Initiated| Customer Initiated| Automatic| Customer Policy Control|
|**Private Cloud**| No| Yes| No| Yes|
|**Offers Availability SLA**| No| 99.95%| SLO| SLO|
|**Database Versions**|11g,12c,18c,19c| 11g,12c,18c,19c| 18c| 19c|
|**Disaster Recovery**|Yes, Across ADs & Regions| Yes, Across ADs & Regions|No|No|
|**Hybrid DR**| Yes| Yes| No| No|
|**Consolidation**|Yes|Yes|No|Yes|

##### Autonomous Database Cloud Service – Deployment Options 

---
Oracle Autonomous Database can be deployed in 2 ways – dedicated and serverless.

Dedicated deployment is a deployment choice that enables you to provision autonomous databases into their own dedicated Exadata cloud infrastructure, instead of a shared infrastructure with other tenants.

With serverless deployment, the simplest configuration, you share the resources of an Exadata
cloud infrastructure. You can quickly get started with no minimum commitment, enjoying quick
database provisioning and independent scalability of compute and storage.

Both deployment options are available for Autonomous Transaction Processing and Autonomous Data Warehouse. 

---

#### Autonomous Database - Serverless 

Oracle automates end-to-end management of the autonomous database
* Provisioning new databases
* Growing/shrinking storage and/or compute
* Patching and upgrades
* Backup and recovery

Full lifecycle managed using the service console

* Alternatively, can be managed via command-line interface or REST API 

##### Automated Tuning in Autonomous Database 

* Load and go;
  * Define tables, load data, run queries
    * No tuning required
    * No special database expertise required
    * No need to worry about tablespaces, partitioning, compression, in-memory, indexes, parallel execution
  * Fast performance out of the box with zero tuning
  * Simple web-based monitoring console
  * Built-in resource-management plans

##### Autonomous Database – Fully-elastic 

* Size the database to the exact compute and storage required
  * Not constrained by fixed building blocks, no predefined shapes
* Scale the database on demand
  * Independently scale compute or storage
  * Resizing occurs instantly, fully online 
* Shut off idle compute to save money
  * Restart instantly
* Auto scaling:
  * Enable auto scaling to allow Autonomous Database to use more CPU and IO resources automatically when the workload requires it. 

##### Full Support of Database Ecosystem 

Autonomous Database service supports :
* Existing tools, running on-premises or in the cloud
  * Third-party BI tools
  * Third-party data-integration tools
  * Oracle BI and data-integration tools: BIEE, ODI, etc.
* Oracle cloud services: Analytics Cloud Service, GoldenGate Cloud Service, Integration Cloud Service, and others
* Connectivity via SQL*Net JDBC, ODBC 

##### Autonomous Data Warehouse: Architecture 

![pic51](/images/pic51.png)

##### Autonomous Transaction Processing: Architecture

![pic52](/images/pic52.png)

##### Getting Started with Autonomous Database

* Provisioning an ADB database requires only answers to 7 simple questions:
  * Database name?
  * Which data center (region)?
  * How many CPU cores?
  * How much storage capacity (in TBs)?
  * Admin password?
  * License Type?
  * Enable Auto scaling
* New service created in a few minutes (regardless of size)
* Database is open and ready for connections

##### Auto Scaling Autonomous Database

![pic53](/images/pic53.png)

*This picture shows how ADW service automatically scales OCPUs up when there is a demand for more computing power and then scales it down once the demand goes down.* 

* Auto scaling allows Autonomous Database to automatically increase the number of CPU cores by up to three times the assigned CPU core count value, depending on demand for processing.
* The auto scaling feature reduces the number of CPU cores when additional cores are not needed.
* You can enable or disable auto scaling at any time.
* For billing purposes, the database service determines the average number of CPUs used per hour.

##### Securing Autonomous Database (ADB)

* Stores all data in encrypted format in the Oracle Database. Only authenticated users and applications can access the data when they connect to the database.
* Database clients use SSL/TLS 1.2 encrypted and mutually authenticated connections. This ensures that there is no unauthorized access to the ADB Cloud and that communications between the client and server are fully encrypted and cannot be intercepted or altered.
* Certificate based authentication uses an encrypted key stored in a wallet on both the client (where the application is running) and the server (where your database service on the ADB Cloud is running). The key on the client must match the key on the server to make a
connection. A wallet contains a collection of files, including the key and other information
needed to connect to your database service in the ADB Cloud.
* You can specify IP addresses (or CIDR block) allowed to access the ADB using the access control list. This access control list will block all IP addresses that are not in the list from accessing the database.

##### Connecting to the Autonomous Database

![pic54](/images/pic54.png)

> Connecting to Autonomous Database Warehouse (ADW) or Autonomous Transaction Processing (ATP) from Public Internet

> Connecting to ADW or ATP (via NAT or Service Gateway) from a server running on a private subnet in OCI (in the same tenancy)

> Connecting to ADW or ATP from a server running on a public subnet in OCI (in the same tenancy)

##### Troubleshooting connectivity issues

![pic55](/images/pic55.png)

* Ensure that the Access Control List for the Autonomous Database (ADB) has the necessary entries for CIDR Block ranges and IP addresses, as your use case dictates.
* When connecting to ADB from a client computer behind a firewall, the firewall must permit the use of the port specified in the database connection when connecting to the servers in the connection. The default port number for Autonomous Data Warehouse is 1522 (find the port number in the connection string from the tnsnames.ora file in your credentials ZIP file). Your
firewall must allow access to servers within the .oraclecloud.com domain using (TCP) port 1522.
* When connecting to ADB from a server running on a private subnet (on the same OCI tenancy as the ADB), ensure that you have a service gateway or NAT gateway attached to the VCN. The route table for the subnet needs to have the appropriate routing rules for the service gateway or NAT gateway. The security lists for the subnet will need to have the right egress rules.
* For connections originating from a server running on a public subnet (on the same OCI tenancy as the ADB), ensure that route table and security lists are appropriately configured.

##### Scaling Your Database

* Scale your database on demand without tedious manual steps
  * Independently scale compute or storage
  * Resizing occurs instantly, fully online
  * Memory, IO bandwidth, concurrency scales linearly with CPU
  * Close your database to save money when not used
  * Restart instantly

##### Monitoring
* Service Console based monitoring
  * Simplified monitoring using the web-based service console.
  * Historical and real-time database and CPU utilization monitoring.
  * Real Time SQL Monitoring to monitor running and past SQL statements.
  * CPU allocation chart to view number of CPUs utilized by the service.
* Performance Hub based monitoring
  * Natively integrated in the OCI console and available via a single click from the ADB detail page
  * Active Session History (ASH) analytics
  * Real Time SQL monitoring

##### Autonomous Database (ADB) Cloud – Backup and recovery

* Autonomous Database Cloud automatically backs up your database for you. The retention period for backups is 60 days. You can restore and recover your database to any point-in-time in this retention period.
* Autonomous Database Cloud automatic backups provide weekly full backups and daily incremental backups.
* Manual backups for your ADB database is not needed.
  * But, you can do manual backups using the cloud console if you want to take backups before any major changes, for example before ETL processing, to make restore and recovery faster. The manual backups are put in your Cloud Object Storage bucket. When you initiate a point-in-time recovery Autonomous Database Cloud decides which backup to use for faster recovery.
* You can initiate recovery for your Autonomous Database using the cloud console. Autonomous Database Cloud automatically restores and recovers your database to the point-in-time you specify.
* Network Access Control Lists (ACL)s are stored in the database with other database metadata. If the database is restored to a point in time the network ACLs are reverted back to the list as of that point in time.

##### Autonomous Database Cloud – Cloning

* Autonomous Database provides cloning where you can choose to clone either the full database or only the database metadata.
* **Full Clone:** creates a new database with the source database’s data and metadata.
* **Metadata Clone:** creates a new database with the source database’s metadata without the data.
* When creating a Full Clone database, the minimum storage that you can specify is the source database’s actual used space rounded to the next TB.
* You can only clone an Autonomous Database instance to the same tenancy and the same region as the source database.
* During the provisioning for either a Full Clone or a Metadata Clone, the optimizer statistics are copied from the source database to the cloned database.
* The following applies for optimizer statistics for tables in a cloned database:
  * Full Clone: loads into tables behave the same as loading into a table with statistics already in place.
  * Metadata Clone: the first load into a table after the clone clears the statistics for that table and updates the statistics with the new load.

##### Pre-defined Services for Autonomous Data Warehouse

3 pre-defined database services identifiable as high, medium and low
* Choice of performance and concurrency for ADW
  * HIGH
    * Highest resources, lowest concurrency
    * Queries run in parallel
  * MEDIUM
    * Less resources, higher concurrency
    * Queries run in parallel
  * LOW
    * Least resources, highest concurrency
    * Queries run serially

> **Example for a database with 16 OCPUs**

| | No of concurrent queries|Max idle time | CPU shares|
|-| ------------------------|--------------|-----------|
|**HIGH**|3| 5 mins| 4|
|**MEDIUM**|20|5 mins|2|
|**LOW**|32|1 hour|1|
*When connecting for replication purposes, use the LOW database service name. For example, use this service with Oracle GoldenGate connections.*

##### Pre-defined Services for Autonomous Transaction Processing 

* Five pre-defined database services controlling priority and parallelism
* Different services defined for Transactions and Reporting/Batch

![pic56](/images/pic56.png)

##### Autonomous Database - Dedicated

**<span style="color:red">High Level Deployment Flow</span>**
![pic57](/images/pic57.png)

* The Autonomous Dedicated database service provides a private database cloud running on dedicated Exadata Infrastructure in the Public Cloud.
* It has multiple levels of isolation protects you from noisy or hostile neighbors.
* Customizable operational policies give you control of provisioning, software updates, availability and density.

**<span style="color:red">Physical Characteristics and constraints</span>**

* Quarter rack X7 Exadata Infrastructure
  * 2 severs( 92 OCPU, 1.44TB RAM)
  * 3 Storage Servers ( 76.8TB Flash, 107TB Disk)
* Cluster / Virtual Cloud Network
  * 1 Cluster per quarter rack
* Autonomous Container Database
  * Maximum of 4 per Cluster
* Autonomous Database
  * High Availability SLA – Maximum 100 DBs
  * Extreme Availability SLA – Maximum 25 DBs Physical Characteristics and constraints

**<span style="color:red">Security</span>**
* Databases always encrypted
* Reduced attack surface
* Automatic protection of customer data from Oracle operations staff
* Database Vault’s new Operations Control feature
* Oracle automatically applies security updates for the entire stack
* Quarterly, or off-cycle for high-impact security vulnerability
* Customer can separately use Database Vault for their own user data isolation

##### Summary

* Compare Autonomous Database (ADB) with DB System Cloud offerings in OCI
* Describe the features of Autonomous Data Warehouse Cloud - Serverless and Autonomous Data Warehouse Cloud - Dedicated, Autonomous Transaction Processing - Serverless and Autonomous Transaction Processing – Dedicated
* Describe how to deploy, use and manage ADB

##### Additional resources
* Autonomous Data Warehouse Service Documentation
https://docs.oracle.com/en/cloud/paas/autonomous-data-warehouse-cloud/
* Autonomous Transaction Processing Documentation
https://docs.oracle.com/en/cloud/paas/atp-cloud/index.html
* Autonomous Data Warehouse Cloud for Experienced Oracle Database Users
https://docs.oracle.com/en/cloud/paas/autonomous-database/adbsa/experienced-database-users.html#GUID-58EE6599-6DB4-4F8E-816D-0422377857E5
* Migrating Amazon Redshift to Autonomous Data Warehouse Cloud
https://docs.oracle.com/en/cloud/paas/autonomous-database/adbsa/index.html

**Exam Preparation Guide: OCI 2020 Architect Associate**
https://learn-prodfapap.oracle.com/education/downloads/1Z0-1072-20/OU_1Z0-1072-OCI-Architect-Associate-2020-Exam-Study-Guide.pdf

