# SAA-C03 Notes
> Notes created during preparation for SAA-C03 exam. Mostly based on Adrian Cantrill's [course](https://learn.cantrill.io).

#### Table of contents
- [1.0 Tech Fundamentals](#10-tech-fundamentals)
- [1.1 AWS Accounts](#11-aws-accounts)
- [1.2 AWS Fundamentals](#12-aws-fundamentals)
- [1.3 IAM, Accounts & Organisations](#13-iam-accounts-and-organisations)
- [1.4 S3 - Simple Storage Service](#14-simple-storage-service-s3)
- [1.5 VPC - Virtual Private Cloud Basics](#15-virtual-private-cloud-vpc-basics)
- [1.6 EC2 Basics](#16-elastic-compute-cloud-ec2-basics)
- [1.7 Containers & ECS](#17-containers-and-ecs)
- [1.8 Advanced EC2](#18-advanced-ec2)
- [1.9 Route 53 - Global DNS](#19-route-53---global-dns)
- [1.10 RDS](#110-relational-database-service---rds)
- [1.11 Network Storage & Data Lifecycle](#111-network-storage--data-lifecycle)
- [1.12 High Availability & Scaling](#112-ha--scaling)
- [1.13 Serverless & Application Services](#113-serverless-and-application-services)
- [1.14 Global Content Delivery & Optimization](#114-global-content-delivery-and-optimization)
- [1.15 Advanced VPC Networking](#115-advanced-vpc-networking)
- [1.16 Hybrid Environments & Migration](#116-hybrid-environments-and-migration)
- [1.17 Security, Deployment & Operations](#117-security-deployment--operations)
---

# 1.0 Tech Fundamentals 

## 1.0.1 OSI 7-layer networking model

### 1.0.1.1 Physical Layer
- Device on specific layer understands its layer and all below. Device on layer 4 understands 1,2,3,4. This is why HUB (L1) is stupid and Router (L3) is not.
- No uniquely identified devices.
- No Media Access Control (MAC).
- No device to device communication (e.g.HUB broadcasts everything on all ports).

### 1.0.1.2 Data-link Layer
- Ethernet is a L2 protocol used generally for local networks. 
- Higher layer devices build on lower layers adapting its capabilities.
- Layer 2 introduces Media Access Control (MAC).
- Layer 2 is using Layer 1 to transmit and receive raw data. Layer 2 uses **frames**, but Layer 1 doesn't understand them and just transmits the data.
- Switch is smarter HUB working on Layer 2 with support of MAC Address Table.
- CSMA/CD (Carrier Sense Multiple Access with Collision Detection). It checks if other side of communication is trying to send a packet. If carrier is detected, it won't send packet until it receives one. 

### 1.0.1.3 Network Layer
- L3 uses IP protocol to connect multiple local networks without point to point connection.
- IP use **packets** instead of frames like L2. Every packet just like frame has source and destination address (L2 - MAC, L3 - IP). 
    - Packets among others contain Protocol information from Layer 4 (UDP, TCP, ICMP) TTL (max no. of hops) and much more.
- There are protocols like BGP (Border Gateway Protocol) which allow routers to communicate with each other and exchange routes they know about.
- L4 Adds ARP which find the MAC for given IP Address
- Doesn't support any data channeling (Which is supported by usage of various Ports in Transport L4).

### 1.0.1.4 Transport Layer
- Layer 4 adds TCP and UDP protocols (and more).
- TCP uses Segments like Packets or Frames before them on different layers. They are encapsulated within IP Packets.
- Flags (SYN, ACK, FIN - handshake) are part of Segments.
- For correct transmission and retransmission incremental sequence numbers are used between client and server.

### 1.0.1.5 Session Layer
- In AWS, ACL is example of stateless firewall and Security Groups are statefull. 
    - Which means wether they can track the state of session or not. 
    - Stateless always has outbound and inbound rules
    - In statefull, allowing outbound implicitly allows inbound

### 1.0.1.6 Application Layer

### 1.0.1.7 Presentation Layer

## 1.0.2 Other Networking

### 1.0.2.1 Network Address Translation (NAT) *
- **STATIC NAT** - exactly one private to one (fixed, permanent) public IP address in both directions. For example. This is how AWS Internet Gateway works.
- **DYNAMIC NAT** - one private to 1st available IP address (temporary). Works based on pool of available IP addresses. Used for example when you have less public IP addresses than private addresses.
- **Port Address Translation (PAT) NAT** - many private to one public. Typical situation for home router. (AWS NAT Gateway)
    - PAT uses NAT Table and random public ports in order to connect to the target. This is why we use AWS NAT Gateway to provide connectivity for private subnets in AWS - external connection cannot be initiated without entry in the NAT table which is created only on external traffic thus allowing only egress connections.

- NAT is not needed for IPv6 (since NAT's idea is to overcome IPv4 shortages which is not a case with IPv6)

### 1.0.2.2 IP Address Space and Subnetting
- Private IP Address spaces:
    - A: 10.0.0.0
    - B: 172.16.0.0
    - C: 192.168.0.0
- Public IP Address spaces:
    - A: 0.0.0.0
    - B: 128.0.0.0
    - C: 192.0.0.0
    - D (multicast): 224.0.0.0
    - E (reserved): 240.0.0.0

### 1.0.2.3 VLANs, TRUNKS & QinQ
- VLANs offert separate broadcast domains.
- They work on Layer 2 Data-Link.
- They provide traffic isolation by VID (VLAN ID) included in the network frames.
- VLANs define two different ports
    - TRUNK port: Connection between two switches
    - ACCESS port: Configured to carry network traffic for a specific VLAN only

### 1.0.2.4 SSL & TLS
#### Connecting with server over HTTPS:
1. **Cipher Suites**: On client side, TLS establishes connection by sending ClientHello with supported cipher suites and server respond with ServerHello returning cyphers they support alongside the server's certificate that includes the public key. Both sides agree what cipher suite they are going to use. For now on, assymetric encryption is used from which we want to move to symmetric ASAP since it's computational heavy.
2. **Authentication**: Checking the server's certificate in CA. Verifying private key on server side by sending public key received in step 1. 
3. **Key exchange**: Client generates pre-master key and encrypts it with server's public key. Server then decrypts it using its private key. Next, server and client can generate their own Master Secret using same pre-master key they both have. Lastly, Master Secret is used over lifetime of connection to create manny Session Keys which are used to encrypt and decrypt data. 

### 1.0.2.5 Border Gateway Protocol (BGP)
- BGP always advertises the shortes path called ASPATH and it doesn't consider performance. Only path length matters.
- AS Path Prepending can be used to artificially lengthen shortest paths to favor the longer ones with better performance. For example, 2-hop fiber connection vs satelite connection.
- This is how routes are learned and communicated when using AWS Direct Connect or Dynamic VPN.

### 1.0.2.6 Stateful vs Stateless Firewall
- Stateless firewall requires full ephemeral port range for outbound connections.
- Because Stateless is 'stupid' it always requires two firewall rules for outbound and inbound.
- Stateful firewall is inteligent enough to recognize a response to a request.

### 1.0.2.7 Jumbo Frames
- Traditional Ethernet frames have a maximum transmission unit
    - Anything bigger than that, is Jumbo Frame with max size of 9000 bytes.
- Jumbo frames allow to send more data (payload) per frame so they are more efficient.
- More payload per frame = Less frames = less waste between frames = more efficient
- Each step must support same size of Jumbo Frames (for example one allows 9000 bytes and other 8500 like Transit Gateway). Otherwise it might result in fragmentation.

What in AWS does not support Jumbo Frames:
- Traffic outside of single VPC, 
- Traffic over an inter-region VPC peering connection, 
- Traffic over VPN connections,
- Traffic over an Internet Gateway,

What supports Jumbo Frames in AWS:
- Same region VPC peering,
- Direct Connect,
- Transit Gateway up to 8500 bytes

### 1.0.2.8 Application Level Firewall (L7)
- Firewals on L4 can understand IPs, Ports, etc. However, they see request and response as two separate connections. (stateless)
- Firewalls on L5 are statefull and they support session.
- Layer 7 Firewalls support and understand HTTP/S communication and can inspect/delete/replace/block/tag incoming data.
- Firewall accept the incoming request, terminate it and decrypt it. Next, creates its own encrypted connection to the server.

### 1.0.2.9 IPSEC VPN Fundamentals
- It is a group of security protocols.
- IPSEC sets up secure tunnels across insecure networks between two peers.
- Provides authentication and encryption.

## 1.0.3 Security

### 1.0.3.1 Envelope Encryption - AWS KMS
- KMS stores Key Encryption Keys which are managed by AWS.
    - They can encrypt data with less than 4KB in size which means they can only encrypt other encryption keys.
- KMS also stores Data Keys which are created by user but not managed by AWS.
    - When AWS KMS generates data keys, it returns a plaintext data key for immediate use and an encrypted (by Key Encryption Key) copy of the data key that can be safely stored with the data. When you are ready to decrypt the data, you first ask AWS KMS to decrypt the encrypted data key. It then returns decrypted key which can be used to decrypt the data.

### 1.0.3.2 Hardware Security Modules (HSMs)
- Keys stores in HSM never leave it.
- Accessed via tightly controlled APIs.
- For example, they might be use on Phone to store biometrics and protect them against malicious applications.

### 1.0.3.3 Digital Signatures
- Digital Signatures with the same idea like asymetric signing.
- Hashed documents can be signed for better authenticity and integrity.

## 1.0.4 DNS & DNSSEC

### 1.0.4.1 DNS
- **Zone** contains multiple nameservers.
- NS stands for 'nameserver,' and the nameserver record indicates which DNS server is authoritative for that domain (i.e. which server contains the actual DNS records). Basically, NS records tell the Internet where to go to find out a domain's IP address.
- **zone** is simply a portion of a domain. For example, the Domain Microsoft.com may contain all of the data for Microsoft.com, Marketing.microsoft.com and Development.microsoft.com. However, the zone Microsoft.com contains only information for Microsoft.com and references to the authoritative name servers for the subdomains. 
    - If there are no subdomains, then the zone and domain are essentially the same. In this case the zone contains all data for the domain.

#### Walking the DNS Tree (technical): 
- Root Zone NS points to TLD Zone NS (e.g., .com).  
    - TLD Zone NS points to the netflix.com zone NS
        - netflix.com zone NS contains the IP addresses for netflix.com and the IP subdomains like support.netflix.com
    - A zone is not tied to a single physical server. It is a portion of the DNS namespace managed by multiple authoritative DNS servers, often distributed globally, to ensure redundancy, reliability, and scalability.

#### **DNS tree path example:**

    Query to Root Zone Nameservers
        Query: Where are the nameservers for .com?
        Response: List of TLD nameservers for .com.

    Query to TLD Zone Nameservers (.com)
        Query: Where are the nameservers for netflix.com?
        Response: List of nameservers for netflix.com.

    Query to SLD Zone Nameservers (netflix.com)
        Query: Where is the A record for support.netflix.com?
        Response: IP address for support.netflix.com.

#### Registering a domain
- Domain Registrar: an entity responsible for **registring** the domain.
- DNS Hosting Provider: an entity responsible for hosting your zone.
- AWS Route53 or Hover acts as both Domain Registrar and Hosting Provider.

### 1.0.4.2 DNSSEC
- Doesn't change the results, just provide the information wether the response is valid or not.
#### Example
Attackers sometimes hijack traffic to internet endpoints such as web servers by intercepting DNS queries and returning their own IP addresses to DNS resolvers in place of the actual IP addresses for those endpoints. Users are then routed to the IP addresses provided by the attackers in the spoofed response, for example, to fake websites. 

You can protect your domain from this type of attack, known as DNS spoofing or a man-in-the-middle attack, by configuring Domain Name System Security Extensions (DNSSEC), a protocol for securing DNS traffic which introduces digital signing.  

## 1.0.5 Containers and Virtualization

### 1.0.5.1 Kubernetes
- Kubernetes Cluster is highly available cluster of compute resources.
    - **Cluster Control Plane** (brain): It manages scheduling, scaling, deploying.
        - It contains etcd, which is key-value storage. Basically a database used within a cluster.
        - **kube-scheduler**: Identifies any pods withing a cluster without assgined node and issigns a node based on resource requirements, affinity, anti-affinity or else.
        - **cloud-controller-manager**: provides cloud-specific control logic. It allows to link Kubernetes with cloud providers APIs.
    - **Nodes:** It actually runs applications. These are VMS or physical servers which function as a worker inside the cluster.
        - Each node contains kubelet which is an agent that interacts with Control Plane. Kubelet communicates with Control Plane using Kubernetes API (kup-apiserver). 
        - Each node also contains containerd or Docker software for running container operations. It is a container runtime.
        - **Pods** are smallest units of computing inside Kubernetes. Pods run and manage containers. One pod can store one or more containers. However, one-container-one-pod architecture is most common.
            - Service is something that runs on one or more pods.
        - **Kube proxy** is running on each node: it coordinates networking with the control plane.

## 1.0.6 Backups & DR

### 1.0.6.1 RPO and RTO
- RPO (Recovery Point Objective) - is how much data a business can lost expressed in time. In other words, maximum time during which business can lost data.
    - Technically, this is time between successfull backups.
- RTO (Recovery Time Objective) - maximum restore time that business can tolerate. From identification to final testing and handover. 
    - Can be reduced by proper monitoring, notifications and planning, spare hardware, more efficient systems.

## 1.0.7 Cloud Computing 1.0.1

### 1.0.8.1 Public vs Private vs Hybrid vs Multi cloud
- Hybrid Cloud != Hybrid environment (public vs on-premises)
    - Hybrid Cloud means connection of Private Cloud like AWS Outposts with Public Cloud

### 1.0.8.2 Cloud Service Models (IAA, PAAS, SAAS)
- IaaS: EC2
- PaaS: RDS
- SaaS: AWS lets you build them

# 1.1 AWS Accounts
- Root account can't be restricted or controlled by IAM. This is why it's always advised to create other users.
- One IAM user can have two active Access Keys.
- Access Key consists of Access Key ID and Secret Access Key which act like username and password. Secret Access Key can be only seen at creation and never be updated.
- Access Keys can be created, deleted, activated, unactivated.

# 1.2 AWS Fundamentals

## 1.2.1 AWS Global Infrastructure
- Edge locations are local distribution points. Enables placing data as close to customer as possible. 
- Regions contain full compute, storage and databases. Each region might have different data governance (US vs EU).

#### Service Resilience
- Globally Resilient: Spread across the globe with one database with data replicated across multiple or all regions. Can't choose a region for such serivce (IAM, R53). It would take a world end to get such service to failure.
- Region Resilient: One set of data in one region. If region fails, service fails. Availability Zones should be used here to enhance the availability.
- AZ Resilient: If AZ fails, the service fails. Very prone to failure.

## 1.2.2 Default VPC
#### Default VPC characteristics
- Only one **default** VPC per region with one public /20 subnet in each of that Region's AZ (In us-east-1 its 6 but ohio its 3). 
    - However, you can have multiple **custom** VPCs in one region.
- Default VPC always has the same CIDR assigned which is 172.31.0.0/16.
- Default VPC can be deleted and recreated but not modified.
- Default VPC comes with default Internet Gateway, defualt Security Group and default NACL.
- Default VPC assigns public IPs for services deployed to subnets within it and its connected to IG.

## 1.2.3 Elastic Cloud Compute - EC2
### 1.2.3.1 AMI
AMI (Amazon Machine Image) is an image used to build EC2 instances. Also, AMI can be built from such instances.

**What is stored on AMI**:
1. Data volume
2. Boot volume
3. AMI permissions (who can use)
    - Public permissions,
    - AWS Accounts specific,
    - Owner-only permissions
4. Block Device Mapping

**What is not stored on AMI**:
1. Network settings
2. Instance settings
### 1.2.3.2 Key pairs
- While creating a **key pair** for authentication, AWS returns private key. The public key is being loaded into an instance on the launch time (if was previously configured). Then user can SSH into the instance using private key.
    - To secure the private key, it is required to run chmod 400 command in order to restrict the access to the private key (grants read-only for the owner user). Without changing the permissions, other users on your local machine would be able to access the key. 
- For Windows and putty use .ppk key format. For Mac/Linux with OpenSSH use .pem
- SessionManager is a great connectivity alternative for SSH. Instead of using SSH keys, we can use this to connect to our instance. It requires special instance role.

## 1.2.4 S3 101
### 1.2.4.1 S3 characteristics
- S3 is global and public service but it's data is stored in specific region at rest.
    - S3 data placed in specific bucket never leaves its region unless you configure it to do so.
    - S3 is region resilient.
- Object in S3 is built of Key and Value. Key is like a filename and value is the actual content (data) which size can vary from 0 bytes to 5TB 
- S3 bucket name MUST be unique globally. Can't be formatted like IP address, 3-63 characters, all lower case, no underscores.
- Bucket is infinitely scalable. It can store unlimited objects with each sized up to 5TB. 
- S3 has flat structure. There is no such thing as folder, S3 uses prefixes that act like folders.
- There's limit of 100 buckets per account and hard limit of 1000 buckets (raised by support ticket). 
- S3 is an object store. Its not block storage because it can't be mounted like (its not logical disk). It's not file storage because it has flat structure.
- In order to delete bucket, you need to empty it.

## 1.2.5 CloudFormation (CFN) Basics
### 1.2.5.3 CloudFormation characteristics
- **[Tricky]** If you add FormatVersion to CloudFormation template, then Description section must always follow the template format version section.
- CloudFormation stack is a collection of AWS resources that you can create, update, or delete as a single unit. Stacks are defined using JSON or YAML templates. 
- Every time a Template is created or uploaded, it is being uploaded and stored in automatically created S3 bucket.
- Parameters section is responsible for values that are passed to the template at runtime (when stack is created or updated).

### 1.2.5.2 Resources and Stacks
- Resources specified in CloudFormation template are called **logical resources**.
- Resources created by CloudFormation template are called **physical resources**.
- CloudFormation creates a physical resource for every logical resource contained in the Stack.
- Stack contains all logical resources that the template tells it to contain.
    - It is a living representation of template.
    - One template can create 1, 10 or more stacks.
    - In general, Stack is created when you take template and tell CloudFormation to do something with it.

## 1.2.6 CloudWatch Basics
### 1.2.6.1 CloudWatch main features
1. Metrics,
    - Time ordered set of data points. CPU, Network usage etc. 
    - Alarms are connected to metrics. Billing alarm (i.e. zero-spend) is an example of this. You can setup alarm for CPU utilization going above given threshold and set up SNS or some actions for it.
2. Logs,
    - You know.
3. Events,
    - It can perform auto scalling or notifications through SNS based on configured events.

## 1.2.7 High-Availability vs Fault-Tolerance vs Disaster Recovery

#### High-Availability (HA)
- It's not about user experience or preventing some disruptions as they are acceptable - it's about maximizing online time of the system, keeping it operational, maximizing uptime.
    - Expressed in percentage of uptime.
    - Sometimes high-availability means having redundant services or infrastructures to be ready for replacing or rerouting.

#### Fault-Tolerance (DR)
- This one actually is actually about disruption preventing. In short, its operating through failure. 
    - Requires redundancy and also exceptional errors tolerance. Just like a plane can fly with one engine failure.
    - Harder to design, costs more, takes longer to implement.
    - High Availability is not enough for Fault-Tolerance.

#### Disaster Recovery (DR)
- Means, what to do when disaster happens. Set of policies, actions, rules and plans.
- Pre-planning part is important because it involves preparing the whole plan of recovery. Taking backups, preparing redundant services, planning the actions etc.

## 1.2.8 Route53 Fundamentals

### 1.2.8.1 Hosted Zones
- Hosted Zones are 'databases' that store DNS records.
    - Four nameservers are allocated to host one zone and each of them contains zone file.
        - Nameservers will be entered into the Domain's record within the top level domain zone.
    - Each hosted zone is billed.

### 1.2.8.2 DNS Record Types
- **A records**
    - contain the IP address of a domain,
    - domain => IP address v4
- **AAAA records**
    - contain the IP address of a domain,
    - domain => IP address v6
- **CNAME records**
    - can only point to other name like A record,
    - [ftp, mail, help] => site-helper.com,
    - used to reduce admin overhead
- **(NS) nameserver records**
    - from within top level domain, point to 
- **MX records**
    - Used by servers to locate mail server to use.
- **TXT records**
    - Used for proving domain ownership by adding some arbitrary TXT record to a nameserver.

- TTL values are used to indicate for how long DNS query should be cached in DNS resolver.
    - DNS resolvers should obey this TTL but this setting can be changed in resolver.

# 1.3 IAM, Accounts and Organisations

## 1.3.1 Identity Policies (IAM Policies)
### Identites types
1. IAM Users
2. IAM Roles
3. IAM Groups (not true identity)

### Policy types
1. **An inline policy** is a policy created for a single IAM identity. One-to-one relationship. They are deleted when you delete the identity. When you have 3 customers, you need to create and update 3 of them for each.
2. **Customer Managed policy** is a reusable policy. Once created, can be applied to many identities. Created and administered by user.
3. **AWS Managed policy** is the same as above but created and  administered by AWS.

### IMPORTANT! - Policy deicision hierarchy
Apply accross multiple Policies - User, Group and Resource policies
1. Explicit Deny
2. Explicit Allow
3. Default Deny (when nothing is set)

## 1.3.2 IAM Users and ARNs

### 1.3.2.1 How ARN is built
> arn:partition:service:region:account-id:resource-id

> arn:partition:service:region:account-id:resource-type/resource-id

### Key difference for S3 ARNs

> arn:aws:s3:::catgifs - **refers to bucket itself**

> arn:aws:s3:::catgifs/* - **refers not to the bucket itself but all the objects inside it**

### 1.3.2.2 IAM Users - key points
- Designed for single identifiable principal like a human or application.
- There can be maximum of 5,000 users per one AWS Account
- IAM user can be a member of 10 groups
    - Both of these points impact the scalability and system design
    - These issue is solved by IAM Roles and Identity Federation


## 1.3.3 IAM Groups
### Characteristics
- IAM Groups DONT SUPPORT NESTING!
- This is simply a container for IAM Users. 
- You can assign both managed and inline IAM Policies to groups.
    - You can't log onto them
    - You can't reference them from resource policies

## 1.3.4 IAM Roles
#### What assume means?
Assuming a role means asking Security Token Service (STS) to provide you with a set of temporary credentials -- role credentials -- that are specific to the role you want to assume. (Specifically, a new "session" with that role.)

### 1.3.4.1 How Role works - Trust Policy and Permission Policy
Role can have assigned **Trust Policy** and **Permission Policy**.

#### Trust Policy within an IAM role tells which Identites can assume given role and it can reference:
1. Identities in the same account like IAM Users, Roles and AWS Services like Lambda or EC2. 
2. It can also reference external web identites like facebook or google.
3. Or even Identities in other AWS Accounts.

#### Permission Policy tells what Identity can do as role. 

If a Role is assumed by something that is allowed to assume it, AWS generates temporary security credentials. These are short-term, time limited credentials generated by STS (Security Token Service). Once expired, identity must renew them by reassuming the role.

#### When to use IAM Roles?
- When there's a lot of users. Using Roles, thousands of identities can use the same role.
- When you want to levarage Identity Federation (Web Identities, Active Directory, Google)
- When you don't want to store/hardcode credentials on some AWS service like Lambda

#### Service-linked Roles & PassRole
Service-linked role:
- It is an IAM role linked to a specific AWS Service. 
- It contains predefines permissions by the service that it needs to interact with other AWS services.
- Service-linked role can't be deleted until its no longere required.

**“Passing”** a role refers to the process of linking an IAM role to a resource, which in turn dictates the actions the resource can perform on other AWS services.
**iam:PassRole** permission is required to pass a role.

## 1.3.5 AWS Organizations

### 1.3.5.1 Accounts in Organization
- Organizations are used to group multiple AWS Accounts into one organization.
- The account that creates organization, becomes **Management (Master) Account**
    - It has administrative control over the organization and can invite other AWS accounts to join the organization as member accounts.
- All the other accounts which are invited to the organization, become **Member Accounts**
- Instead of inviting already existing account, new AWS Account can be created by the management account. 
### 1.3.5.2 Billing
All member accounts are no longer billed separately. Instead, the management account consolidates all member account spendings, and only the management account receives the consolidated bill for the entire organization. It reduces financial overhead.
### 1.3.5.3 Organization structure
On top of the Organization, there's Root Container. Under that, we can create other sub organizational units and these organization units can be nested.
### 1.3.5.4 Role switching - IAM Roles
**OrganizationAccessAccountRole**: This role can be created in any AWS account within an organization. It allows users or services in other AWS accounts within the organization to assume this role. This setup enables users to log in to one AWS account and then switch roles to access other accounts within the organization seamlessly.

## 1.3.6 Service Control Policies (SCP)

### 1.3.6.1 Purpose and usage
Service Control Policies can be enabled and then applied to:
1. Organizational Unit
    - It works down the tree. If there are any sub-organizational units or accounts in the OU that we are applying the policy too - this policiy is applied to everything in it.
2. Single member Account

Once Service Control Policies are enabled, default policy is created and assigned to all members and Organizational Units in given Organization. This default policy explicitly allows all access to all AWS services with no explicit denys.

### Service Control Policy don't grant any permissions. 
They just define the limits what is possible and not within the AWS Account. Identites in the AWS Accounts still need to have permissions granted. 

**Example:** User A in Account B has assigned policy that allows him to only access the EC2 and S3 services. So even though SCP allows the whole account to use RDS let's say, a user A in Account B won't be able to access it.

### Limiting the AWS Root user
It doesn't happen directly! 
It 'limits' root user by limiting what the whole AWS Account can and cannot. 

## 1.3.7 CloudWatch Logs
#### Key Characteristics
- Log Stream = Sequence of log events from the same source for a specific thing.
- Log Groups = Containers for multiple Log Streams for the same type of logging 
    - Log Groups can be manually created
- Log Group stores configuration settings like retention and permissions. They apply to all log streams inside the group. 
- There's also something called Metric Filter which can filter log streams inside the group and create metric based on this

## 1.3.7 AWS CloudTrail

### 1.3.7.1 Type of Events
1. Management Events (free within free tier)
    - Logs events like adding a new policy, deleting a resource, adding member to organization.
2. Data Events (billed regularly)
    - Logs events related to accessing data within the resources. Operations on objects in S3, invoking Lambda, or other data-related activities.
3. Insight Events
    - **Unusual** account activity. Mostly important from security perspective.

### 1.3.7.2 Default CloudTrail
- CloudTrail is enabled by default. It is free and it collects management events only and it does not come with S3. It's only 90 days event history.
- Trails are how you configure S3 and CloudWatch logs
CloudTrail can send the events to specified S3 bucket or CloudWatch logs.

### 1.3.7.3 Creating Trail
Trail can be created to collect data for one region or more specified regions or all regions. This is important because some global services within AWS like STS, CloudFront and IAM send events only to us-east-1 so this must be taken under consideration. When AWS adds new region, it is automatically added to the configuration if you have all regions set for the trail.

A trail can be configured to use CloudWatch and S3 bucket to store the events. As long as you are within free tier of CloudTrail, you only pay for data stored in S3.

### CloudTrail is NOT real time
Data is being delivered with a delay of 15 minutes after account activity. It also publishes log files multiple times per hour.

## 1.3.8 AWS Control Tower

### 1.3.8.1 Landing Zone
Well-architectured multi account environment. Built with AWS Organizations, AWS Config and CloudFormation.

AWS Control Tower created two organizational units and two accounts by default:
1. Security OU
    - Inside, two AWS Accounts are created:
        1. Log Archive Account 
        2. Audit Account (CloudTrail & Config Logs)
2. Sandbox OU 
    - For tests/less rigid security, no account created here.

Landing Zone also provides monitoring and notification using CloudWatch and SNS. 

### 1.3.8.2 Guard Rails
These are rules for multi account governance. There are 3 types of these rules:
1. Mandatory
2. Strongly recommended
3. Elective

Guard Rails can function in two different ways. 

1. **Preventitive** - stop from doing things in AWS Accounts in landing zone. These are defined using Service Control Policies via AWS ORGs. **Example:** Disallow usage of specific AWS Regions or disallow bucket policy changes
2. **Detective** - perform compliance checks via AWS Config Rules. Used only for identification of such things, it won't stop them from happening. For example they can detect if CloudTrail is enabled in AWS Account or if EC2 has public Ipv4 assigned. These are also categorized in 3 ways:
    1. Clear
    2. In Violation
    3. Not Enabled


### 1.3.8.3 Account Factory
Account Factory enables automatic AWS Account creation with predefined settings, network and account configuration. Can be fully integrated with company's SDLC.

# 1.4 Simple Storage Service (S3)
## 1.4.1 S3 Security (Resource Policies & ACLs)

### S3 Bucket Policy (Resource Policy)
Resource policies are like Identity Policies but assigned to AWS resources rather than Identities. They control the access from resource perspective. 

### Principal and anonymous access
Opposed to Identity Policies they always have "Principal" defined in the Policy statements. This is how they can be recognized.

#### Example of allowing access to only your organization:

```
{
    "Version": "2012-10-17",
    "Statement": [{
        "Sid": "AllowGetObject",
        "Principal": {
            "AWS": "*"
        },
        "Effect": "Allow",
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::DOC-EXAMPLE-BUCKET/*",
        "Condition": {
            "StringEquals": {
                "aws:PrincipalOrgID": ["o-aa111bb222"]
            }
        }
    }]
}
```
#### Example of allowing public read-only access to your bucket:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::Bucket-Name/*"
            ]
        }
    ]
}
```
Thanks to this, we can allow full public access outside AWS for unathenticated principals which are not AWS identites.


### ACLs (Deprecated)
These are very limited and should not longer be used. AWS advises against using it unless you are 100% that something can't be done with resource policies.

## 1.4.2 S3 Static Website Hosting
This is a setting that can be enabled in S3 bucket management tab. Once enabled:
1. It provides access over HTTP (Normal access is via APIs)
2. Index and Error documents must be set. Index is an entry point.
3. Website static hosting endpoint is set by AWS. Used for HTTP access. Name is influenced by region and name of the bucket. It is automatically generated by AWS.
4. Custom domain set only via R53. To achieve this, S3 bucket name and domain name must match.
5. VERSIONING IS NOT REQUIRED!

## 1.4.3 Object Versioning and MFA Delete

### Object Versioning
- Once enabled, it can't be disabled. Only suspended.
    - To zero the costs, the only way is to delete the object and re-upload all objects to different non-versioned bucket.
- When you operate on specific object in versioned bucket, actions like delete are not permanent. If you are operating on a specific version, then it is permanent.
    - Objects in specific bucket are not identified only by the object key, but they are also given a unique ID.
    - When given object is 'deleted', a delete marker is created for the object and the object itself is hidden (still generating costs). In order to undelete the object, delete the delete marker.
    - To actually permanently delete the object, specify the version of the object to be removed.

### MFA Delete
- Can be enabled in versioned buckets.
- Once enabled:
    - MFA will be required to change bucket's versioning state
    - MFA will be required to delete versions
    - The actual MFA code will consist of MFA Serial Number + actual MFA code and it must be passed in the API calls
    - Hardware/Software

## 1.4.4 S3 performance optimization

### Direct Upload (single put upload)
- Very flow
- Should be used as long as multipart upload is not available yet to be enabled.
- Uses one stream to move data. Once it fails at 5GB of 6GB, the whole upload must be restarted
### Multipart upload
- Minimum size of data for multipart upload is 100MB.
- Multipart upload should be used as soon as it becomes available which is at 100 MB threshold.
- Parts can fail isolated and restart isolated. 
- In multipart upload, data is broken up maximally up to 10,000 parts 5MB to 5GB each.
- Multipart upload enables better bandwidth usage.

### S3 Transfer Acceleration
- S3 transfer acceleration uses AWS (to and from S3).
    - Instead of uploading data through regular network from hop to hop, it is being routed through the closest Edge Locations which are within AWS Global Network and move data directly to destination bucket.
- Bucket name can't contain dots and must be DNS compatible
- Buckets with enabled acceleration have their specific accelerated endpoint, example:

    ```cloudtrail-animals4life-45439085789s3-accelerate.amazonaws.com```
- Using Transfer Acceleration comes with additional fees.

## 1.4.5 Key Management Service (KMS)

### KMS Characteristics
- KMS is regional and public service used to create, store and manage keys (both symmetric and asymetric).
    - KMS is also capable of encryption operations (encryption, decryption & else).
- Created KEYS NEVER LEAVE KMS UNENCRYPTED - AWS KMS is designed so that no one, including AWS employees, can retrieve your plaintext keys from the service. Provides FIPS 140-2 (Level 2)
- KMS keys - main keys stored in KMS. Used for main cryptographic operations. Contains key ID, date, policy, description & state of key. 
    - KMS keys operates on data that is maximally 4KB in size. Which are basically other keys.
    - KMS Keys can have alias.
- Data is encrypted with key info, encoded into cyphertext so KMS knows which key to use to decrypt data. 
### Data Encryption Keys (DEK)
- DEKs are created by KMS keys
- Data Encryption Keys can work on files larger than 4KB
- DEK is being created with two versions -> Plaintext version is used to encrypt the data and is being discarded. Encrypted version is returned with data.
- Data Encryption Keys are never stored, they are discarded after use.
- KMS doesn't actually do encryption on data larger than 4KB. The service or user using it does.
- Encrypted version of Data Encryption Key is encrypted using KMS key that generated the Data Encryption Key
- Encrypted DEK can be later given to KMS to decrypt the key and then decrypt the data.

### Key Policies and Security
- Permissions for Encryption, CreateKey and Decryption are separate.
- Services using KMS like S3 use different Data Encryption Key for every single object.
- Keys can be customer or AWS owned.
- KMS Keys must be explicitly told they trust AWS Account they are contained within.
- KMS uses Key Policies which are type of Resource Policy and every KMS Key has one. When customer-managed keys are used, they can be changed.

### Example commands for encryption and decryption

```
file: battleplans.txt
aws kms encrypt \
    --key-id alias/amazingkey \
    --plaintext fileb://battleplans.txt \
    --output text \
    --query CiphertextBlob \
    | base64 --decode > not_battleplans.enc 
    
# For decryption, we never pass the key as it is stored with encrypted data.   

aws kms decrypt \
    --ciphertext-blob fileb://not_battleplans.enc \
    --output text \
    --query Plaintext | base64 --decode > decryptedplans.txt
```

## 1.4.6 S3 Object Encryption (SSE - Server Side Encryption)

### Server Side Encryption types
SSE on S3 objects is now mandatory. The only thing a user can influence is choosing the type of SSE and also what version of SSE is utilized.
### 1. **SSE-C** Server Side Encryption with Customer-Provided Keys 
- Customer handles the keys. Once S3 receives plaintext data alongside the key, it encrypts the object, generates and store hash of this key with encrypted object and discards it. 
- When there's an attempt to decrypt the object, the same key must be provided and S3 will check the hash against the prvovided key.

#### Advantage:
- S3 nevers sees plaintext data.
### 2. **SSE-S3 (AES-256)** Server Side Encryption with Amazon S3-Managed Keys (default)
- Amazon takes care of encryption, decryption and managing the keys. User just send plaintext data. No options changing, handled end-to-end by S3.
- S3 generates new key for each object which is used for encryption of that object.
    - S3 uses S3 Key to encrypt the object key and then it stores it and discards the original one.
    - S3 takes care of rotation.

#### Disadvantages:
- Almost no controll over the process. 
- No Role Separation
    - **For example**: When there's an admin with full S3 access, we can't restrict him from accessing, opening and reading the files stored within the bucket. 

### 3. **SSE-KMS** Server-Side Encryption with KMS Keys stored in AWS Keys Management Service
- Enables high control over keys, especially with customer-managed keys. These have key-level permissions.
- This way S3 admins can administrate the S3 and buckets but won't be able to access them.

## 1.4.7 S3 Bucket Keys (For SSE-KMS)
Normally, when we use KMS for SSE encryption, each object has its uniqute Data Encryption Key generated by KMS. It means that each object = one request to KMS and theres a fee for it.

With Bucket Keys, KMS can generate a time limited Bucket Key for given bucket that generated DEKs from within the bucket thus reducing the number of requests sent to KMS.

#### After enabling it:
- CloudTrail KMS events now show bucket not the object
- Works with replication, means when encrypted object is being replicated to other S3 Bucket it retains the same encryption settings.
    - When unencrypted object is being uploaded, replicated object uses encryption settings of destined bucket.

## 1.4.8 S3 Object Object Storage Classes

### 1. S3 Standard
- Replicated all over 3 Availability Zones,
- Used for **frequently** accessed objects,
- For data that is important and non replacable
#### Billing
- per GB/m of stored data (you pay only for duration you store object),
- per GB for transfering data out,
- price per 1,000 requests

### 2. S3 Standard-IA
- Replicated all over 3 Availability Zones,
- Used for **infrequently** accessed objects,
- Used for important and non-replaceable data,
- Not for short term storage because of minimum charges,
- Not for very small objects because of minimum charges
#### Billing
- in addition to transfer fee, theres a per GB retrieval fee,
- per GB for transfering data out,
- price per 1,000 requests
- minimum duration of 30 days, objects can be stored for less but the **minimum billing always** applies,
- minimum capacity charge of 128KB per object. Objects can be smaller, but **minimum billing always applies**.

### 3. S3 One-Zone IA
- Replicated only to one Availability Zone
    - Durability remains as high as in standard or previous classes as long as AZ doesn't fail
- For long-lived data,
- For non-critical data,
- For data that can be easily replaced,
- Not for short term storage because of minimum charges,
- Not for very small objects because of minimum charges

#### Billing
- in addition to transfer fee, theres a per GB retrieval fee,
- per GB for transfering data out,
- price per 1,000 requests
- minimum duration of 30 days, objects can be stored for less but the **minimum billing always** applies,
- minimum capacity charge of 128KB per object. Objects can be smaller, but **minimum billing always applies**.

### 4. S3 Glacier - Instant
- Still provides instant access,
- For infrequent access, lets every quarter due to minimum storage time,

#### Billing
- minimum duration of 90 days, objects can be stored for less but the **minimum billing always** applies,
- minimum capacity charge of 128KB per object. Objects can be smaller, but **minimum billing always applies**.
- in addition to transfer fee, theres a per GB retrieval fee,

### 5. S3 Glacier - Flexible
- Cold state data,
- Replicated all over 3 Availability Zones,
- Designed for archive data,
- When frequent/realtime access isn't needed,

#### Retrieval process
- Objects can no longer be publicly available,
- Retrieval process is required to get access to objects,
    - Temporarily, these retrieved objects are stored in S3 Standard-IA. After accessing, they are removed.
- Retrieval types:
    1. Expedited (1-5 minutes)
    2. Standard (3-5 hours)
    3. Bulk (5-12 hours)

#### Billing
- minimum duration of 90 days, objects can be stored for less but the **minimum billing always** applies,
- minimum capacity charge of 40KB per object. Objects can be smaller, but **minimum billing always applies**.

### 5. S3 Glacier - Deep Archive
- Frozen state data,
- Replicated all over 3 Availability Zones,
- Good for legal or regulation data access,
- For very infrequent access, rarely if ever accessed

#### Billing
- minimum duration of 90 days, objects can be stored for less but the **minimum billing always** applies,
- minimum capacity charge of 40KB per object. Objects can be smaller, but **minimum billing always applies**.

#### Retrieval process
- Objects can no longer be publicly available,
- Retrieval process is required to get access to objects,
    - Temporarily, these retrieved objects are stored in S3 Standard-IA. After accessing, they are removed.
- Retrieval types:
    1. Standard (12 hours)
    2. Bulk (up to 48 hours)

### 6. S3 Inteligent Tiering
- For changing and unknown patterns,
- For long-lived data,
- Objects automatically moved between classes based on access frequency
- Objects are being monitored,
- Accessed object are moved back to Frequent Access class
- Objects smaller than 128 KB are not eligible for auto tiering,
    - These smaller objects may be stored, but they’ll always be charged at the Frequent Access tier rates and don’t incur the monitoring and automation charge

#### Classess tree
1. **Frequent Access** (Like S3 Standard)
    - accessed regularly,
    - no application side changes required
2. **Infrequent Access** (Like S3 Standard-IA)
    - moved here if not accessed by more than 30 days
    - no application side changes required,
3. **Archive Instant Access** (Like S3 Glacier Instant Retrieval)
    - moved here if not accessed by more than 90 days
    - no application side changes required,
4. (optional) **Archive Access** (Like S3 Glacier Flexible Retrieval)
    - 90 > 270 days
    - asynchronous access patterns required on application side required due to retrieval process
    - moving it back up the tree takes more time,
5. (optional) **Deep Archive** (Like Glacier Deep Archive)
    - 180 - 730 days
    - asynchronous access patterns required on application side required due to retrieval process,
    - moving it back up the tree takes more time,
#### Billing
- No retrieval cost,
- Monitoring and automation cost per 1,000 objects instead of retrieval cost,
    - No additional tiering charges apply when objects are moved between access tiers
- Billed exactly like in specific class

## 1.4.9 S3 Lifecycle Configurations
### Key Characteristics
- S3 One Zone-IA can't transition into Glacier Instant Retrieval
- Minimum if 30 days before transition from S3 Standard to Standard-IA or One Zone-IA
- A single rule cannot transition into Standard-IA or One Zone-IA and then to Glacier classes within 30 days - theres a minimum period of 30 days to hop between classes.

## 1.4.10 S3 Replication
SRR - Same Region Replication
CRR - Cross-Region Replication

### Key Characteristics
- Replication Time Control (RTC) - adds SLA of 15 minutes for replication. It comes with additional cost.
- **By default, replication is not retroactive (doesnt work for previously uploaded objects).**
    - But during creation of the Replication Policy, we are asked if we want to move already existing objects to the destination bucket.
- **For replication, versioning needs to be ON**,
- By default, replication of delete markers is not enabled. However, this setting can be changed.

Replication requires an IAM role that will be assumed by S3 to replicate the object to destination bucket.
The following example shows a trust policy, where you identify Amazon S3 as the service principal who can assume the role.

```
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect":"Allow",
         "Principal":{
            "Service":"s3.amazonaws.com"
         },
         "Action":"sts:AssumeRole"
      }
   ]
}
```

### Use cases
- SRR - Log Aggregation 
- SRR - PROD and TEST sync 
- SRR - Resilience with strict sovereignty 
- CRR - Global Resilience improvements
- CRR - Latency Reduction (move objects closer to users)

## 1.4.11 S3 PreSigned URLs 
### Key Characteristics
- You can generate a presigned URL to object that  you have no access to.
    - Person using the URL won't be able to access it either
    - Even URL for non existent object can be created
- URL holds permissions of identity that generated it
    - When accessing the object, always current permissions are being checked.
- Presigned URLs should be generated using long term identities. So don't generate them with a role but with an IAM User

AWS CLI command example:
```aws s3 presign s3://animals4lifemedia44444/aotm.jpg --expires-in 60```

## 1.4.12 S3 Select and Glacier Select 
### Key Characteristics
- Allows to retrieve parts of objects using SQL-like expresssions
- Very usefull tool when application accesses huge objects like for example 2 TB in size
- S3 Select can query only one object per request.
- Supports CSV, JSON, Parquet

### Restrictions
You cannot query an object stored in the S3 Glacier Flexible Retrieval, S3 Glacier Deep Archive, or Reduced Redundancy Storage (RRS) storage classes. You also cannot query an object stored in the S3 Intelligent-Tiering Archive Access tier or the S3 Intelligent-Tiering Deep Archive Access tier

### Without Select
- Application retrieves 2TB object, and pays for 2TB of data retrieval
- Filtering of data is done on application side so even though most of data is discarded, you still pay for the whole object

### With Select
- Using S3/Glacier Select, we are retrieving only parts of data specified by the SQL-like expression
- Filtering is done on S3 side so we don't pay for the whole object, but only for the part that we retrieved
- Up to 400% faster and 80% cheaper

## 1.4.13 S3 Events
### Key Characteristics
- S3 provides notifications on events occuring in bucket

Supported events:
- Object Created (Put, Post, Copy, CompleteMultiPartUpload)
- Object Delete (Delete, DeleteMarkerCreated)
- Object Restore(Post(Initiated), Completed)
- Replication (OperationMissedThreshold, OperationFailedReplication)

Events can be sent to:
1. Lambda
2. SNS
3. SQS Queue
- Each of these must have policy set to allow S3 interact with them

### Alternative
**EventBridge** is modern alternative. Probably should be a default option when it comes to events. Supports more AWS Services and more types of events.

## 1.4.14 S3 Access Logs
### Key Characteristics
- Allow tracking of bucket and object access
- Not real time (best effort delivery)
    - Might even take up to few hours to deliver logs to destination bucket
- Log files consist of log records
- One target bucket can be used for mutliple source buckets
- Lifecycle of the logs must be personally managed by the user

### S3 Log Delivery Group
- It manages the process
- It takes care of reading the config for logs gathering defined in the bucket
- It's set on the source bucket
- Target bucket to which we send access logs, must allow Log Delivery Group in its ACL 

## 1.4.15 S3 Object Lock
### Key Characteristics
- When object lock is enabled, so does versioning. Individual versions are locked.
- Once object lock is enabled on created bucket, it can't be disabled, and versioning can't be suspended.
- Object lock can be enabled on existing as well as newly created buckets.
- Object Lock works in 2 different ways: Retention Period and Legal Hold. Bucket can use both, one or the other or none

### Retention Period
- Retention period is defined in days & years

### Legal Hold
- Legal hold provides the same protection as a retention period, but it has no expiration date. Instead, a legal hold remains in place until you explicitly remove it. Legal holds are independent from retention periods and are placed on individual object versions.

#### 1. COMPLIANCE mode
- Locked object can't be adjusted, deleted or overwritten EVEN by the account root user until retention expires
- Used for compliance data like medical or legal documents.

#### 2. GOVERNANCE mode
- Locked object can be adjusted provided the identity has special permissions set for it.
- Used for preventing accidental deletion
- Might be used for tests before using compliance mode
> Permission: s3:BypassGovernanceRetention

> Request Header: x-amz-bypass-governance-retention:true  # Console default

### Retention Period
- No retention period. Just ON/OFF switch.
- No deletes or changes untill switched OFF
- Special permission required to use this feature on object:
    > 3:PutObjectLegalHold
- Used to prevent accidental deletions
- For actual legal situation when specific object needs to be flagged for specific case or project

## 1.4.16 S3 Access Points
### Key Characteristics
- Simplify managing access to S3 Buckets/Objects
- An access point is associated with exactly one S3 bucket
- Rather than having one bucket with one resource policy, you can have multiple access points each with its own policy and network access controls
- Each access point has its own endpoint address (DNS name)
- Access Points can be created via console or AWS CLI using this command:
    > aws s3control create-access-point --name secretcats --account-id 23423432 --bucket catpics
- You can't add buckets  to access point once its created

### Why use access points?
- You can provide access to only specific part of buckets or whole but with restrictions
- Instead of managing one Bucket Policy, we can create multiple Access Points that point at specific parts of one bucket. Each Access Points will have it's own Policy and network access controls.

### Multi-Region Access POints
- There's consistency lag with Multi Region Access Points. When object is uploaded to bucket A, it needs some time to be replicated to bucket B. But when somebody is redirected by Access Points to bucket B because its closer it might not be replicated and available yet.

# 1.5 Virtual Private Cloud (VPC) Basics

## 1.5.1 VPC Sizing and Structure

### VPC Considerations
- What size should the VPC be?
- Are there any networks we can't use?
    - VPC's, Cloud, On-permises, Partners & Vendors that we interact with - all that must be taken under consideration
- Try to predict the future. Scaling, business growth, spare.
- Architecture tiers & resiliency zones (availability zones)

### Custom VPC
- Default Tenancy 
    - Lets you later pick the tenancy per service. 
- Dedicated Tenancy 
    - You are locking yourself and all other future AWS services to this option.
- AWS VPC main private CIDR block must be minimum /28 (16 IP) and maximum /16 (65,536 IP).
- Optional single assigned IPv6 /56 CIDR block allowed.
- Each VPC has its own DHCP options set linked to it. You can't edit them but you can create new and allocate it to VPC.
    - Auto assign IPv4 and IPv6
- **enableDnsHostnames** - (deisabled by default) provide public DNS names for services inside VPC that have public IP assigned
- **enableDnsSupport** - (enabled by default) wether DNS resolution is enabled or disabled in VPC at the DNS IP address which is VPC Private IP + 2. 

## 1.5.2 VPC Subnets
- In AWS diagrams, blue subnet is private and green is public.
- Subnet can never be in more than one Availability Zone.
- Availability Zones can hold multiple subnets.

### Reserved IP addresses
Lets assume 10.16.16.0/20 Network

Reserved IP addresses are going to be:
- Network Address - 10.16.16.0
- 'Network +1' - 10.16.16.1 - VPC Router
- 'Network +2' - 10.16.16.2 - DNS 
- 'Network +3' - 10.16.16.3 - Reserved for future use
- Broadcast Address - 10.16.16.255 - Last IP in subnet

## 1.5.3 VPC Routing and Internet Gateway
When instances in subnet have allocated IPv4, this IPv4 address is not actually assigned and stored at the instance level. Instead, Internet Gateway stores and links it to the instance. The instance have private IP only, you cannot assign IPv4 directly to it, IPv4 never touches the instance. This is also why if you check network interfaces on EC2 instance for example, you won't see the public IP address. One private IP to one public IP.

When it comes to IPv6, the IP address is actually stored on the instance.

### VPC Routing
- **Destination:** the packet's final destination
- **Target:** where the packet should go next, to get it one step closer to the intended destination.

    Example: Destination: 0.0.0.0 and Target: Internet Gateway - Here all traffic will be passed on to the Internet Gateway

### Internet Gateway
- Internet Gateway is region resilient and it's always attached to VPC.
- Only one can be attached to VPC
- It runs from AWS public zone
- Gateway traffic between Internet or AWS Public Zone (S3, SQS, SNS)
- Managed, AWS handles the performance

### Bastion Host / Jumpbox
- Bastion Host = Jumpbox
- An instance in public subnet
- Incoming management connections arrive here and then access internal VPC resources
- Often, it was the only way IN to a VPC
- NAT is outbound from your instance. Bastion is inbound to your instance

## 1.5.4 Stateful vs Stateless firewalls

### Stateful
- Smart enough to identify the response for the given request. Only one rule required.

### Stateless
- Does not differentiate request and response (for it these are two completely different requests). Needs two rules for inound and outbound connections. 

## 1.5.5 Network Access Control Lists (NACLs)
- STATELES (Both Inbound and Outbound rules required)
- NACLs operate on Subnet level
- Each subnet has its NACL
    - Doesn't mean its created for each, but each subnet must have associated one NACL. 
    - One NACL can be associated with many subnets.
- They inspect all outbound and inbound traffic which is crossing subnet boundary
    - IPs/CIDR, ports & protocols - can't reference logical resources
- They support explicit allows and explicit denys
- Rules are processed in order. Starting from lowest rule number to highest. If matched, processing stops.

### Default NACL
- Default NACL is created during VPC creation
- By default, it allows all inbound and outbound traffic. Very beginner friendly, basically remains inactive untill changed.

### Custom NACL
- Can be created manually and associated with subnets
- Blocks whole traffic by default after creation!

## 1.5.6 Security Groups (SG)
- STATEFUL
- No explicit denys - can't block bad actors
    - only explicit allows. What is not allowed is implicilty denied.
- Support referencing logical resources
    - They can reference itself. Useful when mutliple instances have the same security group assigned.
- In theory Security Group is assigned to an instance. But in reality, it is assigned to ENI (Elastic Network Interface) of that instance.
    - ENI is a logical networking component in a VPC that represents a virtual network card. It can contain example following attributes:
        1. A primary private IPv4 and IPv6 address from the address range of your VPC
        2. One public IPv4 address
- Maximum 5 per network interface

## 1.5.7 NAT and NAT Gateway 
### Key Characteristics
- Runs from public subnet and requires public IP
- AZ resilient
- Charged per hour and per GB
- For region resilience - deploy NATGW in each AZ used, with route table in each pointing at that NATGW as target
- Fully managed, scales up to 45 Gbps
- NAT isn't required for IPv6
    - NAT Gateways don't work with IPv6

# 1.6 Elastic Compute Cloud (EC2) Basics

## 1.6.1 Virtualization 101
### Methods of virtualization
- Emulated Virtualization
    - Between guest OS and host OS there was just a supervisor (privilieged software) that was translating all the privileged calls made to the hardware by guest OS. It was called binary transaltion and it was very, VERY slow.
    - Guest OS's were constantly trying to do privileged calls, do reads, writes, doing calls to CPU - they were unaware they they are being virtualized.
- Paravirtualization
    - This method was faster and better because the guest OS images (source code) were changed in some parts and were prepared for virtualization. Guest OS's were no longer trying to make privileged calls, instead they were using hypercalls, calls directly to hypervisor.
    - Not many supported images and operating systems
- Hardware Assisted Virtualization
    - Here, physical hardware was becoming aware of virtualization. CPU itself was aware of performing virtualization. 
        - CPU was expecting privileged calls and it was blocking them and redirecting to the hypervisor.
    - Very little performance degradation.
    - Still problems with I/O. Network and disk operations.
- SR-IOV
    - More hardware devices beecome virtualization aware.
    - Network Card being aware of virtualization can present itself as multiple mini cards which are given for guest OS's to use as real cards. No translation needed thanks to this.
    - Network Cards which support SR-IOV are required.
    - In EC2 this is called Enhanced Networking.

## 1.6.2 EC2 Architecture and Resilience

### Key characteristics
- EC2 instances run on EC2 hosts. These can be shared or dedicated.
    - Shared (default) - clients using it are fully separated. Customer pay only for the instance that its using.
    - Dedicated - client has the host on its own and is paying for the entire host. 
- EC2 Hosts are tied to 1 AZ
    - AZ Fails -> Host Fails -> Instance fail

### How EC2 Hosts are built 

EC2 hosts contains multiple pieces of hardware besides CPU:
- Instance Store (ephemeral).
    - When instance is moved from one host to another, the instance storage is gone.
- Data Network. 
    - ENIs attached to instances in subnets are mapped to this physical hardware running on EC2 Hosts.
- Storage
    - EC2 Hosts can connect to remote storage like EBS which is AZ resilient. No cross-zone access.

### Most common usage
- Traditional OS+Application compute
- Long-running compute
- Server style applications
- Burst or steady-state loads
- Monolithic application stacks
- Migrated application workloads (that expect traditional virtualized system) or Disaster Recovery


## 1.6.3 EC2 Instance Types

### Main categories
1. General-purpose
    - Default, diverse workloads, equal resource ratio
2. Compute optimized
    - For 3D graphics, gaming, media processing, machine learning
3. Memory optimized
    - For processing large in-memory sets, some database workloads
4. Accelerated computing
    - Very niche. Hardware GPU, FPGAs
5. Storage optimized
    - For data warehousing, elasticsearch, for sequential and random i/o, analytics workloads

### EC2 Instance Types

#### Instance type encoding - R5dn.8xlarge
- R5dn.8xlarge - Instance type
- R = Instance Family
- 5 = Generation
- dn = Additional capabilities (**a** - AMD cpu, **d** - network optimized)
- 8xlarge - size of the instance

#### Good web references
1. https://aws.amazon.com/ec2/instance-types/
2. https://instances.vantage.sh/

#### Instance Types memorizing
- **G** for GPU
- **C** for COMPUTE
- **I** for I/O
- **D** for Dense Storage
- **P** for Parallel Processing
- **R** for RAM
- **X** for Large Instances

## 1.6.4 EC2 Connection Methods
### Key Pair
- Using assymetric key pair. Public key is being uploaded at the creation of the instance and then we connect to it using our private key.
    - Quite problematic from admin and security perspective when multiple users must to connect.
    - It requires allow rule in Security Group.

### EC2 Instance Connect
- AWS permissions are used for connection. No need to provide the keys or credentials.
- Connection doesn't originate from local user's machine. The connection goes through AWS and AWS connects to the instance using Instance Connect service.
    - When only user's IP is allowed in inbound security group rule, Key Pair connection will work but Instance Connect won't.
    - IP range for Instance Connect service can be added and Instance Connect is going to work. Check: https://ip-ranges.amazonaws.com/ip-ranges.json

## 1.6.5 Storage
- Direct - local, attached storage - Storage on the EC2 Host
- Network attached - Volumes delivered over network like EBS
- Ephemeral - Temporary Storage (like instance storage)
- Persistent - Permanent storage, lives on past the lifetime of the instance


### Types of storage
1. Block
- Typical volume that can be mounted and is bootable. Presented to the OS as a collection of blocks. (OS decides what to do with them)
- EBS
2. File
- Presented as file share and has structure. Mountable, not bootable.
- EFS
3. Object
- Infinitly scalable. Just for storing objects. Not mountable, not bootable. 
- S3

### Storage Performance
1. Throughput - how much data can be transferred per second. Generally expressed in MBps 
2. IO Size - size of blocks of data that are being written to disk. 
3. IOPS - how many operations can be done in given second

> IO Size x IOPS = Throughput | 16KB x 100IOPS = 1.6MB/s

## 1.6.6 EBS Volumes
- EBS presents allocation of physical disks as volume. They can be written to or read from. 
    - Volumes can be unencrypted or even encrypted with KMS
- Operate in one AZ (AZ resilient). If AZ fails, EBS volumes fail.
- Volumes can be attached, deattached and reattached over storage network.
- One EC2 instance can have multiple ebs volumes attached to it.
- Is some special cases, one EBS volume can be attached to multiple instances.
- Billed based on GB/month.
- Provide different physical disk types like hdd, ssd etc. and different performance profiles.

#### Migrating data between AZs
EBS is AZ resilient. S3 is region resilient. EBS Snapshot can be uploaded to S3, and from that snapshot an EBS volume can be created in a different availability zone.

### 1.6.6.1 EBS - General Purpose SSD volumes
Medium-sized, single-instance databases, Dev and test environments, Boot volumes

#### GP2 
- **Volume size:** 1GB - 16GB
- **Max IOPS:** 16,000 (64 KB I/O)
- **Max throughput per volume:** 250 MB/s

#### GP3
- **Volume size:** 1GB - 16GB
- **Max IOPS:** 16,000 (16 KB I/O)
- **Max throughout per volume:** 1,000 MB/s

### 1.6.6.2 EBS - Provisioned IOPS SSD Volumes
#### io1
Workloads that require sustained IOPS or I/O-intensive database workloads
- **Volume size** 4GB - 16TB 
- **Max IOPS:** 64,000 (16 KB I/O)
- **Max throughput per volume:** 1,000 MB/s
    - To achieve maximum throughput of 1,000 MiB/s, the volume must be provisioned with 64,000 IOPS and it must be attached to an instances built on the Nitro System.

Amazon EBS Multi-Attach enables you to attach a single Provisioned IOPS SSD ( io1 or io2 ) volume to multiple instances that are in the same Availability Zone

#### io2
For sub-millisecond latency, sustained IOPS performance. 

- **Volume size** 4GB - 64TB
    - Volumes over 16 TB in size can only be attached to instances built on the Nitro System.
- **Max IOPS:** 256,000 (16 KB I/O)
    -  Volumes over 64,000 IOPS can only be attached to instances built on the Nitro System. Volumes up to 64,000 IOPS can be attached to non-Nitro instances, but they can only achieve up to 32,000 IOPS.
- **Max throughput per volume:** 4,000 MB/s

Amazon EBS Multi-Attach enables you to attach a single Provisioned IOPS SSD ( io1 or io2 ) volume to multiple instances that are in the same Availability Zone

### 1.6.6.3 EBS - HDD-Based

#### st1
A low-cost HDD designed for frequently accessed, throughput-intensive workloads. Designed for big data, warehouses, log processing.
- **Max volume size:** 125GB - 16TB
- **Max IOPS:** 500 IOPS (500 MB/s)

####  sc1
The lowest-cost HDD design for less frequently accessed workloads. Designed for colder data requiring fewer scans per day.
- **Max volume size:** 125GB - 16TB
- **Max IOPS:** 250 IOPS (250 MB/s)

## 1.6.7 Instance Store Volumes
#### Naming
The virtual devices for instance store volumes are ephemeral[0-23]. Instance types that support one instance store volume have ephemeral0. Instance types that support two instance store volumes have ephemeral0 and ephemeral1, and so on.

#### Key Characteristics

- Block storage
- Physically connected to one EC2 Host
    - Instances on that Host can access them
- Highest storage performance (because physically attached). More IOPS and throughput vs EBS
- Included in the EC2 Instance price. 
- Attached at launch only. Impossible to add another later on.
- Instance is not moved between hosts when instance is rebooted, only when stoped and started. So the volume is not cleared neither.
- **Ephemeral**. Data is lost when moving Instance from one Host to another.
    - When Instance is stopped and started.
    - When there's Host maintenance and instances are moved to different Host.
- Data is also lost when instance is resized or on hardware failure.

#### Different instance types have different Instance Store volumes
- D3 = 4.6 GB/s throughput
- i3 = 16 GB/s of sequential throughput

## 1.6.8 Storage summary (decisions)
- Cheap = st1 or sc1
- Throughput, streaming = st1
- Boot = not st1 or sc1
- GP2/3 = up to 16,000 IOPS
- IO1/2 = up to 64,000 IOPS (*256,000)
- RAID0 + EBS up to 260,000 IOPS
- More than 260,000 IOPS - Instance Store (if you can accept tradeoffs)

## 1.6.9 EBS Snapshots
### Key characteristics
- EBS Snapshots backup only the data stored on volume, not the whole storage!
- **Snapshots are incremental**. First one is a full copy of data, future snaps are incremental which means that next snapshot are difference between the previous snapshot and the state of volume at the moment of taking snap.
- Billed GB/month
- Thank to how EBS snapshots work, you pay only for the snapped data, not the whole volume allocation.

### Snapshot performance
- By default, restoring EBS volume from snapshots is lazy, data is fetched gradually.
    - Linux tools like dd can be used to speed up this process

#### FSR
Fast Snapshot Restore is a feature that can be enabled on snapshot. It provides immediate restore. There's a limit of 50 FSR snapshots per region. This also comes with extra cost.

## 1.6.9 EBS Encryption

### Key Characteristics
- No encryption by default
- Accounts can be set to encrypt by default - default KMS key or custom KMS key
- Each volume uses 1 unique DEK 
- Snapshots & future volumes use the same DEK
- Once volume is encrypted, you can't make in unencrypted
- OS isn't aware of any encryption so there's no performance loss
- No additional charge

### Encryption and Decryption
Data is encrypted at rest and unencrypted only in the memory of instance. Decryption and encryption happens on EC2 Host between Volume and Instance and EC2 Host stores decrypted DEK to perform decryption. 

## 1.6.10 Network Interfaces, Instance IPs and DNS
### Primary ENI
Each instance has its Primary ENI and it stores:
- Mac address
- Primary IPv4 private address
- 0 or more secondary private IPs
    - Example: 10.16.0.10
        > DNS: ip.10-16-0-10.ec2.internal
- **0 or 1 public IP address** (OS never sees public IP)
    - Dynamic. When stopped and started the IP will be different.
    - Example: 3.89.2.39
        - **It also gets DNS name like:** 
            > ec2.3-89-2-39.compute-1.amazonaws.com
        - This DNS name resolves to private IP address within VPC and to public IP anywhere outside the VPC
- 1 elastic IP per 1 private address
    - Fixed IP address. Can be reassigned to different instances. 
    - Once assigned, it replaces default IP address.
- 0 or more IPv6 addresses
- Security Groups
    - For example, one instance can have two ENIs with different IPs and each of these ENIs will have different Security Groups = Different SG for different IP
- Source/Destination check

There are also secondary ENIs which can be attached to the instance and they store the same information. They can also be deattached unlike primary ENIs.

#### Licensing
Some legacy software has its license tied to MAC address. Using secondary ENI + Mac address, we can move this secondary ENI to different instance and keep the license.


## 1.6.11 Amazon Machine Images (AMI)
- AMIs are regional entities.
    - AMI can be copied from one region to another
    - Even though its a copy, these are completely different objects (AMIs)
    - They still keep Device Block Mapping between EBS snapshots
- AMI can't be edited. You launch instance from this AMI, do changes, and create AMI from it.

### Creating AMIs
- Created AMIs can store paid software
- Creating AMIs creates snapshots for EBS volumes used. 
    - AMI holds is linked to (Device Block Mapping) these snapshots so when instance is launched using AMI, volumes are being restored from the snapshots. This way data is saved.
- Don't create AMI from running instance as it may lead to consistency issues. Stop the instance before creating AMI.
- AMI baking = preparing instance (installing stuff) for creating AMI from it.

### Permissions (who can access)
1. Private (default - only your account)
2. Public
3. Account specific


## 1.6.12 EC2 Purchase Options (Launch Types)

### 1.6.12.1 On-demand
- Isolated but run on the same EC2 Host. Customer instances run on the same hardware.
- Billed per second while instance is running.
    - Storage Instances of this instance are billed regardless of the instance state.
- No upfront cost, predictable pricing
- Default purchase option
- No interruptions
- No discount
- Apps which can't be interrupted


### 1.6.12.2 Spot
- Unused EC2 Host capacity
- Not reliable
- Huge discounts, up to 90%
- Can be terminated at any time whenever AWS needs this capacity for regular instances
- Maximum price is set by user. As capacity is needed more and more, the price goes up. Once price goes above threshold, AWS terminates the spot instances.
- Appropriate for non time crtiical
- Anything that can be rerun
- Cost sensitive workloads
- Anything which is stateless

### 1.6.12.3 Reserved
- Long term usage
- Commitment made to AWS for long consumption of instances
    - 1 year or 3 years
    
    Payment structures:
    1. no upfront = least discount, discounted fee per second wether instance is running or not
    2. partial upfront = more reduced per second fee
    3. all upfront = no per second fee
- Cost per second reduced or entirely removed
- Reservation can be locked to specific region or AZ

### 1.6.12.4 Dedicated Hosts
- You pay for the whole Host instead of single instances
    - No per second charges or anything.
- Hosts may have different types of hardware
- Capacity is managed by user.
- Socker and core licensing requirements

### 1.6.12.5 Dedicated Instances
- You don't own or share the host
    - Hardware is not shared between other customers.
- Extra charges for instances
- For strict regulations where you can't share physical hardware with other customers

### 1.6.12.6 Scheduled reserved instances
- For long term usage that doesn't run consistantly
- Weekly, each Friday from 3PM to 10PM
- Doesn't support all instance types or regions
- Minimum 1200 hours per year & 1 year minimum

### 1.6.12.7 Capacity reservations
- You reserve capacity (lets say for 2 instances of given type) and then you pay for this reservation with discount
- Reservation can be regional or Zonal (AZ) 
    #### On-demand capacity reservation
    - This can be hooked so you always can be sure you have access to given capacity but at full on-demand price
    - No term limits, but you pay for regardless you consume it.

### 1.6.12.8 Saving Plans
- A hourly commitment from 1 to 3 years for a given amount
- A reservation of general compute $$ usage (example, $20 per hour for 3 years)
- Products like EC2, Fargate, Lambda have on-demand rate and savings plan rate. You set up commitment for example for $500, you use it with discount and once you're beyond that amount you pay as with on-demand
- Very useful during migration from EC2 to Fargate 

## 1.6.13 Instance Status Check and AutoRecovery
### 1.16.13.1 Instance Checks -  "2/2 Checks passed"
1. System Check
    - Loss of system power
    - Loss of network connectivity
    - Host software issues
    - Host hardware issues
2. Instance Check 
    - Corrupted file system
    - OS kernel issue
    - Incorect instance networking

### 1.16.13.2 CloudWatch Alarms 
Alarms can be set for each of theses checks. Theses alarms can send event to SNS, recover instance (autorecovery), reboot, move to different EC2 Host.

### 1.16.13.3 EC2 Termination Protection
It can be turned per EC2. It can add another layer of protection since disabling it requires permissions (disableApiTermination). 

## 1.6.13 Horizontal vs Vertical Scaling

### 1.6.13.1 Horizontal
Adding more instances next to the original one.
> 1 instance --> 3 instances ---> 8 instances
- Infinite scaling posibilities
- There might be even a hundred of small instances running the application
    - Because of this load balancer is required
- Sessions, sessions, sessions!!
    - Requires application support or off-host sessions
- No disruption while scaling either in or out.
- Often less expensive
- More granular than vertical scaling

### 1.6.13.2 Vertical
Instance gets bigger, adding more resources to given instance.
> t3.large --> t3.xlarge ---> t3.2xlarge
- Each resize requires a reboot which causes disruption!
    - Because of this, there might be need for resize windows
- Larger instances often carry much higher fees
- There is an upper cap on performance because of the instance size
- No application modification required
- Works for all applications, even monoliths

## 1.6.14 Instance Metadata
- This is a service running behind all of instances running on AWS Account
    
    Address: `http://169.254.169.254/latest/meta-data/<attribute_param>`
    
    Example: `http://169.254.169.254/latest/meta-data/public-hostname`

- There's a tool that can be downloaded and used for searching this service: **ec2-metadata**
- It holds information like user-data, NATed public IPv4 address, public hostname, mac address, availability zone and much more
- SSH keys are temporary uploaded there for **Instance Connect**


# 1.7 Containers and ECS

## 1.7.1 Docker and containers

### 1.7.1.1 Image layers
- It is a read-only change on an image (an intermediate image)
- Every command you specify (FROM, RUN, COPY, etc.) in your Dockerfile causes the previous image to change, thus creating a new layer
- Once created, that layer identified by a sha256 hash will never change

#### Example

```
FROM rails:onbuild
ENV RAILS_ENV production
ENTRYPOINT ["bundle", "exec", "puma"]
```

First, we choose a starting image: rails:onbuild, which in turn has many layers. We add another layer on top of our starting image, setting the environment variable RAILS_ENV with the ENV command. Then, we tell docker to run bundle exec puma (which boots up the rails server). That's another layer.

#### Why do we need that?
The concept of layers comes in handy at the time of building images. Because layers are intermediate images, if you make a change to your Dockerfile, docker will rebuild only the layer that was changed and the ones after that. This is called layer caching.

**IMPORTANT:** If you change or add a layer, Docker will also build any layers that come afterwards because they might be affected by the change.

### 1.7.1.2 Separation 
Containers built with the same image are almost identical, but they have different **WRITE/READ layers** for their own use. This is how containers are kept separated and isolated.

**Container A**
```
897348asds21293421 = unique read/write layer
84734jdkjsdu323kla = same customization layer
j89743987344sadasd = same web layer
jknfhewiu8723y4238 = starting image layer
```
**Container B**
```
9834jksdhf3ukjnss3 = unique read/write layer
84734jdkjsdu323kla
j89743987344sadasd
jknfhewiu8723y4238
```

## 1.7.2 ECS Concepts

### 1.7.2.1 Container Definition
- Container Definition is just a pointer to Task Definition
- Stores information about which image to use for container and exposed ports

### 1.7.2.2 Task Definition
- Task Definition can have one or more containers running inside it
- They store resources, networking information and compatibility information wether it uses EC2 or Fargate (Cluster types)
- **IMPORTANT:** Task Definition contains Task Role which containers can assume to access AWS resources
- Doesn't scale on its own
- Its not Highly Available by default

### 1.7.2.3 Service Definition
- Stores scaling instructions for tasks
- How many copies, HA, restarts

### 1.7.2.4 Cluster modes
1. Network Only (Fargate)
2. EC2 Linux + Networking
3. EC2 Windows + Networking

## 1.7.3 ECS Cluster Types

### EC2 Mode
- In EC2 mode, EC2 instances act as container host. 
- Not serverless, you must manage the EC2 instances
- You pay regardless of whats running on these instances just like with normal EC2 Instances
- You can take advantage of pricing discounts like spot and reserved instances
- Good for large workload, price conscious

### Fargate
- Serverless option, pay for what you use, no waste
- Tasks are running on Fargate Shared Infrastructure but the tasks are isolated just like in EC2 Hosts
- **IMPORTANT:** Each running task has assigned ENI and its also injected into the VPC. From now on, it acts like regular VPC resource
- Good for large workloads, admin conscious
- Also good for batch/periodic/small/burst workloads


## 1.7.4 ECR
### Key Characteristics
- Each AWS Account has public and private registry
    - Public: public R/O, but R/W requires permissions
    - Private: both R/O and R/W requires permissions
- Each registry can contain repositories
- Each repository can hold multiple images
- Images can have several tags (unique)
- Integrated with IAM
- Provides image scanning
    - Basic
    - Enhanced (inspector)
- Provides near real time Metrics to CloudWatch (auth, push, pull)
- API Actions are logged in CloudTrail
- Events can be sent to EventBridge
- ECR supports replication - both cross-region and cross-account

## 1.7.5 EKS

**The control plane** runs in an account managed by AWS, and the Kubernetes API is exposed via the Amazon EKS endpoint associated with your cluster

#### Kubernetes
Notes on Kubernetes [here](#1051-kubernetes)

#### Nodes
1. Self-managed
    - runs on EC2
2. Managed node groups
    - runs on EC2
3. Fargate

# 1.8 Advanced EC2 

## 1.8.1 Bootstrapping EC2 using User Data

### Key characteristics
- ONLY on launch
- EC2 doesn't interpret it, only OS needs to understand it
- Anything in user data is executed by OS
- If there's a problem with user data, you have a problem. Instance works as usual regardless.
- User data is not secure
- Limited to 16 KB in size
- Can be modified when instance is **Stopped**
    - Content only executed once when instance is initialy launched!
- In console, it can be added in plaintext but through CloudFormation or elsewhere it must be Base64 encoded

### 1.8.1.1 Boot-Time-To-Service-Time
To reduce it, we can use AMI baking + bootstrapping. Thanks to this we can make sure that time-consuming instalation part of setting instance up is already there. And then, bootstrapping can be used for final configuration which also speeds up the process and also gives some configurability.

- cfn-init helper script - installed on EC2 OS
- Simple configuration management system
- Procedural (User Data) vs Desired State (cfn-init)
- Packages, Groups, Users, Sources, Files, Commands and Services
- Provided with directives via Metadata and AWS::ClodFormation::Init on a CFN resource
- Variables passed into User Data by CloudFormation

## 1.8.2 Enhanced Bootstrapping with CFN-INIT
CFN-INIT is a powerful desired-state-like configuration engine which is part of the CFN suite of products.
It allows you to set a state for things like packages, users, groups, sources and files within resources inside a template - and it will make that change happen on the instance, performing whatever actions are required.

### CFN Stack updates
Normally, user-data is run only once at instance launch. But thanks to usage of cfn-init and CloudFormation user-data can be run on every Stack update. 

### CFN-SIGNAL and CFN Creation Policy
CFN-SIGNAL functionality enables checks on user-data execution. Creation policy will wait for example for 15 minutes for either success or failure signal from CFN-SIGNAL. CloudFormation won't show CREATED status before it receives SIGNAL.

**Signal example:**
>Received SUCCESS signal with UniqueId i-031471332d0113dfa

## 1.8.3 EC2 Instance Roles

### Key Characteristics
- Most preferable way of managing permissions of applications running inside EC2. Should always be used instead of uploading some keys into instance.
- CLI tools running from within the instance will use ROLE credentials automatically
- Credentials are inside meta-data
   `curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role_name>`
- Automatically rotated - always valid (thanks to STS)

### Instance Profile
Allows permissions to get into the instance. It's created automatically with the same name as role when you use console. But when you use CLI or CloudFormation, you have to do it separately. **It is the Instance Profile, not Role that is attached to the Instance.**

## 1.8.4 SSM Parameter Store 

### Key Characteristics
- Sub service of Systems Manager
- Public service
- IAM integrated
- Storage and retrieval of:
    1. string,
    2. stringlist 
    3. secure string.
- Support 
- Changes can create events
- Hierarchies and versioning
- Public parameters - managed by AWS. For example latest AMI per region.
- Can store ciphertext values which are encrypted by KMS
    - Two permissions required:
        1. KMS permission for decryption
        2. For interaticing with Parameter store

#### Accessing parameter from CLI
```aws ssm get-parameters --names /my-dog-app/dbstring ```

## 1.8.5 System and Application Logging on EC2 

- In order to get information from within the instance, you need to install CloudWatch Agent
    - This allows to gather application logs and system logs like memory usage 
- First you go through config preparation wizard which you can store in Parameter Store and the reuse it in CloudFormation
    - Each Agent must have this loaded before running
- CloudWatch Agent needs permissions for accessing CloudWatch and Parameter Store
    - CloudWatchAgentServerPolicy
    - AmazonSSMFullAccess

## 1.8.6 EC2 Placement Groups 
### 1. Cluster placement groups (PERFORMANCE)
Pack instances physically close together. 
- When you need to have instances running on the same hardware.
- For maximum performance, low latency, fast speeds
- Only one Availability Zone
    - All members of PG have direct connections to each other
    - Setting Locked when launching first instance
- Same rack, sometimes same host
- 10Gbps stream against 5Gpbs normal
- Enhanced networking must be enabled
- Supported instance type is also required (in order to handle the streaming)
    - Very recommended (not mandatory): Run instances at the **same time** and of the same **type**
### 2. Spread (RESILIENCE)
Keep instances physically separated from each other. 
- When you need to have instances running on different hardware.
- Can span multiple Availability Zone (unlike cluster placement)
    - Each instance is placed on a different rack using different hardware
    - Infrastructure isolation
    - There's a limit of 7 instances per AZ (hard limit)
- Not supported for Dedicated Instances or Hosts
- Used for small number of critical instances that must be separated from each other
### 3. Partition (TOPOLOGY AWARENESS)
Groups of instances that are together but on different hardware.
- When you need to have instances grouped together but running on different hardware.
- Span accross multiple AZs
- Instances grouped into partitions (max 7 per AZ)
    - Each partition has its own rack
- No limit of instances running and you can also choose the partition to launch instance into
- Designed for huge scale paralel systems
- Provide visibility into the partitions to see what instances run into it

## 1.8.7 Dedicated Hosts 
### Key Characteristics
- You pay for the HOST, not the instances inside
- Specific family like a1, m5, c5
    - No type mixing like medium and large - NOPE!
    - To get more, you need to use NITRO based dedicated hosts which allows mixing family types 
        https://aws.amazon.com/ec2/dedicated-hosts/pricing/
- On-demand & reserved options available
- Amazon RDS instances are not supported
- Placement Groups are not supported, can't be used together
- AMI Limit: Windows AMIs, RHEL, SUSE Linux are not supported
- Hosts can be shared between Organization Accounts using RAM (Resource Access Manager)

## 1.8.8 Enhanced Networking & EBS Optimized Instances

### 1.8.8.1 Enhanced Networking

- Uses SR-IOV
    - Thank to this NIC (Network Interface Card) is virtualization aware
    -  Network Card being aware of virtualization can present itself as multiple mini cards which are given for guest OS's to use as real cards. No translation needed thanks to this.
- Can be enabled without any additional cost
- More bandwith (higher throughput), higher I/O and PPS (Packets Per Second) and lower CPU usage
    - Now software and CPU doesn't have to work on handling the network operations.
- Consistent low latency

### 1.8.8.2 EBS Optimized Instances
- It means dedicated capacity for EBS
- Historically 
- Most instances support it and have it enabled by default
    - Some older instances support it but with extra cost
- Historically, network was shared for data and EBS networking and this resulted in limited performance for both

# 1.9 Route 53 - Global DNS

## 1.9.1 R53 Hosted Zones
- R53 hosted zone is DNS DB for domains
- Globally resilient
- Created with domain registration via R53 - can be created separately
- Hosted Zones host DNS records (A, AAAA, TXT, MX, NS)
- Hosted Zones are what the DNS system references - Authoritative for a domain e.g. a4l.com

### 1.9.1.1 Public Zone
- DNS database (zone file) hosted by R53 (on public name servers)
- Accessible from internal and public network
- Hosted on 4 R53 Name Servers specific for the zone
- Externally registered domains can reference R53 hosted domains

#### Walking DNS tree
> User request -> Some ISP resolver ->  Root server (points at TLD .com) -> TLD Server (.com pointing at a4l.com) -> R53 Public Zone (a4l.com pointing at shop.a4l.com)

### 1.9.1.2 Private Zone
- Accessible only from VPCs, not accessible from public internet
- VPC must be associated with Private Zone, otherwise it won't be able to connect
    - Using different accounts is possible using CLI/API
- Split view (overlapping public & private IP addresses) for public and internal use within the same zone name. So both user from public internet and internal VPC will connect using the same a4l.com domain name but with different set of records for each. 

## 1.9.2 CNAME vs R53 Alias

### 1.9.2.1 CNAME
- Maps a NAME a another NAME
    - Example: www.catagram.io -> catagram.io
- CNAME is invalid for naked/apex domains (catagram.io, without any domain in front of it like shop.catagram.io)
- Many AWS Services use DNS names
    - With just CNAME catagram.io -> ELB DNS name would be invalid
    - You could point www.catagram.io -> ELB but that's not really great

### 1.9.2.2 R53 Alias
- Maps a NAME to AWS resource
- Support naked/apex domains
- For non naked/apex domains, works exactly the same as CNAME
- There is no charge for ALIAS requests pointing at AWS resources
- For AWS Services - default your choice to Aliases instead CNAME
- Should be the same "type" as the record they're pointing at. A record to A record for example
- Can be used to point at API Gateway, CloudFront, ElasticBeanstalk, ELB, Global Accelerator & S3
- Just in AWS, outside the global DNS system, only when R53 hosts your domain

## 1.9.3 R53 Routing

Quick summary in notes: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html

### 1.9.3.1 Simple Routing
- Not flexible
- Only one record with the same name
- Each record can have mutliple values
- **Doesn't support health checks**
- All values from a query are being returned to client in random order and client decides to which he connects
    - Because health checks are not supported, it might try to connect to currently not working instance
- Use case: When you want to point requests only to one service such as web server

### 1.9.3.2 Health Checks
- Performed by health checkers located globally
    - If 18%+ health checkers report as healthy, the target is healthy
- TCP, HTTP/HTTPS/ HTTP/HTTPS with String Matching
- Health checks are separate from DNS records but are used by them

Healthchecks can monitor:
1. Application's Endpoint 
2. CloudWatch Alarms (State of CloudWatch alarm) 
3. Calucalted Checks (checks of other checks)

### 1.9.3.3 Failover Routing
- Support multiple records with the same name
    - Primary record
        - EC2 Instance web server
        - Only this one has health checks set
    - Secondary record
        - S3 bucket with static maintenance page
        - If the primary target is unhealthy, DNS will respond to request with the secondary record.
- For simple high availability

### 1.9.3.4 Multi Value Routing
- Mixture between **Simple** and **Failover**
- Support multiple records with the same name poiting to different values
    - Each record is independent and have its own health check
- Any record with failed health checks won't be returned when queried
- If there's more than 8 healthy records, there will be 8 chosen and returned randomly
- **Improves availability** - but it IS NOT replacement for load balancing!

### 1.9.3.4 Weighted Routing
- For simple load balancing or testing new software versions
- Support multiple records with the same name poiting to different values
- Each record is returned based on its **record weight vs total weight**
    ```
    www A 1.2.3.3 - 60 (returned 60% of time)
    www A 1.2.3.4 - 20 (returned 20% of time)
    www A 1.2.3.5 - 20 (returned 20% of time)
    total weight = 100 (might be more)
    ```
- If a chosen record is unhealthy, the process of selection is being repeated until healthy record is chosen
- Weight '0' means the record is never returned. If all records have weight '0' then they're all returned

### 1.9.3.5 Latency Routing
- Latency based routing supports 1 record with the same name in each AWS region
    - AWS maintains latency database (not real-time) which stores latency of all available regions and is used when routing the user
    - Healthy, and lowest latency record is being returned, if unhealthy, second lowest latency record is returned
- Use latency-based routing when optimising for performance & user experience

### 1.9.3.6 Geolocation Routing (not closest!)
- User and AWS resource geolocation influences the routing
    - IP check verifies the location of the user
- **Doesn't return closest record**, it returns only relevant records. 
- Records are tagged with location. Hierarchy:
    1. State (US State)
    2. Country
    3. Continent
    4. Default 
- If user tries to access UK page from US, the response will be "NO ANSWER". 
- It returns most specific record or "NO ANSWER"
- Can be used for regional restrictions
    - Language specific content
    - Load balancing across regional endpoints

### 1.9.3.7 Geoproximity Routing
- Returns records based on actual distance (+ bias)
    - Records can be tagged with an AWS Regions or lat & long coordinates
- Bias (like **radius**) expands or shrinks the size of the geographic region from which traffic is routed to a resource.

## 1.9.3 R53 Interoperability
- R53 can do both, Domain Hosting and Domain Registrar

### What happens when you register domain in R53
1. R53 accepts your money (domain registration fee)
2. R53 allocates 4 Name Servers (domain hosting)
3. R53 created a zone file(domain hosting) on the above NS
4. R53 communicates with the registry of the TLD - external identity (Domain Registrar)
5. R53 sets the NS records for the domain to point at the 4 NS (Name Servers) above

# 1.10 Relational Database Service - RDS

## 1.10.1 Database refresher

### 1.10.1.1 Row Store (MySQL)
- Regular OLTP (Online Transaction Processing) database
- Access to data by rows
- Ideal for updating, deleting, adding etc.

### 1.10.1.2 Column Store (Redshift)
- Data warehousing, tragic for OLTP (CRUD)
- Access to data by columns
- Ideal for reporting or when all values for a specific attribute are required
- Columns are stored together

## 1.10.2 ACID vs BASE (transaction models)

### 1.10.2.1 ACID (Consistency, RDS)
- **Atomic** - all or no components of transactions succeeds or fails 
- **Consistent** - transactions in database move from one valid state to another, nothing in between is allowed
- **Isolated** - transactions can't interfere with each other
- **Durable** - once transaction is commited, it will remain committed in case of system failure
- These rules are great for reliable systems like finance but not really scalable

### 1.10.2.2 BASE (Availability, usually NoSQL)
CAP Theorem (you can only choose 2):
1. Consistency
2. Avaiability
3. Partition Tolerant (resilience)

- **Basically Available** - Read and writes are available as much as possible but without any consistency guarantees, kinda, maybe.
- **Soft State** - The database doesn't enforce consistency, this is offloaded to application/developers. Data being read by one instance might not be the same data saved minute ago by another instance.
- **Eventually Consistent** - If we wait long enough, reads from the system will be consistent. There might be currently no data for previous writes.
- Highly scalable, high performance but not really reliable. Most of logic is offloaded to application/developers.
- DynamoDB is example of BASE database (but also supports ACID with DynamoDB transactions)

## 1.10.3 Databases on EC2
### 1.10.3.1 Communication inside AZ and between different AZs
- Data transfer between two Amazon EC2 instances in the same availability zone is free (if there's no VPC peering connection)
- Data transfer between two Amazon EC2 instances in different availability zones within the same region is charged at $0.01 per GB
### Why in the world you should run DB on EC2?
- For access to the DB instance OS
- DBRoot access for more configurability that AWS doesn't provide (AWS provides most of it now through RDS) - might be required by software vendor
- Just because management wants to... - try to question it, look for justification
- To use DB version which is unavailable in AWS or specific OS/DB combination that AWS don't provide
- Maybe in order to implement some specific architecture that AWS don't provide

### Why you shouldn't...
- Admin overhead, managing EC2 and DBHost
- Backup and data recovery management
- EC2 is single AZ
- Missing features - some of AWS DB products are amazing
- No easy scaling, no serverless
- Replication - skills, setup time, monitoring & effectiveness
- Performance, performance and performance - AWS invests a lot of time into optimisation & features

## 1.10.4 RDS Architecture

### Architecture
- First of all, Amazon Aurora is completely different product!
- Managed service, no SSH or OS access
- Multiple databases on one DB Server (instance)

### Cost based on:
- Resource allocation
- Instance size & type  
- Multi AZ or not 
- Storage type & amount (based on EBS attached)
- Data transferred (in and out per gb)
- Backups & snapshots

### Subnet groups
Subnet groups are used to inform RDS which Subnets within a VPC can be used for databases. These must be created before creating RDS instance.

## 1.10.5 RDS - MultiAZ  (HA)

### 1.10.5.1 MultiAZ Instance
Not included in the free tier.
It provides two instances:
1. Primary
    - Synchronous replication
    - This one is used for read and writes
    - In case it fails, standby is used (60-120 seconds failover because of dns caching) 
    - Switching from primary to standby can be also done manualy for testing
    - Historicaly, only option to enable MultiAZ
2. StandBy replica
    - Only one standby which can't be used for read or writes
    - Backups are done from standby replica to offload the primary instance and improve performance

### 1.10.5.2 Cluster
- One primary writer, two readers in different AZs which can be used for **reading**
- Synchronously replicated
- Read instances allow some reading scaling
- Cluster type provides faster hardware (graviton + local NVME SSD storage)
- Each instance has its own local storage
- Failover is faster 
- **IMPORTANT**: Writes are viewed as **"commited"** when at least one reader has confirmed replication!
#### Endpoints
- There's a cluster endpoint (points at writer)
- There is also reader endpoint that points to available reader instance
- Additionally, there are also instance endpoints but these are generally used for testing/fault finding

## 1.10.6 RDS backup & restore and snapshots

### 1.10.6.1 Snapshots
- Snapshots are fully manual
- First snapshot is full (they work like regular EC2 snapshots) - then onward incremental.
- Snapshots must be explicitly/manually deleted
- When deleting RDS, theres a final snapshot to do (optional) which is not deleted automatically.


### 1.10.6.2 Automated Backups
- Automated Backups are like automatic snapshots.
- In addition to automated backups, there are transaction logs saved every 5 minutes
- First automated backup is full, then onward incremental.
- Automated backups are cleared by AWS based on retention period (1-35 days) 

### 1.10.6.3 Cross-Region
- RDS can replicate backups, snapshots and transaction logs to another region. Charges apply for cross-region data copy and the storage on destination.
- Cross-Region is not default, it has to be manually configured in automated backups

### 1.10.6.4 Restore
- Restores aren't fast - think about RTO (Recovery Time Objective)
- Restoring means creating new instance with new address
    - Configuration changes are required, like pointing apps to new endpoint
- Backups are restored from latest snapshot and transaction logs are being replayed to bring DB to the desired point in time (good RPO - Recovery Point Objective)

## 1.10.7 RDS Read Replicas (read scaling)

An RDS read replica instance is an **asynchronous** read-only replica of an upstream primary ("master") database instance. It can be used by your application for any query that does not require changing data, thus relieving load from the master. If the replica crashes or fails, it has no impact on the master but the replica itself can no longer handle any traffic.

**IMPORTANT**: Read Replica can be in a different AZ or even in a different region.

- **Synchronous** = MultiAZ
- **Asynchonous** = Read Replicas

### Characteristics
- 5x direct read-replicas per DB instance 
- Read-Replicas can have their own read-replicas - but lag starts to be a problem
- Read replicas are only for failures - should not be used for data corruption
- Read only - until promoted to regular instance

## 1.10.8 RDS Security

### 1.10.8.3 Encryption
- SSL/TLS (in transit) is supported by RDS, **can be mandatory**
- RDS supports EBS volume encryption - **using KMS**
    - Encryption is done by the Host that the Instance is running on
- KMS customer or AWS managed Master Keys generate DEKs which are used to encrypt logs, storage, snapshots and replicas
- Encryption can't be removed once added

### 1.10.8.2 TDE (better encryption)
RDS MSSQL and RDS Oracle support TDE. TDE = Transparent Data Encryption = Encryption handled within the DB engine. Thanks to this, data is being encrypted before leaving the instance
#### IMPORTANT:
RDS Oracle supports integration with CloudHSM which is even more secure than KMS because CloudHSM is fully managed by customers and keys are never exposed to AWS. Good for very strict regulations and requirements.

### 1.10.8.3 IAM Authentication
- Logins to RDS are controlled by local database users
    - One gets provisioned when creating RDS instance
    - They are outside of control of AWS
- IAM user can be allowed to authenticate against RDS instance
    1. Create local DB account configured to use AWS authentication token.
    2. Policy attached to either User or Role maps that IAM identity to the created local db user
    3. IAM generates (generate-db-auth-token) 15 minute token that can be used to login to RDS without password
    - It does not perform authorization - this one is still handled by DB engine

## 1.10.9 RDS Custom
- It fills the gap between RDS and EC2 running database engine
- In comparison to to RDS - everything is running within your RDS Account
- Supports MSSQL and Oracle
- Provides RDP, SSH, Session Manager access to the instance

## 1.10.10 Amazon Aurora Architecture

### 1.10.10.1 Architecture
- Aurora uses Clusters which consist of single primary instance + 0 or more replicas
- No local storage - Aurora uses cluster volumes which is created by cluster nodes.
    - Cluster volume is based on SSD - high IOPS, low latency. Storage is billed based on what's used.
- Replication happens on storage level, not on the instance level. It offloads the instances.
- By default, only primary instance can perform write operations.
- Aurora automatically detects disk failures and repairs these parts of disk. Then, data on that part is rebuilt using cluster nodes. It provides better storage resilience.
- Aurora supports up to 15 standby, read-only instances.
- Replicas can be added and removed without requiring storage provisioning.
- Aurora has cluster endpoint and reader endpoint. Cluster endpoint always points at primary instance. If there are any replicas, reader endpoint will point at reader replicas. If there are no replicas, it will also point at primary instance.

### 1.10.10.2 Exclusive features
1. Backtrack - allows to revert to a specific point in time in case there's any problem.
2. Fast clones - make new database much faster which points to the original one. It stores only the new data.
3. **Custom endpoints** - Aurora supports custom endpoints which can point at any specific instance.

### 1.10.10.3 Billing
#### High Water Mark (AWS is currently working on changing this)
You're billed for what's used. If you use 500GB you pay for 500GB, if you reduce size to 300GB you still pay for 500GB. Freed up storage can be re-used. In order to cut costs, new Aurora instance must be created and data migrated to create new high water mark.

#### Billing key points
- No free tier option in Aurora since it doesnt support micro instances
- **Compute** - hourly charge, per second, minimum 10 minutes
- **Storage** - GB month consumed based on High Water Mark
- 100% backups storage is included in the instance cost

### 1.10.10.4 Backups & Restore
- Backups and restoring work the same like in RDS
- Restoring creates new **Cluster**

## 1.10.11 Aurora Serverless
Aurora Serverless to Aurora is what Fargate is for ECS.

### 1.10.11.1 Architecture
- Aurora Serverless compute is based on Aurora Capacity Units (ACU) instead instances. Cluster has a MIN & MAX ACU. Cluster adjusts based on load. 
    - Clusters ACU can go to 0 and be paused. 
    User or application connects to ACU through instances in Proxy Fleet, not directly.
- Same resilience as Aurora (6 copies across AZs)

#### Usage
- Aurora Serverless usage - infrequently used applications
- New applications where you're unsure of database load.
- For unpredictable workloads.
- Variable workloads - thanks to ACU scaling
- Dev and test databases

## 1.10.12 Aurora Global Database
### Key characteristics
- Enables Cross-Region replication
- Improves latency and performance
- Up to 5 secondary regions
    - Eeach region can have up to 16 read-only replicas (without writer)
    - Read only replicas can be promoted to R/W instances only in case of any disaster situation!
- Data is replicated on storage level from the master region
    - It has no performance or latency impact on the instances
- ~1s or less replication time between master and secondary regions

## 1.10.13 Multi-master writes 
- Multi-Master doesn't use cluster endpoint. Application must maintain connection to both writers.
    - Thanks to this, if failure happens, the fail-over is immediate.
- Aurora Multi-Muster have no load balancing concept like standard Aurora.

### Writing data
- If writer receives a write request operation, it immediately proposes that data to be committed to all the storage nodes in the cluster
- Writing is waiting for quorum of nodes to agree on the write. If it agrees, data is commited to storage and replicated across all the nodes
    - It can be rejected because there might be another ongoing write (coming from another application). In that case, the instance returns an error.
- **IMPORTANT**: When writing, instances replicate the data between each other so they both can cache it

## 1.10.14 RDS Proxy
### Key characteristics
- **REDUCED LATENCY:** RDS Proxy is used to reduce latency and reduce connections made with database. Instead of connecting to DB instance every time, we connect to RDS proxy through which we use existing pool of connections.
- RDS proxies manage Long Term Connection Pool 
- **FULLY MANAGED:** and works with RDS/Aurora. Has auto scaling and is highly available by default. 
- RDS proxy runs only within AZ and can be accessed from within VPC. Accessed via proxy endpoint.
- RDS proxy can enforce SSL/TLS
- **ABSTRACTED FAILURE:** away from your applications - proxies will handle switching to failover instances and will handle underlying db failures.
    - Can reduce failover time by over 60%


## 1.10.15 Database Migration Service (DMS)

### 1.10.15.1 Architecture
- Source and destination endpoints are crucial
- **At least one DB instance (endpoint) must be in AWS**, no option from on prem to on prem migration
- DMS is a service which runs on **EC2 Replication Instance**. This replication instance runs Replication Tasks which are connected to source and destination endpoints which store replication information.
    - Task moves the data contained in the endpoints.

### 1.10.15.2 Types of jobs performed by DMS
1. **Full load** - full data migration, only if you can affort downtime.
2. **Full load + CDC** (Change Data Capture) - existing data and replicate any ongoing changes.
3. **CDC only** - migrates only new/ongoing changes. Some databases might already have great builtin tools to perform full migrations. After that, DMS with CDC only can be used to move changes made after that initial bulk migration.

### 1.10.15.3 Schema Conversion Tool (SCT)
- This is separate service that runs when needed
- SCT is used when migratin data from one DB engine to another
    - That includes RDS -> S3 migration using DMS
- SCT is not used when there's migration between instances with the same DB engines
    - **Example**: On-premises MySQL -> RDS MySQL
- Works with OLTP and OLAP database types

#### Usage examples:
- On-premises MSSQL -> AWS MySQL
- On-premises PostgreSQL -> AWS Aurora
- AWS MariaDB -> AWS MySQL

### 1.10.15.4 AWS Snowball
- Used for offline data transfering
- Good for huge amounts of data
- AWS delivers a device to you. Then you move your data into it and ship it back to Amazon. Next, Amazon moves this data into S3 and from S3 it uploads it to the desired database.
    - CDC can be used to transfer new changes

# 1.11 Network Storage & Data Lifecycle

## 1.11.1 EFS Architecture
### Characteristics
- EFS is an AWS implementation of NFSv4
- EFS filesystems can only be mounted to linux instances (POSIX - Portable Operating System Interface for UNIX)
- EFS filesystems can be shared between multiple instances
- **Throughput modes:** Bursting and Provisioned
- **Performance Modes**: General purpose or MAX I/O
- **Classes**: Standard and Infrequent Access (IA)
- **Lifecycle policies** can be used to move data between classes

#### Mount Targets
AWS EFS is a private service, accessed via mount targets
- EFS filesystems are accessed through these mount targets. They work like proxies that communicate with the EFS service.
- Mount targets are placed within subnet.
- Mount targets use private IP from subnet IP addresses pool.
- Can be accessed from on premises (for example through Direct Connect).

## 1.11.2 AWS Backup
Service for centralized backup management. Cross account and cross region. Supports wide range of AWS services like EC2, Aurora, DocumentDB, S3, EFS, FSX, EBS
### Key Components
- Backup plans - frequency, backup windows, lifecycle, vaults, region copy
- Resources - what resources are backed up
- Vaults - backup destination, at least 1 must be created and is R/W by default. 
    - Vault lock can be applied: write-once, read-many (WORM). 72h cool-off period, then even AWS cant delete.
- On demand - backups can also be created manually, not only through backup plans
- PITR - some services supports point in time recovery (example S3 and RDS). Restoring to specific point in time during the backup window.

# 1.12  HA & SCALING 

## 1.12.1 ELB evolution

### 1.12.1.1 Version 1 Load Balancer
- **Classic Load Balancer**: Its old load balancer that was introduced in 2009 and lacked many features. Currently replaced by v2 ELBs.

### 1.12.1.2 Version 2 Load Balancers
1. **ALB - Application Load Balancer**: Support HTTP/WebSockets communication
2. **NLB** **- Network Load Balancer**: Supporot TCP, TLS & UDP
These are faster, cheaper, support target groups and rules.

## 1.12.2 ELB (Elastic Load Balancer) Architecture
- Load Balancer actually consist of multiple nodes to which customers are connecting. They scale automatically.
- Load Balancer is created with DNS A record which is pointing at all it's nodes. So all connections are made to the nodes and are equally distributed.
- Load Balancers need 8+ free IP addresses per subnet (=16). AWS recommends /27 subnets to allow scaling.
- ELBs are connected to only one subnet in each AZ
- Configured by **Listener** configuration 

### 1.12.2.1 Public-facing load balancer
- Nodes are given both public and private IP addresses.
- Public facing load balancers can communicate with both private and public instances.
- EC2 doesn't need to be public to communicate with Public Facing LB

### 1.12.2.2 Internal load balancer
- Internal load balancers can communicate with private instances only.
- Nodes have assigned private IP only. (doesnt change where nodes are created)

### 1.12.2.3 Cross-Zone Load Balancing
- **Disabled** by default in Load Balancers and Gateway Load Balancers
- **Enabled** by default in Application Load Balancer
- **IMPORTANT!** Helps with uneven traffic distribution across mutliple AZs
    - **Example**: Under **Node A** there is 10 instances but under **Node B** there is 1 instance. If there are 2 nodes, then each is getting 50% of load. Then, these 10 instances under **Node A** would get each 10% and single instance under **Node B** would get 100% of load from this 50% that each node had.

## 1.12.3 Application Load balancing (ALB) vs Network Load Balancing (NLB) 

### 1.12.3.1 NLB
- Unlike ALB, NLB allows **unbroken SSL encryption** by forwarding TCP connections to instances. This is a huge deal in terms of security! 
- Operates on layer 4 - TCP, UDP, TLS and else
- Blazingly fast. Milions rps
- It provides **static IP** (good for whitelisting)
- Doesn't support HTTP/HTTPS
    - No headers, cookies, path handling etc.
- Healthchecks check only ICMP/TCP handshake, no app awareness

#### Use cases
- Should be default for everything that doesn't need HTTP/HTTPS
- Private Link

### 1.12.3.2 ALB
- **SSL encryption is terminated at ALB**. Then new connection is created in ALB and used to communicate with Instance.
    - Because of this, ALB needs his own SSL certificates
- Operates ONLY on layer 7 - HTTP/HTTPS
    - content type, cookies, custom headers, path, user location, app behaviour
- ALBs are slower than NLB (they have more levels of network to process)
- Support layer 7 healt checks - can health check application

#### Rules
ALB works based on rules
- Rules are directing connections which are arriving at listener
- Rules are processed in priority order
- Default rule is **catchall**
- Rule Conditions:
    - host-header, http-header, http-request-method, path-pattern, query-string, source-ip
- Rule Actions:
    - forward, redirect, fixed-response, authenticate-oidc, & authenticate-cognito

#### Use cases
- Anything that NLB can't handle

## 1.12.4 Launch Configuration and Templates 
- Configuration of EC2 instances in advance
    - AMI, instance type, storage, key pair
    - Networking and security groups
    - User data & IAM role
- Both are not editable - defined once but Launch Templates have **VERSIONS**
- **LT** provides newer features - more advised to use by Amazon
- **Launch Configurations** are used by **auto scaling groups** (no editable, no versioning).
- **LT** can be used for the same thing as LC but they support versions and you can directly create instances using them.

## 1.12.5 Auto-Scaling Groups

### 1.12.5.1 Key Concepts
- ASGs are used for automatic scaling and self-healing for EC2.
- ASGs use Launch Templates or Launch Configurations.
- ASG defines WHEN and WHERE (launched). Launch Templates/LC defines WHAT (config they have)

### 1.12.5.2 Scaling Policies
ASGS don't need scaling policies - they can have none (means they use min, desired, max).
1. **Manual Scaling** - Manually adjust the desired capacity
    - There are 3 states: MIN, DESIRED, MAX. ASGs keep instances running at the DESIRED capacity by provisioning or terminating instances. 
2. **Schedule Scaling** - Time based adjustment - e.g. Sales periods/windows.
3. **Dynamic Scaling**: 
    1. Simple - "CPU above 50% +1", "CPU Below 50 -1". It can be memory, disk I/O. It can even be length of SQS queue.
    2. Stepped Scaling - Bigger +/- based on difference/spikes.
        - 40-50% do nothing, when 60-70% add one, when 80-95% add 4 instances
    3. Target Tracking - maintain desired aggregate CPU = 40%. ASGs will do the job to keep the CPU at 40%.

### 1.12.5.3 Scaling Processes
- Scaling Processes can be set to SUSPENDED or RESUMED. 
    - When Launch is set to SUSPENDED the ASG won't scale out if any alarms are raised.
    - When Terminate is set to SUSPENDED no instance will be terminated
- **AddToLoadBalancer** - wether any instances provisioned are added to Load Balancer
- **AlarmNotification** - control wether ASG will react to any CW alarms
- **AZRebalance** - controls wether ASG will try to redistribute instances across available AZs
- **HealthCheck** - instance health checks
- **ReplaceUnhealthy** - terminates unhealthy instance and replace
- **ScheduledActions** - wether ASG will perform any scheduled actions (on/off)
- **Standby** - use this option per instance. Set either to InService or Standby. Useful when you need to perform maintenance on one or more instances. Set them to standby, and they won't be affected by anything that ASG does.

### 1.12.5.4 LB + ASG
- ASGs can be used with Load Balancers. The target group can be connected to ASG which will add or remove instances
- LB can also perform health checks when used with ASG.

### 1.12.5.5 Billing
- **IMPORTANT:** Cooldown Periods - it tells how long ASGs will wait before they perform action. It helps avoid creating/terminating instances very frequently
- Use cool downs to avoid rapid scaling.
- Think about more smaller instances rather than fewer, big instances.
- - Autoscaling Groups are free, only the resources created are billed.

## 1.12.6 ASG Lifecycle Hooks
- Allow **Custom Actions** on instances during ASG Actions - Instance Launch or Instance Terminate
    - Instances are paused within the flow and wait until:
        1. Timeout (then either continue or abandon)
        2. ASG process is resumed by **CompleteLifecycleAction** (happens on PENDING/TERMINATING: Wait step)
- Can be integrated with EventBridge (to initiate other processes) or SNS Notifications

### 1. Example when Scaling OUT
#### a. Normal scaling without hooks: 
>Scale out -> PENDING -> InService
#### b. Scaling out with hooks:
>Scale out -> PENDING -> [PENDING: Wait -> (this is place for actions like loading or indexing data) -> PENDING: Proceed] -> InService

### 2. Example when Scaling IN
#### a. Normal scaling without hooks: 
>Scale in -> TERMINATING -> TERMINATED`
#### b. Scaling in with hooks:
>Scale in -> Terminating -> [TERMINATING: Wait -> (this is place for actions like saving logs or doing backups) -> TERMINATING: Proceed] -> TERMINATED


## 1.12.7 ASG HealthCheck Comparison - EC2 vs ELB 

### 1.12.7.1 EC2
- Default option
- EC2 - Stopping, Stopped, Terminated, Shutting Down, IMpaired (not 2/2 status) = UNHEALTHY

### 1.12.7.2 ELB
- Can be enabled
- HEALTHY = Running & Passing ELB health check. So both instance and application (thanks to l7 support) must be healthy.
- Support Custom Health Check - instances can be marked healthy & unhealthy by external system
#### IMPORTANT: Grace Period
Default is 300s. It is a delay before starting health checks. Allows sytem launch, bootstrapping and application start. Health check grace period is most important, otherwise instance might fall into fail-replace circle. 


## 1.12.8 SSL Handling & Session Stickiness 

### 1.12.8.1 SSL Bridging (ALB)
- SSL encryption is terminated at ALB. Then new connection is created in ALB and used to communicate with Instance. ELB and all instances must store certificates in order to handle cryptographic operations. Certificate is exposed to AWS on LB level.

### 1.12.8.2 SSL Pass Through (NLB)
- Allows unbroken SSL encryption by forwarding TCP connections to instances. No need to store certificate on ELB so it's never exposed to AWS. However, the certificate must be stored on Instances.

### 1.12.8.3 SSL Offloading
- Encrypted SLL connection is being made to ELB, it gets terminated and decrypted and passed as unencrypted HTTP traffic to the instances and other AWS resources. If that's not a problem, it can offload the instances so they don't have to store any certificates. Load Balancer still needs to have certificate.

### 1.12.8.4 Connection Stickiness
- Stickiness (LB) creates a cookie which locks user to specific instance. Thanks to this, traffic is distributed between instances but client's won't be redirected to other instance on every connection. 
- If origin instance fails, user will be moved to another
- If cookie expires, the process starts over

Best practice - all application should stateless and the session state should be stored somehwere else for example in Dynamodb. This way session is not stored on instance level, and all instances can access the session info for the users and Load Balancers can do their job regularly without any additional cookies.

## 1.12.9 Gateway Load Balancer (GWLB)
### Key Characteristics
- GWLB help to run and **scale** 3rd party appliances, things like:
    - Firewalls, intrusion detection & prevention systems
- Inbound and Outbound traffic (transparent inspection)
- It uses GLWB Endpoints (GLWBE) - traffic enters and leaves through these endpoints
- GLWB distributes traffic across multiple Backend Appliances
- Traffic and metadata is tunelled using Geneve protocol. 

### Traffic flow
>Client -> GWLB Endpoint -> GWLB -> Backend Appliances -> GWLB -> GWLBE -> Destination Instance
- Geneve protocol is used between GLWB and Backend Appliances. Packets are encapsulated when it routes traffic to Backend Appliances and is decapsulated when its leaving GLWB to the destination.
- Original packets remain unaltered encapsulated through to appliances & back
- Packets cant be changed because thats the whole point of the inspection by Backend Appliances
    - These can perform scanning, analyzing the traffic or blocking packets if needed

# 1.13 Serverless and Application Services

## 1.13.1 AWS Lambda
- Stateless
- Each invocation is different from previous and runs in brand new environment
- Billed for compute resources used during invocation
- Function timeout = 15 minutes

### Usecases
1. Serverless applications
2. Database triggers (DynamoDB, DynamoDB streams, Lambda)
3. File processing (S3, S3 Events, Lambda)
4. Serverless Cron (EventBridge/CWEvents + Lambda)
5. Real-time data processing (Kinesis + Lambda)

- In order to enable logging on lambda, it must be given permissions via Execution Role.

### 1.13.1.1 Running Lambda Publicly
- Lambda Func running in public networking mode can access other public services but (DynamoDB, S3) can't access private services running inside VPC.
- Public networking offers the best performance because no customer specific VPC networking is required.
- Public setting is default.

### 1.13.1.2 Running Lambda Privately
- When Lambda is running in private mode can access resources places within but can't access anything outside the VPC by default - it must be configured
- Lambdas in order to run require 1 ENI in VPC if all run in the same subnet and have the same security group.
- If they have the same security groups but run in different subnets, it requires 1 ENI in each Subnet
- Lambda networking configuration is created on lambda creation. So there's no need to create one on each invocation.

### 1.13.1.3 Lambda Permissions
- Lambda uses Execution Roles.
- Lambda has resource policies - they control what services and accounts can invoke lambda functions.

### 1.13.1.4 Lambda Invocation Methods
1. Synchronous
    - CLI/API invoke lambda function
    - API Gateway invocation
    - Client is always waiting for response (Success/Failure)
    - Any errors have to be handled on client side
2. Asynchronous
    - Typically when other AWS Services invoke lambda
    - S3 Event is an example where it just triggers lambda and forgets about it, doesn't wait for response.
    - Lambda needs to be configured to handle re-processing. It must be idempotent. Lambda will retry between 0-2 times (configured).
3. Event source mappings
    - Typically used with services that don't support event generation to invoke lambda. (sqs, kinesis, dynamodb stream)
    - Event Source Mapping component is looking for and pulling new data from source in Source Batches. It splits these batches into Event Batches if required and sends to Lambda Function to process. Event Batches must be in proper size so Lambda won't timeout (15 minutes).
    - Event Source Mapping uses Lambda Execution role to pull data from the source.
    - SQS queues or SNS topics can be used for any discarded failed event batches

### 1.13.1.5 Lambda Versions
- Lambda functions have versions - v1, v2, v3, v4...
    - `$latest` points at newest lambda version
- Each version is code + configuration
- Versions are immutable, never changes once published
- Each version has it's own ARN
- Lambda Functions support aliases (DEV, PROD, STAGE) that can point to a specific lambda version 

### 1.13.1.6 How Lambda runs

**Execution Context** is an environment that lambda function runs in. 

#### Cold Start
A cold start is a full creation and configuration of the execution context including function code download. It takes ~100ms.

#### Warm Start
When there's not much delay between invocatios, lambda function cas reuse existing previously created environment. New event is passed to the execution context and the creation of it can be skipped. It takes ~1-2ms. /tmp folder can be used to download some resources like images and use in multiple invocations since the context will be shared.

- Lambda must always assume that warm start is not available.
- For best performance, it should also be aware that there might be warm start available and should be able to take use of that.
- **Provisioned Concurrency** can also be used. AWS will create and keep X contexts warm and ready to use for a given period. It highly improves start speed.
    - More about [provisioned concurrency](https://stackoverflow.com/questions/60059435/provisioned-concurrency-not-resolving-cold-start)
    - They dont fully solve the problem but they reduce the cold start time as it happens in the background before lambda is available to be invoked.

## 1.13.2 CloudWatchEvents and EventBridge 
- EventBridge is basically CloudWatch Events v2 - it also support 3rd party services and custom applications.
- CloudWatch Events has one default event bus per AWS Account and it's implicit, invisible to the UI.
- EventBridge allows creating custom event buses
- **Bus** is a stream of events which occur from any supported service in AWS Account.
- Rules match specific events and it routes the event to a specific target defined in the rule (ec2 instances stopped -> event generated -> lambda function get the info)
    - Event Pattern Rule
    - Schedule rule
- Events are json structures

## 1.13.3 Simple Notification Service (SNS)
### 1.13.3.1 Pub-sub
- Publisher sends message into a **topic**
- Each topic can have **subscribers**
- Subscribers receive all **messages** from topic
    - **Subscriber** examples: HTTP(S) endpoints, EMAIL addresses, SQS queues (each messages is added to the queue as its send to the topic), Mobile Push notification systems, SMS messages & Lambda 
    - Filters can be applied to subscribers
### Fanout
 The Fanout scenario is when a message published to an SNS topic is replicated and pushed to multiple endpoints, such as Firehose delivery streams, Amazon SQS queues, HTTP(S) endpoints, and Lambda functions. This allows for parallel asynchronous processing.

### Key Characteristics
- Public AWS service
- Coordinating the sending and delivery of messages
    - Messages are <= 256KB
- Provide delivery status
- Provide delivery retries
- HA and Scalable
- Support SSE (Server Side Encryption)
- Cross-account via **Topic Policy**


## 1.13.4 Step Functions

### Problems (Design) of Lambda that Step Functions solve
- Lambda is FaaS
- 15 minute max execution time
- Can be chained together... but gets messy at scale
- Runtime environments are stateless

### 1.13.4.1 State Machines
- START -> **STATES** -> END
- States are things that occur inside workflow with maximum duration of 1 year
- They support standard workflow (1y) and express workflow (5min)
- State Machines can be started via API Gateway, IOT rules, EventBridge, Lambda and more
- IAM role is used for permissions
- Amazon States Language - json template for State Machines

### 1.13.4.2 States of State Machines
- SUCCEED & FAIL
- WAIT
- CHOICE (based on input)
- PARALLEL (multiple things at the same time)
- MAP (accept list of things, for each perform action or set of actions)
- TASK (single unit of work performed by state machine)
    - Lambda, SNS, glue, sagemaker, dynamodb, ECS, Step Functions

## 1.13.5 API Gateway 101 

### Key Characteristics
- Caching is configured on stage level
    - Cache size 500MB to 237GB
    - Can be encrypted
    - Cache TTL default is 300s
    - Calls are only made to backend integrations if request is a cache miss
- API GW can use custom or cognito authentication
- Support regular APIs, REST or Web-socket

### 1.13.5.1 Endpoint Types
1. Edge optimized - routed to the nearest CloudFront POP (Point of Presence)
2. Regional (Clients in the same region)
3. Private (accessible only within VPC via interface endpoint)

### 1.13.5.2 Errors
- 4XX - Client errors
- 5XX - Valid request, server errors
- 400 - bad request (generic)
- 403 - access denied
- 429 - API Gateway can throttle, it indicates that threshold was exceeded
- 502 - Bad Gateway, bad output returns by Lambda
- 503 - Service Unavailable
- 504 - Integration Failure/Timemout

## 1.13.6 Simple Queue Service (SQS)
- Public, fully managed, highly-available
- Standard (could be out of order) and FIFO queues
- Messages up to 256KB, other can be stored on S3 and linked
- Received messages are hidden (VisibilityTimeout)
    - Then either reappear (retry) or explicitly deleted
- ASGs can be scaled based on queue and lambdas can be invoked based on the queue length
- **Billed** based on requests
    - 1 request = 1-10 messages up to 64KB
- Messages live up to 14 days. Encryption at rest is supported (KMS)
- Access based on Identity Policies or Queue Policies (can allow access from external accounts)
    - Both can controll access within the same account

### 1.13.6.1 Polling Types
1. Short
    - response is immediate
    - if queue has 0 messages, it still consumes request which is billed and 0 messages is returned
    - keeping queue to 0 length would require contanst requests 
2. Long
    - preferable way to use SQS
    - more cost effective
    - it waits for messages and returns up to 10 messages per request
    - wait up to 20 seconds

### 1.13.6.2 Queue Types
1. Standard
    - nearly unlimited TPS
    - much faster and more scalable
    - best-effort ordering, no message order
    - At-Least-Once-Delivery: **there could be more than one copy of the same message** so applications using queue must be able to handle it
2. FIFO
    - 300 TPS w/o batching, 3,000 with batching
    - first in, first out - order guaranteed
    - **fifo suffix** in queue name is required
    - exactly one processing, **duplicates are removed**

#### Delay Queue (0 minutes - 15 minutes)
It allows Delay in seconds once the message is added to the queue, it can't be processed until the delayed period.
- Messages timers allow a per-message-visibility to be set overriding any queue settings (not supported on FIFO queues).

#### Visibility Timeout (0 seconds - 12 hours)
Messages becomes invisible for a period of time (in seconds) after its processed. Then it reappers after this timeout if its not explicitly deleted. Reapearing message might tell us that there's something wrong with it and should be sent to DLQ for other type of processing.
- ChangeMessageVisibility changes the value per message

### 1.13.6.3 DLQ
- One DLQ can be used for many source queues
- DLQ can perform different operations for problematic messages
- Messages are moved from source to DLQ based on how many times it was added to the queue = **maxReceiveCount**
- All SQS queues have retention period for messages. If a messaged passes the certain point and hasn't been processed then that message is dropped. Enqueue of message is unchanged when moved to DLQ. If you have retention period of 2 days, message got to DLQ after one day then this message will only remain for one last day.

## 1.13.7 Kinesis Data Streams 
- **PURPOSE:** Enables large data ingestion into AWS
- **Producers** send data into Kinesis stream
- **Consumers can read data from the streams in real time**
- **Streams** can scale from low to almost infinite data rates
- It is public service & is highly available by design
- **Streams** store data in 24 hour moving window of data. It means for how long the data can be accessed in the stream. After 24h it is being discarded. 
    - It can be increased to 365 days (additional cost).
- Multiple **consumers** can use the same data stream and access that data from a moving window.
- Kinesis stream consist of **shards** the more shards it has, the better performance it has and is more expensive
- Designed for **huge scale ingestion**, data analytics, monitoring, data ingestion
- Doesn't offer persistent storage, once they pass rolling window they're gone forever. Firehose must be used for persistent storage.

## 1.13.8 Kinesis Data Firehose (now Amazon Data Firehose)
- **PURPOSE**: Used with Kinesis Streams for data persistency.
- Fully serverless and managed
- Near real time delivery of data (~60s)
    - *Note:* It receives data in real time but delivers in not real time
    - It waits for 60s or 1MB then delivers the data
- Supports data transformation on fly with lambda (adds latency)
- Billing - volume through firehose, pay as you use
- Fully managed service to load data for data lakes, data stores and analytics services

### Valid Targets:
1. HTTP endpoint
2. Splunk
3. Redshift (it uses intermediate S3, then copies all data from S3 into Redshift. All handled by firehose.)
4. ElasticSearch
5. Destination S3 Bucket

## 1.13.9 Kinesis Data Analytics (now Amazon Apache Flink)
- **PURPOSE**: Real time data analytics service
- Uses SQL for analytics
- After data is processed, it can send it to multiple supported targets
- It doesnt modify the sources anyway
- You only play for what you use
- Usecases: 
    - Streaming data that need real time data processing
    - Elections/esports
    - Realtime game dashboards
    - Real time metrics - security & response teams

### Valid Sources For Analysis
1. Firehose
2. Kinesis Streams 

### Valid Targets
1. Firehose (and indirectly all destinations that it supports)
    - after sending data to firehose, its no longer real-time
2. Lambda (real-time)
3. Kinesis Data Streams (real-time)

## 1.13.8 Kinesis Video Streams
- Ingest live video streams from producers
    - Cards, drones, security cameras, smartphones, time-serialised like thermal data, radio, depth and radar data
- Consumers can access data frame-by-frame or as needed
- Can persist and encrypt (at rest and at transit)
- Accessible only via API (you can't directly access through S3 for example)
- Integrates with other AWS services like Rekognition and Connect 

## 1.13.10 AWS Cognito
- Identity platform for web and mobile apps
- It’s a user directory, an authentication server
- **Not multi region support**
- No way to export/import passwords


### User Pools
This is about sign up & sign in for users. It allows users to create accounts that are centrally stored in Cognito (there’s even a hosted UI provided by Cognito to complete the sign up process which you can integrate into your site).
- Used to access apps,
- After succesfull login, it returns JWT
- CAN'T be used to access AWS resources

### Identity Pools
Used for swapping identity tokens from external identity provider to AWS temporary credentials. Swapping credentials is knows as **identity federation**.
- Used to access AWS resources,
- Swap authenticated or unauthenticated identity to get temporary AWS credentials (it uses IAM roles),
    - Can swap credentials from Google, Facebook, Twitter etc.
    - A possible identity is also a User Pool identity

## Cognito User Pools vs. Identity Pools

| Feature | **User Pools** | **Identity Pools** |
|---------|--------------|----------------|
| **Main Purpose** | Authenticate users for your **frontend application** | Grant users **temporary AWS credentials** to access AWS services |
| **How it Works?** | Manages users with sign-up/sign-in, MFA, and user attributes | Maps authenticated users (from Cognito or external providers) to **IAM roles** |
| **Access Control** | Provides a **JWT token** (ID & Access token) for authentication | Provides **temporary AWS IAM credentials** (Access Key & Secret) |
| **Use Case** | When users need to log in to your **app** (Web/Mobile) | When users need **direct access** to AWS services (S3, DynamoDB, etc.) |
| **Identity Providers** | Cognito itself (built-in user directory) | Cognito User Pools, Google, Facebook, Apple, SAML, OpenID Connect |
| **Where is it Used?** | Frontend (authentication) | Backend & AWS services (authorization) |
| **Token Type** | JWT (ID token, Access token) | Temporary AWS IAM credentials (Access Key, Secret Key, Session Token) |

---

## Sccenarion and usage

| Scenario | Use **User Pool** | Use **Identity Pool** |
|----------|----------------|----------------|
| Sign up/sign in users for your app | ✅ | ❌ |
| Authenticate with Google, Facebook, etc. | ✅ | ✅ (federated identity) |
| Protect backend API (e.g., API Gateway, Lambda) | ✅ | ❌ |
| Allow frontend to access AWS services directly (e.g., S3, DynamoDB) | ❌ | ✅ |
| Custom authentication (multi-tenancy, custom logic) | ✅ | ❌ |


## 1.13.11 AWS Glue 101 
- **Serverless & cost effective ETL service** (Extract, Transform, Load)
- It moves and transforms data between source and destination

### Valid Data Sources
1. Stores
    - S3
    - RDS
    - JDBC Compatible 
    - DynamoDB
2. Streams
    - Kinesis Data Stream
    - Apache Kafka

### Valid Data Targets
- S3
- RDS
- JDBC Databases

### 1.13.11.1 Data Catalog
- Persistent metadata about data sources in region/Central metadata repository to view your data.
- One catalog per region per account
- Crawlers connect to the data stores, determine schema and create metadata in the **data catalog**

### 1.13.11.2 Glue Jobs
- Glue jobs can be initiated manually or via events using EventBridge
- Use provided scripts (python, scala)

Glue vs Lambda: https://stackoverflow.com/questions/63599886/is-aws-lambda-preferred-over-aws-glue-job

## 1.13.12 AWS MQ
- Open source message broker based on Apache ActiveMQ or RabbitMQ.
- VPC based, its not a public service. Private networking required.
    - To connect to on prem, VPN or Direct Connect is required!
- No AWS native integration. It delivers a product which you manage.

### Use cases
- SNS and SQS for most new implementations
- SNS and SQS if AWS services integration is required
- Amazon MQ if you need to migrate from and existing system with little to no application change
- Amazon MQ if APIs such as JMS or protocols like AMQP, MQTT, OpenWire and STOMP are needed
- SNS and SQS if public access is required

## 1.13.13 Amazon AppFlow
- Fully managed integration service
- Exchange data between applications using **connectors** and **flows**
    - Connections can be reused across many flows
- Securely transfer data between Software-as-a-Service (SaaS) applications like Salesforce, SAP, Zendesk, Slack, and ServiceNow, and AWS services like Amazon S3 and Amazon Redshift, in just a few clicks
- Sync data across applications
- Support tickets from Zendesk to S3

# 1.14 Global Content Delivery and Optimization

## 1.14.1 CloudFront

- CloudFront reduces load on Origin (S3 for example).
- It uses Edge Locations
- After configuration, Distribution is pushed to Edge Locations.
- CloudFront integrates with ACM for HTTPS 
- CloudFront performs no write caching. Its read-only (download-only operations).

### 1.14.1.1 Distribution
- Distribution has its own DNS name
- Distribution has information about Origin
- It stores behaviours which can control what is being cached
- After configuration, Distribution is pushed to Edge Locations.
- Distribution can use domain name

#### Cache check path
User -> Checks Edge Location -> Checks Regional Edge -> Origin Fetch -> Return data to user

### 1.14.1.2 Cache Behaviour
- Cache behaviour can be configured. 
- Each Distribution have one or more behaviours with different restrictions.
- Default behaviour is `*`. There can be behaviour like `img/*` which looks for specific components to cache.
- Distribution is configuration unit of CloudFront

### 1.14.1.3 TTL and Invalidations 
- How long objects are stored at edge locations and when they're rejected
- Default TTL (behaviour) = 24 hours (validity period). After this period, they're viewed as expired. 
- Minimum TTL and Maximum TTL can be set.
- **Headers used**. These headers can be set by Custom Origin (injected) or S3 (via object metadata:
    - Cache-Control max-age (in seconds), Cache-Control s-maxage (second)
    - Expires (date & time) - when object expires

#### 1.14.1.3.1 Cache Expiration
- If object expires, its not immediately removed. 
- When user reuqests it, there's request made to origin to check if there's new version. If there's new version, it returns 200 OK with new version. If theres no new version, origin will return 304 Not Modified.

**It comes with PROBLEM:** If there's new version in Origin, users will still receive outdated objects because objects hasn't expired yet.

#### 1.14.1.3.2 Invalidations
Cache Invalidations is performed on a distribution. It expires objects regardles of TTL based on specified path. Its applied to all edge location in given distribution.
- Only should be used for correcting errors
- If you often need to use invalidations on specific miles you might want to use Versioned Filenames

#### 1.14.1.3.3 Versioned Filenames
...whiskers1_v1.jpeg -> whiskers1_v2.jpeg -> whiskers1_v3.jpeg .... and so on
- Each object is cached separately on every edge location
- Loggin is more effective because nothing has the same name
- Less expensive, no need to run cache invalidations

### 1.14.1.4 Cloudfront and SSL/TLS 
- SSL can be enabled on CloudFront default domain which ends with cloudfront.net. 
- CloudFront needs publicly trusted certificates. It can't use self-signed certificates.
- Connection Protocols:
    - User -> CloudFront = Viewer Protocol
    - CloudFront -> Origin (S3, ELB) = Origin Protocol

#### 1.14.1.4.1 SNI
SNI = **Server Name Indication**
- SNI is an extension to the TLS protocol that allows a client to indicate which hostname it is attempting to connect to at the start of the handshake process. This allows a server to present multiple certificates on the same IP address and TCP port number, enabling HTTPS websites to serve multiple SSL certificates.
- Old browsers don't support SNI
    - CloudFront charges extra for dedicated IPs
    
**Example**:
Imagine you have three websites:

    example1.com
    example2.com
    example3.com

Without SNI:

    Each site would need its own dedicated IP address if using HTTPS, because the server would not know which SSL certificate to present before the TLS handshake is completed.

With SNI:

    All three websites can share the same IP address.
    During the TLS handshake, the browser tells the server which website it's trying to reach.
    The server then presents the correct SSL certificate for that site.

### 1.14.1.5 Securing CloudFront Custom and S3 Origins

#### 1.14.1.5.1 Securing S3 origin using OAI (Origin Access Identity)
- Currently, OAI is being replaced by OAC (Origin Access Control)
- OAI is only applicable for S3 origins and is like identity
- It can be associated with CloudFront distribution
    - CloudFront Distribution 'becomes' that OAI when accessing S3 Origin
    - OAI can be used in Bucket Policies
    - implicit default deny =  DENY all but ONE or more OAis
    - Edge Locations can access Origin
- Using this, only CloudFront distribution can access the S3 Origin. So useres won't be able to go around the CloudFront and directly access the S3.

#### 1.14.1.5.2 Securing Custom Origin
1. Custom Headers
    - These custom headers are injected into at the Edge Location level so they can't be seen by users
    - Custom Headers are configured within CloudFront
    - Custom Origin won't accept requests without headers
2. Firewall
    - It can be used to allow only cloudfront public IPs
    - Amazon lists all their public IPs so they're easily accessible

### 1.14.1.6 Private Distribution & Behaviours 
1. Public behaviour
    - Accessible by everyone
2. Private behaviour
    - Accepts only requests with signed cookie or signed URL
        - SignedURLs allow access to one object only
        - Cookies provide access to a group of objects

## 1.14.2 AWS Certificate Manager (ACM)
- Can't be used with EC2. ACM can be used with specific services only - ALB, CloudFront, API Gateway
    - It can't be used within EC2 because it would be easily accessible.
- **ACM is regional service.**
    - Certificate needs to be the in the same region as service which want's to use it. 
    - Global Services such as CloudFront operate as though within us-east-1 so ACM Certificate must be placed in us-east-1
    - Certificates can't be moved across Regions when created
- When ACM certificate is linked to CF Distribution, it's deployed to all Edge Locations to prove identity, establish trust and secure connections.
- **Rotation in ACM:** If certificate was generated within ACM it will auto-rotate, if it was imported, it needs to be manually rotated.


## 1.14.3 Lambda@Edge 
- Run lightweight Lambda at the CloudFront edge locations
- Run in AWS Public Space (not VPC)
- Different time limits (much less)
- Layers are not supported
- Only node.js and python supported

#### Communication between clients and CloudFront Distribution
    
    (from client to edge) Viewer request -> (from edge to origin) Origin request
    (to client from edge) Viewer response <- (from origin to edge) Origin response
Lambda@edge can run at any of these stages.

### 1.14.3.1 Use cases
1. A/B testing (viewer request)
2. Migration between S3 origins (origin request)
3. Different object based on device (origin request)
4. Content by country

## 1.14.4 Global Accelerator 
- Moves AWS Network to customers as close as possible
- Helps with connection to multi-region applications
    - Transit traffic over AWS backbone to 1+ locations
- Connections enter at the edge using **anycast IP addresses**
    - Amazon provides 2 anycast addresses
- Can be used for TCP/UDP (unlike CloudFront)
- Doesn't provide any cache
- It's network optimisation product, it doesn't understand HTTP

# 1.15  Advanced VPC Networking

## 1.15.1 VPC Flow Logs
- VPC Flow Logs **capture metadata only**, not content of packets. For packets you would need packet sniffer.
- Flow Logs are **NOT realtime**
- Can be attached to VPC - All ENIs in that VPC
- Can be attached to Subnet - all ENIs in that subnet
- Can be attached to ENIs directly
- **Log destinations** are S3 or CloudWatch Logs. WIth S3 you can use Athena for querying and you can integrate it with 3rd party log tools. When using CloudWatch Logs you can access them through API or integrate with other services within AWS.
- Flow Logs can record ACCEPTED, REJECTED or ALL metadata

### 1.15.1.1 Flow Log Records
- Flow Logs captures objects known as **Flow Log Records**
- Sample:
    >2 123456789010 eni-1235b8ca123456789 172.31.16.139 172.31.16.21 20641 22 6 20 4249 1418530010 1418530070 ACCEPT OK
    - 1 = ICMP, TCP = 6, UDP = 17
- They contain information like srcIp, dstIp, srcPort, dstPort, protocol, log status

## 1.15.2 Egress-Only Internet gateway
- Egress-Only IGW is outboundy-only for IPv6
- It works exactly like regular Internet Gateway but designed for IPv6. Its attached to VPC and traffic is routed to it based on Route Tables.
- All IPv6 addresses are public so it needs some piece of functionality like IPv4 which has NAT.
- Traffic coming back from the Internet is allowed because Internet Gateways are stateful. Inbound connection initiated from public Internet is denied.

## 1.15.3 VPC Gateway Endpoint 
- Its used to **provide private instances access to public resources** - S3 and DynamoDB.
- **Gateway Endpoint** isn't attached to a particular subnet, its attached to a VPC and AWS automatically configures the route tables.
- When added, a **Prefix List** is added to the route table and it points at Gateway Endpoint. Prefix List are addresses for S3 and DynamoDB resources.
    - So it routes all S3 destined traffic to Gateway Endpoint
- Its highly available across all AZs in a region by default
- **Endpoint Policy** is used to control what Gateway Endpoint can access
- Gateway Endpoint isn't cross-region service. It can only access resources in the same region.
- Gateway Endpoints are only accessible from the VPC they were created in.

## 1.15.4 VPC Interface Endpoints
- Just like GW Endpoints, Interface Endpoints provide private access to AWS Public Services.
- When you create an Interface Endpoint for a particular service, you're getting new DNS name for that service.
- It allows AWS Services to be injected into VPC and be given Network Interfaces.
- Added to specific subnets - an ENI - not Highly Available
- Interface Endpoints use PrivateLink
- Support any service but DynamoDB (now it even support S3)

### 1.15.4.1 PrivateDNS
- Overrides the default DNS for services
- If you use default DNS name for SNS, it no longer resolves to its private IP address. It resolves to the private IP of Interface Endpoint. 
- Even services that can't use endpoint specific DNS name can use Interface Endpoints
- This option is enabled by default


## 1.15.5 VPC Peering
- Direct, encrypted network between two VPCs
- Works for both same region VPCs and Cross Region
- (optional) Public Hostnames can resolve to private IPs
- Same region SGs can reference peer SGs
- Routing is required and SGs and NACLs can filter traffic
- **MOST IMPORTANT:**
    - VPC Peering is not transitive - in order to connect 3 VPCs you need to create 3 vpc peering connections between each. If VPC 'A' is connected to VPC 'B' and VPC 'C' is connected to VPC 'B' - it doesn't mean that VPC 'A' can connect to VPC 'C'
    - VPCs CIDR ranges can't overlap
    - Route tables and SG rules at both sides are required (if they both want to communicate)

# 1.16 Hybrid Environments and Migration

## 1.16.1 BGP (Border Gateway Protocol)
- BGP is a path vector protocol. It exchanges best path to a destination between peers.
    - The best path is called **ASPATH**
    - BGP always advertises the shortes and it doesn't consider performance. Only path length matters.
        - AS Path Prepending can be used to artificially lengthen shortest paths to favor the longer ones with better performance. For example, 2-hop fiber connection vs satelite connection.
- **iBGP** = Internal BGP = Routing within AS
- **eBGP** = External BGP = Routing between AS's
    - **AS** = Autonomous System = Routers controlled by one entity (ISP network)
    - **ASN** = AS's number given by IANA
- BGP operates on Autonomous Systems, its all it sees. It routes based on them as black boxes, not hops/devices in it.
- This is how routes are learned and communicated when using AWS Direct Connect or Dynamic VPN.

## 1.16.2 IPSec VPN Fundamentals
- It's a group of protocols that enable secure connection between two peers
- Setsup secure connection across insecure network
    - It provides authentication and encryption between two peers (local and remote)

## 1.16.2 AWS Site-to-Site VPN 
- Required components for setting up Site-To-Site VPN (Both connect to each other)
    1) **Virtual Private Gateway**
        - VGW stands at VPC boundary and its routable like other gateways.
        - VGW is highly available by design as it operates on multiple physical endpoints across different AZs.
    2) **Customer Gateway** 
        - CGW - Customer Gateway is created in AWS and it represents physical device (router) on client on-prem side.
        - To make Site-To-Site VPN fully Highly Available, two Customer Gateways (physical routers) are required to be setup.
- Dynamic VPN uses BGP (router must support it)
- VPN can be used as a backup for Direct Connect 
- VPN can also be used with Direct Connect. Since DX might require months of configuration, VPN might be temporarily set up and then replaced.

### VPN Considerations
- VPN connection has speed limitations = around 1.25Gpbs (AWS limit) 
- VPN latency - might be inconsistent, it goes through public internet
- VPN Cost - AWS hourly cost, GB out cost
- Might be set up in hours or sometimes even less than hour. It's faster than any other private connections technologies.

## 1.16.3 Direct Connect

### 1.16.3.1 Direct Connect Concepts
- **Phyiscal Connection** - Business Premises => DX Location => AWS Region
- When you request DX, you basically request Port Allocation at a DX location and access to that port.
    - DX Location is not owned by AWS, its not AWS location nor building. AWS usually rent space there.
    - In DX location, there is DX router (AWS) and there must be Customer Router (yours)
    - You must handle connection between DX router and Customer Router
    - This connection is called cross-connect
- Public VIF = Interface that enables connecting to public AWS zone, public services
- Private VIF = Interface that enables connecting to a VPC and private IPs only


### Direct Connect (DX) Considerations
- Port Access is bille dhourly + Outbound Data transfer
- Provisioning time is very long, weeks or months. Physical cables must be set up.
- DX provides low & consistent latency and high speeds. (1, 10, 100 Gbps)

### 1.16.3.2 Direct Connect (DX) Resilience
- It's not resilient in any way by default. 
    - Routers can fail, whole DX location can fail, cables can be damaged

#### How to improve resilience:
- **Ok**: Order 2 Direct Connect connections. Each connection is made through separate DX device/port and Customer Router.
- **Better!**: Use two DX connection but each in different DX locations. Also, prepare separate physical locations for on premises routers (like different building). 
- **Extreme**: Combine 'Ok' and 'Better'. Use two DX connections per one separate DX location. Use two DX locations + 2 on premises locations.

## 1.16.4 Direct Connect (DX) - Public VIF + VPN (Encryption)
...

## 1.16.5 Transit Gateway
- Network Transit Hub to connect VPCs to on premises networks
- Significantly reduces network complexity
- TGW is a **single network object** - Highly Available and Scalable
- It allows **attachments** to other network types: VPC, Site-to-Site VPN & Direct Connect 
- Transit Gateway support **transitive routing**. All VPCs connected to this TGW can connect with each other
- Transit Gateway support **Cross-Region**, **Same Region** and **Cross-Account** VPC connection

## 1.16.6 Storage Gateway - Volume
For raw block storage, Gateway presents itself as block volumes.

- Storage Gateway is a Virtual Machine or hardware appliance which integrates with EBS, S3 and Glacier within AWS. 
- It helps with Migrations, Extensions (depends on mode), Storage Tiering, DR and Replacement of backups systems.
- It run cun in two modes
- **Raw block storage** - It doesn't provide any S3 interface or something. It presents storage using iSCSI, NFS or SMB.
- Connection is done via public internet or public VIF and Direct Connect

### 1.16.6.1 Volume Stored
1. **Local Storage** - (primary storage location for all volumes that STGW presents over iSCSI) Everything is stored locally, all volumes presented locally store data locally on premises.
2. **Upload Buffer** - Any data that is written to any volume is also written to this buffer temporarilly and copied asynchronously to AWS.

- Volume Stored type doesn't improve datacenter capacity. Main copy of data is stored on the gateway locally. So if you have no more capacity on prem, this wont help you as again - everything is stored locally
- Great for full disk backup and can assist with disaster recovery

### 1.16.6.2 Volume Cached
- Main storage location is in AWS, specifically S3 and it's cached locally
    - S3 is placed in AWS managed part of S3 (not public)
- This mode can be used to extend on-prem with cloud storage
- Good for frequently accessed data

## 1.16.6 Storage Gateway - Tape (VTL)
For tape storage, Gateway pretends to be tape backup system.

- Storage Gateway presents itself to Backup Server as Tape Library. It has media changed and tape drivers
- The Storage Gateway supports Virtual Tape Library (S3) and Tape Shelf (Glacier)
- It allows to extend on premises backup system
- Storage Gateway Tape (VTL) is used with on premises Backup Server to backup data

## 1.16.7 Sto-rage Gateway - File
- Bridges on-prem file storage and S3. Map directly onto an S3 bucket. Primary data is held in S3
    - File stored into a mount point on prem, are visible as objects in an S3 bucket. You have full visibility into the bucket.
- **Mounts Points** (shares) available via NFS or SMB = if you see these protocols mentioned in exam, default to File GW.
    - NFS = Linux, SMB = Windows
- File Gateway can store multiple **FileShares** and each FileShare is connected to one S3 bucket (bucket share). There is 10 bucket shares per File GW
    - It can read and interpret folders and directories and name S3 objects accordingly by using prefixes
- Multiple on-prem locations with FileShares can connect to one bucket in AWS. 
    - If Prem1 uploads files, Prem2 doesn't immediately see it after its saved into S3. He has to explicitly list new objects in S3. 

## 1.16.8 Snowball / Edge / Snowmobile

### 1.16.8.1 Snowball
- Contains **ONLY storage**
- Move large amounts of data IN and OUT of AWS
- Physical storage
- Ordered from AWS empty -> load up -> return
- Data encryption at rest using KMS
- IMPORTANT: 
    - If data is between 10TB to 10TB - then its more economical to transfer data using Snowball rather than over Internet
    - Support shipping multiple devices to multiple premises

### 1.16.8.2 Snowball Edge
- Contains both **Storage and Compute**
- **Compute Types**:
    - Storage Optimized: (with EC2) - 80TB, 24 vCPU, 32GB RAM, 1TB SSD
    - Computer Optimized: 100TB + 7.68 NVME, 52 vCPU, 208GB RAM
    - Compute with GPU: As above + GPU
- Larger capacity vs Snowball + its faster
- Should be used when you need to perform any operations on data as its ingested

### 1.16.8.3 Snowmobile
- Portable datacenter with shipping container on a truck
- Available only on special order
- Ideal for single location when 10PB+ is required
    - Not economical for multi-site
    - Not economical for data less than 10PB
- Up to 100PB per snowbile

## 1.16.8 Directory Service 
- Runs within VPC
- To implement HA, deploy into multiple AZs
- Some services need directory, e.g. Amazon Workspaces
- Can be integrated with on-prem directory
- 

### 1.16.8.1 Simple AD
- Cheapest, most simple and lightweight
- Based on open source Samba 4
- Designed to be used in isolation
    - Not designed to be used with any existing on premises systems
    - It's not full implementation like Microsoft AD

### 1.16.8.2 AWS Managed Microsoft AD
- Designed to be used with existing on premises system
    - Connected with VPN
- Actual implementation of Microsoft AD in AWS (2012 R2)
- Primary location is in AWS

### 1.16.8.3 AD Connector
- Proxy to integrate existing on premises AD with AWS
- It enables to use AWS Workspaces using on-prem AD environment
- Primary directory located on premises

## 1.16.9 AWS DataSync
- Can transfer data **in & out of AWS over network**
    - AWS services integration: EFS, FSX, S3
- **DataSync Agent** must be installed on premises. It integrates with on-prem storage using SMB/NFS. This Agent runs on virtualization platforms such as VMware and it communicated with AWS DataSync Endpoint
- Features: 
    - schedules,
    - bandwidth limits,
    - compression, 
    - encryption,
    - automatic recovery from transit failures,
    - built-in data validation

## 1.16.10 FSx for Windows File Server
- Single or Multi-AZ within VPC.
- Designed for integration with windows environments
    - Supports managed or self-managed AD (on prem)
    - It is native windows file system
- **EFS** (NFS) = Linux, **FSx** (SMB) = Windows
- Supports de-duplication (sub file), distributed file system (DFS), KMS at-rest encryption and enforced encryption in-transit

### Key Features:
- VSS - Support volume shadow copies (file level versioning). You can right click on file and get previous version
- Native file system accessible over SMB
- FSx uses Windows permission model
- Supports DFS (distributed file system) - scale-out file share structure
- Managed - no file server admin overhead

## 1.16.11 FSx For Lustre 
- Filesystem designed for high computing workloads
    - **Extreme performance** for scenarios such as Big Data, Machine Learning and Financial Modeling
- **LINUX ONLY**, supports POSIX
FSx for Lustre provides two deployment types: 
1. Scratch
    - Pure performance, short term, temp workloads.
    - No HA, no replication.
    - More chance of failure with larger file systems.
2. Persistent
    - Replication within one AZ only. Auto-heals on hardware failure. 
    - Long term workloads
    
With both of these deployment types you can backup to S3. Manual or automatic - 0-35 day retention. 3 = every 3 days, 0 = turned off.

## 1.16.12 AWS Transfer Family
- Thanks to various protocols support, external clients can connect to Transfer Family using protocols they support and which are connected to S3/EFS on AWS side.
- Managed file transfer service - supports transferring TO or FROM S3 and EFS. 
- Provides managed "servers" which support different protocols: FTP, FTPS, SFTP, AS2
- **MFTW** - Managed FIle Transfer Workflows. Serverless file workflow engine. It can run operations on files.
- AWS Transfer Family has authentication with identity provider Service, Directory service or custom one

### 1.16.12.1 Endpoint Types

#### 1.16.12.1.1 Public
- **Supported protocols**: SFTP
- DNS name should be used as its managed by AWS and IP can change
- **Security**: Can't access control via IP lists

#### 1.16.12.1.2 VPC-Internet
- **Supported protocols**: SFTP, FTPS
- It has static private IP address as the endpoint is located within VPC
- **Public**: It also has allocated static public Elastic IP and can be accessed from Internet 
- Can be accessed through VPN or Direct Connect
- **Security**: SGs and NACLs can be used to control access

#### 1.16.12.1.3 VPC-Internal
- **Supported protocols**: SFTP, FTPS, FTP
- It has static private IP address as the endpoint is located within VPC
- Can be accessed through VPN or Direct Connect
- **Security**: SGs and NACLs can be used to control access

# 1.17 Security, Deployment & Operations

## 1.17.1 AWS Secrets Manager 
- Supports automatic rotation (this uses lambda which needs permissions)
- Directly integrates with some AWS products (...RDS)
    - For example, you can auto rotate credentials in RDS (all managed by Secrets Manager).
- Secrets are encrypted using KMS
    - Role separation: Permission needed to both KMS and Secrets Manager

## 1.17.2 Application Layer (L7) Firewall 
- HTTPS encrypted connection is terminated on L7 Firewall. Data is inspected and then new encrypted connection with backend is created.
- Based on the software, it could even work with SMTP - this would mean visibility of email metadata and for plaintext emails the contents.

## 1.17.3 Web Application Firewall (WAF), WEBACLs, Rule Groups and Rules
- WAF can protect against Cross Site Scripting, SQL Injections and L7 attacks
- Can be added to API Gateway, CloudFront and ALB (WEB ACL)
- **WEB ACL**: Main configuration unit in WAF is WEB ACL that can be attached to services like CloudFront/ALB/APIGW
    - WEB ACLs work based on **Rules**
        - Rules have compute capacity (WCU - default max 1500)
        - Rules can be grouped into Role Groups
        - Rule types:
            1. **Regular**: With regular, you allow/block SSH traffic for example.
            2. **Rate-Based**: With rate-based you always decide what to do when rate is met.
- **Logs**: WAF can produce logs and send them to S3 (delayed), CW Logs and Firehose
- **Pricing**:
    - Monthly $5 for each WEB ACL
    - Monthly $1 for each rule
    - Monthly requests per WEB ACL ($0.60 for 1 milion requests)
    - There are also additional options that can be turned ON that come with extra cost

## 1.17.4 AWS Shield 
- It provides DDOS protection
- Work in two modes: Standard & Advanced

### Shield Standard (free)
- Provided with R53, CloudFront, AWS Global Accelerator
- protection at the perimeter - region/VPC or the AWS edge
- for common l3/l4 attacks
- working in the background, no extra actions required

### Shield Advanced (paid)
- it can protect API Gateway, ALB, NLB, CLB, anything associated with ENIs + all that standard supports
- cost $3,000 per month, per organisation with 1-year agreement (so 12x3,000) + charges for data out
- must be **explicilty enabled** in **Shield Advanced** or **AWS Firewall Manager Shield Advanced Policy**
- you also get **cost protection** - you receive refund for any unmitigated attacks that caused cost spikes
- it also provides **Proactive Engagemenet** with **AWS Shield Response Team**
    - This team will contact you if any attack is detected
    - You can also submit a ticket for them
- Shield Advanced provides **WAF integration**
- you also get **real time visibility** of DDOS events and attacks
- **health based detection** - required for proactive engagement team. These are application specific healtchecks
- you can create **protection groups** with requirements for any service to be added to it, then you can manage the group instead each service separately

## 1.17.5 CloudHSM
### Characteristics (+ differences between KMS)
- KMS runs on shared servers compared to HSM - this might be a big security concern for some companies. 
- HSM is **Single Tenant.** AWS provisioned but fully customer managed. 
- If you need **FIPS 140-2 Level 3** = You must use HSM, KMS supports only Level 2
- HSM is accessed by **PKCS#11**, **JCE** (Java Cryptography Extension), Microsoft CryptoNG. 
    - If EC2 wants to access HSM, **CloudHSM Client** must be installed.
- HSM doesn't provide any native integration with any AWS services.
- CloudHSM can be used for **server-side encryption**
### Use cases
- CloudHSM can **offload SSL/TLS** processing for Web Servers. So cryptographic operations will be run by CloudHSM not by EC2 server itself.
- Enable **Transparent Data Encryption** (TDE) for Oracle Databases
- If you run your **CA**, you can use CloudHSM to protect the private keys.

## 1.17.6 AWS Config
- **Default**: Keeps track of configuration changes over time on resource.
    - Who makes any changes, what the changes are, with what services it relates, security groups changes etc.
    - It stores all the data in S3 bucket to which customer has full access
    - It stores the data in **configuration item**
- **With additional Configuration**: It audits changes and check compliance with standards
    - **It doesn't prevent anything** and it doesn't block anything.
- Changes can generate SNS notifications and near real-time events via EventBridge & Lambda
- It's regonial service (can be configured as cross-region and cross-account)

## 1.17.7 AWS Macie
- Uses machine learning and pattern matching to discover and protect your sensitive data in AWS (for example, scan S3 bucket to find sensitive data)
- Amazon Macie **Discovery Job** (these run the 'scans') work with **Identifiers** which can be Managed by AWS or Custom managed by customer.
- Identifiers work based on keywords, regex, can find finance data, credentials, health information, PII etc.
    - Identifiers provide Policy Findings or Sensitive data findings.

## 1.17.8 Amazon Inspector
- Scans **EC2 instances**, the Instance OS, **ECS Containers**, **Lambda & Layers** and provides a report of findings ordered by priority
- Amazon Inspector can run with both **agent** an without agent. Agent provides additional OS visibility
- Inspector can check **Network Reachability** (no agent required)
- If you want to do any Host Assessment like checking for OS level vulnerabilities, you need to install the **inspector agent**

**Rules packages** which can be used in Inspector (**agent required**): 
1. CVE checking
2. CIS (Center for Internet Security) Benchmarks,
3. Security best practises for Amazon Inspector

## 1.17.9 Amazon GuardDuty
- It is **continous** security monitoring service
- Amazon GuardDuty **can learn what normal activity on account is**. It identifies unexpected and unauthorised activity
- It analyses supported **Data Sources**: 
    - DNS Logs
    - VPC Flow Logs
    - CloudTrail Event Logs
    - CloudTrail Management Events
    - CloudTrail Data Events
- Can be multi account

# 1.18 NOSQL Databases & DynamoDB 

## 1.18.1 DynamoDB Architecture
- NoSQL public DBaaS - Key/Value & Document
- **Capacity** (speed) is is set by **WCU** (write capacity units) and **RCU** (read capacity units)
- Highly resilient. Really fast, support **backups**, point in time recovery and encryption at rest.
    - On demand backup - full copy of table retained until removed. 
    - PITR (Point in time recovery) - it must be enabled on table. Its disabled by default. It continously record changes lets say every 35 days and lets you go back to specific moment in time.
- Support **event-driven integration** so you can do actions when data changes
- Billing based on WCU, RCU, Storage and features

## 1.18.2 Keys in DynamoDB
- Each item in the table must consist of Primary Key = Partition Key + optional Sort Key. If both used its reffered as Composite Key.
- **Partition key**: This is a simple primary key. If the table has only a partition key, then no two items can have the same partition key value.
- **Composite primary key**: This is a combination of partition key and sort key. If the table has a composite primary key, then two items might have the same partition key value. However, those items must have different sort key values.

## 1.18.2 DynamodDB - Operations, Consistency and Performance
### 1.18.2.1 Capacity Modes
- DynamoDB has two capacity modes which are configured per table: On-Demand and Provisioned.
    1. **On-demand** is for unknown and unpredictable workloads with low admin overhead. Might be sometimes 5x expensive as provisioned. WCU & RCU handled automatically.
    2. **Provisioned** is for well known, stable & predictable workloads. **You must set RCU and WCU by yourself.**
- You can switch tables from on-demand mode to provisioned capacity mode at any time.

### 1.18.2.2 RCU & WCU (!!)
- Every operation consumes at least 1 RCU/WCU. 
    - 1 RCU is 1 x 4KB read operation per second
    - 1 WCU is 1 x 1KB write operation per second
- If you retrieve 4 files, 1KB each - one RCU will be used.

#### Example 1 (RCU):
    You need to retrieve 10 ITEMS per second 2.5KB avg. size
    Round up avg. size, one RCU = 4KB so 1 RCU per item in this case is required = strongly consistent reads 10 RCU, for eventually consistent its 50% so 5 RCU
#### Example 2 (WCU):
    You need to store 10 ITEMS per second 2.5KB avg. size
    Round up avg. size, one WCU = 1KB so 3 WCU per item in this case is required = 10 x 3WCU = 30 WCU

### 1.18.2.3 Query vs Scan
- **Query** 
    - You query based on specific primary key Scan - moves through whole table consuming capacity of every item. 
    - You can't query only using a Sort Key. You need to specify a partition key to perform query operations. Else, you need to create a global secondary index or perform a scan operation
- **Scan** 
    - You can scan by any attributes
    - Scans are very expensive and slow for bigger tables

### 1.18.2.4 Consistency modes
- DynamoDB replicates data across three **Storage Nodes**. 
    - 2 Storage Nodes and 1 Leader Storage Node
- **Leader Storage Node** is elected from three storage nodes and it's used for writes. 
    - That's why writes in DynamoDB scale worse that reads. Because write is done on one Node while Reads on many (1 chosen randomly).

#### 1. Eventually Consistent
When using eventually consistent reads, the operation in run on random Storage Node so there is a small chance to get data from the not updated one.

#### 2. Strongly Consistent
When using strongly consistent reads, reads are always performed on Leader Storage Node so you will always have the latest results

## 1.18.3 DynamoDB Local and Global Secondary Indexes 
### 1. LSI
- LSI is an alternative view for a table
- Must be created with a table, can't be added later.
- Maximum 5 LSIs per table
- A local secondary index maintains an alternate sort key for a given partition key value
    - The data in a local secondary index is organized by the same partition key as the base table, but with a different sort key
- LSI shares the RCU and WCU with the table
- Contains a copy of some or all of the attributes from its base table
- The base table's primary key attributes are always projected into an index

### 2. GSI
- Can be created at any time
- Maximum 20 GSIs per table
- A global secondary index contains a selection of attributes from the base table, but they are organized by a primary key that is different from that of the table
    - It can have both different partition and sort key
- GSIs have their own RCU and WCU
- GSIs are always eventually consistent
- The base table's primary key attributes are always projected into an index

### 1.18.4 GSI and LSI projection

**KEYS_ONLY** – Each item in the index consists only of the table partition key and sort key values, plus the index key values. The KEYS_ONLY option results in the smallest possible secondary index.

**INCLUDE** – In addition to the attributes described in KEYS_ONLY, the secondary index will include other non-key attributes that you specify.

**ALL** – The secondary index includes all of the attributes from the source table. Because all of the table data is duplicated in the index, an ALL projection results in the largest possible secondary index.

## 1.18.4 DynamoDB Streams & Lambda Triggers
- Stream = Time ordered list of of item changes. Every time the item is changed, this information is saved and stored chronologically. 
- Streams have 24-hour rolling window.
- Enabled per table.
- Streams record **Inserts, Updates, Deletes**
- Provides different **view types** which influence what is in the stream
    - **KEYS_ONLY**: only changes to PK and SK. 
    - **NEW_IMAGE**: stores new, updated item
    - **OLD_IMAGE**: stores only previous version of item, db must be queried to see the what's changed
    - **NEW_AND_OLD_IMAGES**: stores both which enables better comparison
- DynamoDB Streams + Lambda Triggers
    - ITEM change generate an event which trigger lambda. That event contains the data which changes and action is taken using that data.
    - Lambda can be integrated to provide trigger functionality - invoking when new entries are added on the stream.

## 1.18.5 DynamoDB Global Tables
Global Table is a single DynamoDB table that functions as a part of a global table. Every replica has the same table name and the same primary key schema.

- Global Table should be seen as an entity. It's group of connected tables spread across regions.
    - Tables are created in multiple regions and added to the same global table (becoming replica tables)
- Provide multi-master replication, all tables can be used for Read and Write operations.
- Global eventual consistency. Same-region eventual or strongly consistent.
- Provides global HA and global DR/BC
- **Last writer wins** = if the same piece of data is being written to two tables at the same time, DynamoDB will pick most recent one and replicate it
- **How to create**: First you create regular DynamoDB table. Through its settings (global tables tab) you can create replicas in any region you want.

## 1.18.6 DynamoDB - Accelerator (DAX) 
- It's a fully managed, in-memory cache that speeds up the read and write performance of your DynamoDB tables
    - It can perform cache operations as data is being written to DynamoDB
- It can heavily speed up reads as it's integrated with DynamoDB
- Its a cluster service. It works on Primary Node (W/R) and Read Replicas and its deployed within VPC
- Supports caching items and query/scans results

## 1.18.7 DynamoDB TTL
Its a **per-item timestamp** which determines when an **item** is no longer needed. Shortly after the date and time of the specified timestamp, DynamoDB deletes the item from the table **without consuming any write throughput**. 
- When TTL is enabled on a table a specific attribute is selected for TTL. You must insert items with this attribute and specify TTL in seconds.

#### Cost
TTL is provided at no extra cost as a means to reduce stored data volumes by retaining only the items that remain current for your workload’s needs.

## 1.18.8 Amazon Athena
- Its an interactive serverless **Querying Service** that uses SQL language
- Allows ad-hoc queries on data
- Original data is not being changed
- Output can be sent to other services
- With Athena, you define Schema in which you define (you can create regular SQL tables) tables. Data flows through Schemas
    - Tables define how to get from the source data to desired table structure.
- No need to preload data to Athena thats why its great for Occasional/Ad-hoc queries on data in S3
    - Data is being fetched from source as the query is run
    - You pay only for data consumed
- Athena support **many file formats** like: JSON, XML, CSV/TSV, ORC, Apache, CloudTrail, VPC Flow Logs

### 1.18.8.1 Athena Federated Query
This must not be confused with regular Athena. Federated Query is basically extension that allows to use other non-S3 services with Athena.

- **Athena** = S3
- **Athena Federated Query** = Non S3 data sources that work based on **connectors** (DynamoDB, CW Logs, RDS)

## 1.18.9 Amazon ElastiCache
- ElastiCache is in-memory cache service. Can be used for building stateless, highly available and resilient applications and manage their sessions
- Good for read heavy workloads
- Provides sub-milisecond data access
- **Application changes are required** 

### 1.18.9.1 ElastiCache Engines

1. **Memcached**
- Simple data structures, strings only
- No replication
- No backups
- Multi-threaded by design

2. **Redis**
- Advanced data structures like lists
- Multi-AZ, Supports replication
- Provide backups & restore
- Transactions - multiple operations as one, all success or fail. Provides consistency

## 1.18.10 Amazon Redshift
- Petabyte-scale **Data Warehouse**
    - Data Warehouse is place where all your data sources can pump its data into for long data analysis. Its not for operational style usage
- **Redshift** = **OLAP** = Online Analytical Processing
- **RDS** = **OLTP** = Online Transaction Processing
- OLTP like RDS put their data into OLAP system like Redshift
- Access to data by columns not record
    - So column for all userId, column for all phoneNumbers
- **Its not serverless** like Athena. Its server based.
- Runs in one AZ only. 
- Works based on cluster. Most of the cluster is private. **Leader node** manages communication with client programs and all communication with **compute nodes**. 

#### Redshift Spectrum
Enables querying data directly on S3 without needing to load data into the Warehouse.

### Overcoming the single AZ risk
**S3 Backups** can help with resilience and recovery
    - Redshift can be configured to copy backups to another region for disaster recovery

### 1.18.10.1 Enhanced VPC Routin
By enabling Enhanced VPC Routing, it can be limited by Security Groups, it can use Gateways etc. It must be enabled if you have specific network design or requirements.


# 1.19  Machine Learning 101 

## 1.19.1 Amazon Comprehend
Amazon Comprehend is a natural-language processing (NLP) service that uses machine learning to uncover valuable insights and connections in text.
- It accepts document as an input and it provides the feedback:
    - Entities (people, companies), Key phrases, Lanugage, PII, Sentiment (negative, positive)
- Can be used through CLI, API, Console

## 1.19.2 Amazon Kendra
- Inteligent search service designed to mimic interacting with human expert
- Support wide range of question types
- Learns your data and provides answers based on that knowledge
- Index = searchable data organised in efficient way. This is what Kendra uses to provide answers
- Source = Where your data lives, Kendra connects and creates indexes from this location.
    - S3, Confluence, RDS, Google Workspaces, Salesforce, FSX, Workdocs...
- Integrates with many AWS Services like IAM, Identity Center...
- Backend service, mainly mainly used through API/CLI or integrated into Application (might be a search service backed by Kendra)

## 1.19.3 Amazon Lex
- Used for creating **interactive chatbots** (voice and text)
    - Chatbots, Voicebots, Q&A Bots
- Amazon **Lex powers Alexa service**
- Provides **Automatic Speech Recognition** (ASR) - speech to text
- **Natural Language Understanding** (NLU) - Intent, capable of connecting sentences
- Scales very well, quick to deploy, pay as you go
- Backend service, used mostly through API/CLI or integrated into Application

## 1.19.4 Amazon Polly
- Service that turns text into lifelike speech
    - Doesn't provide language translation
- It support Synthesis Markup Language (SML)
    - It provides additional control over how Polly generates speech
        - Emphasis, pronounciation, whispering
- Typically backend, mostly integrated with other things

## 1.19.5 Amazon Rekognition
- Deep learning Image and Video analysis
- It can recognize specific people, animals, faces
    - Can analyse live video stream using kinesis video streams
    - It can even provide game analysis on a football match
- **Billing**: Per image or per minute of video
- Integrates with AWS application & event driven apps
f
## 1.19.6 Amazon Textract
- Detect and and analyze text in input documents using optical character recognition
    - **Input**: JPEG, PDF, TIFF, PNG
    - **Examples: Receipts, driving licenses, tax documents
- Can be integrated with Applications and or other AWS Services and also used from console

## 1.19.7 Amazon Transcribe
- Speech to Text - Automatic speech recognition service
- Pay per use, per second of transcribed audio
- Can be used to create meeting notes
- Phone call analysis
- Can be integrated with other AWS Services and Applications

## 1.19.8 Amazon Translate
- Provides high-quality and customizable translation
- Provide semantic representation, meaning of sentence is translated
- Can auto-detect input language
- It can translate data from Comprehend, Polly, Transcribe stored in S3, RDS, DynamoDB

## 1.19.9 Amazon Forecast
- Forecasting for time-series data
- Based on data like retail demand, supply chain, staffing it can provide predictions
- Works based on historical & related data 
- Accessed through web console (visualization), CLI, APIs, Python SDK

## 1.19.10 Amazon Fraud Detector
- Fraud Detection for:
    - Online Fraud - new customer account
    - Transaction Fraud - transactional history, identifying suspect payments
    - Account Takeover - identify phishing or another social based attack
- All things are scored and rules/decision logic allows to react based on business activity

## 1.19.11 Amazon SageMaker
- Fully managed service that is a group of different products that help with Machine Learning lifecycle
- It provides:
    - **SageMaker Studio**: build, train, debug and monitor models. IDE for ML lifecycle
    - **SageMaker Domain**: EFS Volumes, Users, Apps, Policies, VPCs. Everything used to interact with the product
    - **Containers**: Deployed to ML EC2 instances to provide the environment
- It can host machine learning models

# 1.20 Other Services & Features

## 1.20.1 AWS Local Zones
- Local Zone can be seen as 1 additional zone in region near your location. Because they are closer, the provide lower latency.
- Not all services support local zones.
- Local Zones have their own compute and networking. 
- Direct Connect to Local Zone is supported
- Used when highest performance is required. 
- Parent Region is the region in which local zone is working.
    - For example, us-west-2 is Parent to local zone placed in Las Vegas us-west-2-las-1
- By creating subnets in local zones, you can extend your VPC 

## 1.20.2 AWS Lake Formation
- A data lake is a centralized repository that allows you to store all your structured and unstructured data at any scale
    - Inside data lake, you can perform many actions like:
        - ETL, source crawlers, analysis, security findings
- It can ingest data from various sources like S3, RDS, DynamoDB

## 1.20.3 AWS Artifact
- AWS Artifact is central resource for compliance-related information
- It provides on-demand access to security and compliance reports from AWS and Software Vendors who sell their products on AWS Marketplace

## 1.20.4 Amazon EMR (formerly MapReduce)
- Platform for running **big data** operations

## 1.20.5 Beanstalk
- Layer of abstraction away from the EC2
- More managed way of running instaces
    - It will setup an "environment" for you that can contain a number of EC2 instances, an optional database, as well as a few other AWS components such as a Elastic Load Balancer, Auto-Scaling Group, Security Group

## 1.20.6 Amazon QuickSight

# 1.21 Infrastructure as Code - CloudFormation 

## 1.21.1 CloudFormation Physical & Logical Resources 
- CloudFormation Template - YAML or JSON document which contains logical resources (WHAT).
- Stacks create **physical** resources based on defined **logical** resources.
- If a stacks template is changed - physical resources are changed
- If a stack is deleted, normally, the physical resources are deleted

## 1.21.2 Template Parameters & Pseudo Parameters
- **Template Parameters** can be defined inside the template and then provided when stack is created or updated via CLI or Console
    - Parameters can be referenced from within the template
- **Template Parameters** can be configured with **Defaults**, Min and Max, Allowed Patterns and even list of allowed values (dropdown)
- **Pseudo Parameters** are injected/populated into stack when creating a stack. For example, region name

## 1.21.3 


- Amazon Quantum Ledger Database (QLDB)
    - NoSQL transparent transaction database. Fully managed, immutable, cryptographicly secured.
- **Neptune**
    - Fully managed Graph database. Milisecond latency and bilions of relationships. Support read replicas across 3 AZs. ACID compliant, scalable. Neptune Streams is a feature within Amazon Neptune that allows for the capture of changes made to graph data in real time
- **Keyspaces**
    - Distributed, scalable, schemaless, column-oriented, NoSQL db. It helps to run Apache Cassandra workloads more easily. 
- **FSx types**
    - NetApp ONTAP supports NFS + other like SMB and iSCSI
    - OpenZFS support just NFS
- Amazon Managed Streaming for Apache Kafka 
    - MSK can be used for Apache Kafka without managing cluster capacity. It helps to run the tool, just like AWS Keyspaces. Message Broker suited for high workloads.
- **Opensearch**
    - Real-time application analytics, interactive log analytics, website search. Designed for huge scale workloads. Automatically scales, instals, configures. Customer have full control over cluster configuration.
- **QuickSight**
    - Connects various data sources into interactive dashboards for Business Inteligence. Used for analytics. Can be embeded into your application and even accessed from phone. Supports Machine Learning insights.
- Outposts
    - You can request a physical compute rack from AWS which will be delivered to you, setup, and connected to closest AWS Region using Direct Connect or VPN.
- Serverless Application Repository
    - Managed repository for serverless applications. It enables teams, and developers to store and share reusable applications, and easily assemble and deploy serverless architectures. No need to clone, build, package, or publish source code to AWS before deploying it. Instead, you can use pre-built applications from the Serverless Application Repository in your serverless architectures
- Device Farm
    - Application testing service that lets you improve the quality of your web and mobile apps by testing them across an extensive range of desktop browsers and real mobile devices; without having to provision and manage any testing infrastructure. The service enables you to run your tests concurrently on multiple desktop browsers or real devices to speed up the execution of your test suite, and generates videos and logs to help you quickly identify issues with your app.
- **Managed Grafana**
    - Powerful Interactive data visualizations and analytics. 
- **Managed Service for Prometheus**
    - Monitoring and alerting functionality for containerized systems. Can be visualised, graphed, queried in Grafana (setup AMP as source).
- AWS Proton
    - Deployment workflow tool for modern applications that helps platform and DevOps engineers achieve organizational agility. Central dashboard for application management. Uses CFN underhood.
- AWS Service Catalog
    - With AWS Service Catalog and AWS Proton you can pre-define set of CFN templates, and allow your users and developers to easily provision those. It also provides clear role separation in your account. You have users which manage infrastructure and are administrators, while the other ones are normal users/developers.
