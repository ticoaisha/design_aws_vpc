In order to design AWS VPC, first of all, let identify its CIDR (Classless Inter-Domain Routing) block. CIDR represents a range of IP addresses that we assign to our VPC when we create it in AWS. This range defines the network space in which all subnets, resources, and instances within the VPC will operate and communicate. It serves as the primary network boundary for our cloud infrastructure, enabling us to segment and organize resources.

1. VPC CIDR Block: 192.168.0.0/16. This block provides a large address range with 65,536 private IP addresses, ensuring scalability for future growth.
2. In order to achieve high availability, we will create two availability zones: Availability Zone 1 and Availability Zone 2. 
2. VPC will have 6 subnets: two public subnets (one in each AZ) and four private subnets (two in each AZ).



Public Subnets:

Create two public subnets:
Public Subnet 1 (e.g., 10.0.1.0/24) in Availability Zone (AZ) 1.
Public Subnet 2 (e.g., 10.0.2.0/24) in AZ 2.
These subnets will host services that require Internet access, such as web servers and NAT Gateways, ensuring high availability.
Private Subnets:

Create four private subnets:
Private Subnet 1 (e.g., 10.0.3.0/24) and Private Subnet 2 (e.g., 10.0.4.0/24) in AZ 1.
Private Subnet 3 (e.g., 10.0.5.0/24) and Private Subnet 4 (e.g., 10.0.6.0/24) in AZ 2.
These subnets will host the application servers and databases, ensuring redundancy and high availability.
3. Internet Connectivity
Internet Gateway (IGW):

Attach an Internet Gateway to the VPC.
Configure the public subnets to have routes pointing to the Internet Gateway for outbound Internet access.
NAT Gateway:

Deploy a NAT Gateway in one of the public subnets (e.g., Public Subnet 1).
Configure the private subnets' route tables to use the NAT Gateway for outbound Internet access. This ensures that instances in private subnets can access the Internet for updates without being directly accessible from the Internet.
4. Route Tables Configuration
Public Route Table:
Associate the public subnets with a route table that has a route to the Internet Gateway (0.0.0.0/0 pointing to the IGW).
Private Route Table:
Associate the private subnets with route tables that have a route to the NAT Gateway (0.0.0.0/0 pointing to the NAT Gateway).
Ensure internal communication within the VPC for the application servers and databases is properly routed.
5. Security Configuration
Network Access Control Lists (NACLs):

Configure NACLs to allow inbound and outbound traffic for the public subnets as required (e.g., HTTP/HTTPS traffic).
Configure stricter NACLs for the private subnets, allowing only necessary traffic for communication with the NAT Gateway, web servers, and database.
Security Groups:

Web Server Security Group (for instances in public subnets):
Allow inbound traffic on port 80 (HTTP) and port 443 (HTTPS) from the Internet.
Allow outbound traffic for communication with private subnets.
Application Server Security Group (for instances in private subnets):
Allow inbound traffic from web servers (using the web server security group as the source) on required ports (e.g., port 80 for HTTP traffic or a custom port).
Allow outbound traffic to databases.
Database Security Group:
Allow inbound traffic only from the application servers (using the application server security group as the source).
Deny all other inbound access.
6. Documentation
Architecture Diagram:

Create a diagram showing:
VPC and CIDR block.
Public and private subnets in two AZs.
Internet Gateway.
NAT Gateway in one public subnet.
Route tables and their associations.
Security groups and NACLs.
Rationale and Assumptions:

High availability is achieved by deploying resources across multiple AZs.
Security is enforced by using security groups and NACLs to control access.
Private subnets ensure the database is isolated and secure, accessible only by the application servers.
The NAT Gateway allows secure outbound Internet access for instances in private subnets.