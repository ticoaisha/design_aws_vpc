In order to design AWS VPC, first of all, let identify its CIDR (Classless Inter-Domain Routing) block. CIDR represents a range of IP addresses that we assign to our VPC when we create it in AWS. This range defines the network space in which all subnets, resources, and instances within the VPC will operate and communicate. It serves as the primary network boundary for our cloud infrastructure, enabling us to segment and organize resources.

1. VPC CIDR Block: 192.168.0.0/16. This block provides a large address range with 65,536 private IP addresses, ensuring scalability for future growth.

2. In order to achieve high availability, we will create two availability zones: Availability Zone 1 and Availability Zone 2. 

3. VPC will have 6 subnets: two public subnets (one in each AZ) and four private subnets (two in each AZ).
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

4. Internet Connectivity through the following components:
**Internet Gateway (IGW):**
* Internet Gateway is atttached to the VPC
* Public subnets are configured so that they have routes pointing to the Internet Gateway for outbound Internet access.
**NAT Gateway:**
* NAT Gateway is deployed in one of the public subnets (e.g., Public Subnet 1).
* Private subnets' route tables are configured to use the NAT Gateway for outbound Internet access. This ensures that instances in private subnets can access the Internet for updates without being directly accessible from the Internet.

5. Route Tables Configuration:
**Public Route Table:**
Public subnets are associated with a route table that has a route to the Internet Gateway (0.0.0.0/0 pointing to the IGW).
**Private Route Table:**
Private subnets are associated with the route tables that have a route to the NAT Gateway (0.0.0.0/0 pointing to the NAT Gateway).
Ensure internal communication within the VPC for the application servers and databases is properly routed. For this to happen, among other requirements' configuration, we need to make sure that private route tables are configured in a right way:
1. Create and associate separate route tables for the subnets hosting your application servers and databases.
For internal communication within the VPC, ensure there are routes in the private route table that allow communication between the subnets. In AWS, this is automatically handled since all subnets within a VPC can communicate by default (using private IP addresses).
2. No Need for Internet Routes: For internal communication, you don't need to add routes to the Internet Gateway (IGW) or NAT Gateway in the private route tables, unless instances in these subnets need to make outbound Internet requests.

6. Security Configuration:
**Network Access Control Lists (NACLs):**
* NACLs are configured to allow inbound and outbound traffic for the public subnets as required (e.g., HTTP/HTTPS traffic).
* NACLs are configured stricter for the private subnets, allowing only necessary traffic for communication with the NAT Gateway, web servers, and database.

**Security Groups:**
* Web Server Security Group (for instances in public subnets):
    * Allow inbound traffic on port 80 (HTTP) and port 443 (HTTPS) from the Internet.
    * Allow outbound traffic for communication with private subnets.

* Application Server Security Group (for instances in private subnets):
    * Allow inbound traffic from web servers (using the web server security group as the source) on required ports (e.g., port 80 for HTTP traffic or a custom port).
    * Allow outbound traffic to databases.

* Database Security Group:
    * Allow inbound traffic only from the application servers (using the application server security group as the source).
    * Deny all other inbound access.

Rationale and Assumptions:

High availability is achieved by deploying resources across multiple AZs.
Security is enforced by using security groups and NACLs to control access.
Private subnets ensure the database is isolated and secure, accessible only by the application servers.
The NAT Gateway allows secure outbound Internet access for instances in private subnets.