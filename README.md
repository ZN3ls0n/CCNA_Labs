# Lab Objective

This is the first of a multi-phase lab of building and simulating an enterprise network. In this first phase, I implemented secure device management practices and validated connectiovity. 
This will be the foundation of lbs going forward. 

The lab simulates two network connected sites - a branch office and a remote office - with routers and access switches. I enabled administrative access on the switches and routers so users with authorized access can access them. 
Administrative access to network devices is secured using console protection, privilege mode security, and SSH remote access restricted to a management VLAN. Realistically, this is not enough to secure your physical network. 
In this lab, routers are managed through in-band interfaces due to the absence of dedicated management ports. SSH access is restricted to the management VLAN to reduce exposure. Future improvements will include stronger access control mechanisms to further secure device management.

# Network Design
The topology consists of two LAN environments connected by a site-to-site router WAN link. Each site contains:
- 1 router
- 2 access switches
- 3 PCs
-   2 PCs are in Management VLANs
-   1 PC is in data subnet

# Security Configuration

## Console Security
Console access was secured using a password to prevent unauthorized physical access. Passwords for each switch and router are presented in the topology, provided with switch and router names
<pre><code>line console 0
password {password}
login</code></pre>

## Privileged EXEC Security
Privileged mode was secured using an enable secret password. Enter global configuration <code>configure t</code>:
<pre><code>enable secret 0 {password}</code></pre>
Using the <code>enable secret 0</code> encrypts your password, which is entered originally as plaintext. This prevents unuathoirzed users from modifying decice configurations.

## Secure Remote Access (SSH)
Remote management access was configured using SSH.

Configuration steps include 
- Creating a domain name for SSH to create RSA keys for the encrypted session and as a way for the hosts to connect to the SSH server
- VTY lines are the virtual terminal lines that allow for remote access
- The server will accept SSH traffic only for remote management, not telnet (insecure, clear-text)
- Create a username and password to start the SSH session
- Allow it for authentication

<pre><code>ip domain-name {domain-name}
username {configured_user} secret 0 {configured_passwd}
crypto key generate rsa
# You will be asked for your modulus key size
line vty 0-4
transport input ssh
login local</code></pre>

## Management VLAN
Each switch has a VLAN designed solely fopr management access. Only devices connected to the ports within the managemnt VLAN have access for SSH management. Such devices will not have access to the internet or to reach other devices within the internal network. <b>This is not enough for physical security. Future improvements will include additional protections to prevent unauthorized devices from joining the management network.</b> This will include port security.

## Connectivity Testing
After configuring the network devices, connectivity was verified using ICMP echo requests. 
- Host to default gateway
- Router to router connectviity
- Lack of Internet connectivity for MGMT VLAN
- Inter-host connectivity between branch and remote networks.



