# Design an AWS VPC

Designing a VPC (Virtual Private Cloud) is an essential skill for cloud architects, DevOps engineers, and anyone working with cloud environments. Here are several reasons why it is important to know how to design a VPC:

**1. Security and Isolation** -> a well-designed VPC provides network isolation and security for the resources. We can control inbound and outbound traffic using security groups, Network Access Control Lists (NACLs), and private / public subnets. By properly designing our VPC, we can ensure that sensitive resources, such as databases, are kept private while still allowing public access to the web servers.

**2. Scalability and High Availability** -> VPC design impacts the ability to scale our infrastructure as needed. Properly designed subnets, CIDR blocks, and availability zones can ensure high availability for our applications, making them resilient to failures and capable of handling increased demand.

**3. Performance Optimization** -> designing the VPC with consideration for network traffic flow, routing, and placement of resources within the availability zones can help optimize application performance. Efficient routing tables and minimized latency between resources can significantly impact application response times.

**4. Cost Management** -> proper VPC design helps minimize unnecessary costs by ensuring efficient use of networking resources (e.g., NAT Gateways, data transfer charges). Keeping resources within private subnets reduces exposure and may lower costs for certain use cases.
**5. Compliance Requirements** -> many industries have strict compliance and data protection requirements (e.g., GDPR, HIPAA). Designing a VPC allows us to enforce access controls, logging, and secure data transfer practices to meet compliance needs.

**6. Flexible Connectivity Options** -> we can design the VPC to securely connect with on-premises data centers via VPNs, AWS Direct Connect, and other methods. This allows us to build hybrid cloud solutions. Understanding the VPC design helps us to implement peering connections, private link services, and secure cross-account access if needed.

**7. Network Traffic Control and Monitoring** -> with proper VPC design, we can control and monitor traffic using flow logs, AWS CloudTrail, and other tools. This helps detect potential security breaches, monitor network performance, and identify any misconfigurations.

**8. Support for Multi-Tier Architectures** -> designing a VPC allows us to implement multi-tier architectures (e.g., web, application, database tiers) securely. By using separate subnets and security controls for each tier, we can enforce least privilege and reduce risk.

**9. Disaster Recovery Planning** -> a well-designed VPC can play a critical role in disaster recovery plans. By replicating resources across multiple regions and availability zones, we can ensure your systems remain operational during outages or failures.

**10. Understanding AWS Networking Best Practices** -> proper VPC design reflects a deeper understanding of AWS networking services and concepts such as route tables, subnets, Internet Gateways, NAT Gateways, and VPC Peering. This knowledge is critical for working efficiently in AWS and for troubleshooting networking issues.

Designing a VPC involves making sure that its essential components are working efficiently together. The objective of this assignment is to design and implement a secure and highly available VPC architecture for Cloud Growth Limited's web application using AWS services. The basic architecture will consist of a public-facing web component and a backend database, ensuring that the web component is accessible from the Internet while the database remains accessible only within the VPC.

**1.** First of all, let's identify its **CIDR (Classless Inter-Domain Routing) block.** CIDR represents a range of IP addresses that we assign to our VPC when we create it in AWS. This range defines the network space in which all subnets, resources, and instances within the VPC will operate and communicate. It serves as the primary network boundary for our cloud infrastructure, enabling us to segment and organize resources. The CIDR block for a VPC determines the range of IP addresses available for your AWS resources. Proper planning of the CIDR block ensures efficient IP address utilization, scalability, and network isolation. 

VPC CIDR Block: 192.168.0.0/16 (as an example). This block provides a large address range with 65,536 private IP addresses, ensuring scalability for future growth.

**2. VPC will span two availability zones**, providing better resiliency, failover capabilities, and optimized resources distribution. We will create two availability zones: Availability Zone 1 and Availability Zone 2.

**3. VPC will have six subnets: two public subnets (one in each AZ) and four private subnets (two in each AZ).** Public and private subnets are needed to enhance security and control access to resources within a VPC
* **Public Subnets:**
    * Public Subnet 1 (AZ 1): 192.168.1.0/24
    * Public Subnet 2 (AZ 2): 192.168.2.0/24

These subnets will host services that require Internet access, such as web servers and NAT Gateway, ensuring they can communicate with the Internet through an Internet Gateway (IGW).
* **Private Subnets:**
    * Private Subnet 1 (AZ 1): 192.168.3.0/24
    * Private Subnet 2 (AZ 1): 192.168.4.0/24
    * Private Subnet 3 (AZ 2): 192.168.5.0/24
    * Private Subnet 4 (AZ 2): 192.168.6.0/24

These subnets will host application servers and databases, ensuring they remain private and secure while ensuring redundancy and maintaining high availability across multiple AZs.
This subnets separation allows us to securely expose only necessary resources while keeping others protected within the VPC.

**4. Internet Connectivity** of the VPC will be handled through the following components:
* **Internet Gateway (IGW):**
    * Internet Gateway is atttached to the VPC
    * Public subnets are configured so that they have routes pointing to the Internet Gateway for outbound Internet access.

* **NAT Gateway:**
    * NAT Gateway is deployed in one of the public subnets (e.g., Public Subnet 1).
    * Private subnets' route tables are configured to use the NAT Gateway for outbound Internet access. This ensures that instances in private subnets can access the Internet for updates without being directly accessible from the Internet.

**5. Route tables** are needed to be configured properly as following:
* **Public Route Table:**
Public subnets are associated with a route table that has a route to the Internet Gateway (0.0.0.0/0 pointing to the IGW).

* **Private Route Table:**
Private subnets are associated with the route tables that have a route to the NAT Gateway (0.0.0.0/0 pointing to the NAT Gateway).
It is important to ensure that internal communication within the VPC for the application servers and databases is **properly routed.** For this to happen, among other requirements' configuration, we need to make sure that private route tables are configured in a right way:
1. Create and associate **separate route tables for the subnets hosting our application servers and databases.**
For internal communication within the VPC, we need to ensure there are routes in the private route table that allow communication between the subnets. In AWS, this is automatically handled since all subnets within a VPC can communicate by default (using private IP addresses).
2. There is no need for internet routes, meaning that for the internal communication, we don't need to add routes to the Internet Gateway (IGW) or NAT Gateway in the private route tables, unless instances in these subnets need to make outbound Internet requests.

**6. Security configuration** of the VPC is established throuth the following components:
* **Network Access Control Lists (NACLs):**
    * NACLs are configured to allow inbound and outbound traffic for the public subnets as required (e.g., HTTP/HTTPS traffic).
    * NACLs are configured stricter for the private subnets, allowing only necessary traffic for communication with the NAT Gateway, web servers, and database.

* **Security Groups:**
* Web Server Security Group (for instances in public subnets):
    * Allows inbound traffic on port 80 (HTTP) and port 443 (HTTPS) from the Internet.
    * Allows outbound traffic for communication with private subnets.

* Application Server Security Group (for instances in private subnets):
    * Allows inbound traffic from web servers (using the web server security group as the source) on required ports (e.g., port 80 for HTTP traffic or a custom port).
    * Allows outbound traffic to databases.

* Database Security Group:
    * Allows inbound traffic only from the application servers (using the application server security group as the source).
    * Denies all other inbound access.

I would like to highlight the following key aspects of this architecture. Web servers (in public subnets) can communicate with application servers (in private subnets) directly through private IP addresses. This communication happens over the VPC's internal network, without going through any external entity, such as a NAT Gateway or the Internet Gateway. The communication happens in the following way: the web server sends requests to the application server using its private IP address; the application server processes the request and may send a response back to the web server over the VPC's internal network. Communication can be controlled using security groups. For example, the web server's security group can allow inbound traffic from any source (for HTTP/HTTPS traffic) but allow outbound traffic only to the private subnets' application servers. The application server's security group can allow inbound traffic only from the web server's security group.

Web servers are typically deployed in public subnets and are directly accessible from the Internet through an Internet Gateway (IGW). They do not need to use a NAT Gateway because they already have direct Internet access.
The primary role of a NAT (Network Address Translation) Gateway is to enable instances in private subnets (e.g., application servers or databases) to initiate outbound connections to the Internet (for tasks like downloading updates or accessing external services) while preventing inbound traffic from the Internet.

The main features of this basic design of the VPC are the following: 
* High availability is achieved by deploying resources across multiple AZs.
* Security is enforced by using security groups and NACLs to control access.
* Private subnets ensure the database is isolated and secure, accessible only by the application servers.
* The NAT Gateway allows secure outbound Internet access for instances in private subnets.