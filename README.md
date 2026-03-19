# Lab Objective

This is the first of a multi-phase lab of building and simulating an enterprise network. In this first phase, I implemented secure device management practices and validated connectiovity. 
This will be the foundation of lbs going forward. 

The lab simulates two network connected sites - a branch office and a remote office - with routers and access switches. I enabled administrative access on the switches and routers so users with authorized access can access them. 
Administrative access to network devices is secured using console protection, privilege mode security, and SSH remote access restricted to a Management VLAN. Realistically, this is not enough to secure your physical network. 
In this lab, routers are managed through in-band interfaces due to the absence of dedicated management ports. SSH access is restricted to the Management VLAN to reduce exposure. Future improvements will include stronger access control mechanisms to further secure device management.

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
Each switch has a VLAN designed solely fopr management access. Only devices connected to the ports within the managemnt VLAN have access for SSH management. In this lab, management VLAN traffic is intentionally restricted from reaching other internal and external networks. <b>This is not enough for physical security. Future improvements will include additional protections to prevent unauthorized devices from joining the management network.</b> This will include port security.

## Connectivity Testing
After configuring the network devices, connectivity was verified using ICMP echo requests. 
- Host to default gateway
- Router to router connectviity
- Lack of Internet connectivity for MGMT VLAN
- Inter-host connectivity between branch and remote networks.

<table>
        <thead>
            <th colspan="4">Router Interfaces (Branch Office)</th>
        </thead>
        <tbody>
            <tr>
                <td>Interface</td>
                <td>Network</td>
                <td>IP Address</td>
                <td>Description</td>
            </tr>
            <tr>
                <td>g0/0</td>
                <td>10.10.10.0/30</td>
                <td>10.10.10.1</td>
                <td>WAN to Remote</td>
            </tr>
            <tr>
                <td>g0/1</td>
                <td>192.168.10.0/24</td>
                <td>192.168.10.1</td>
                <td>Branch network access gateway</td>
            </tr>
            <tr>
                <td>g0/2</td>
                <td>192.168.20.0/24</td>
                <td>192.168.20.1</td>
                <td>Branch network access gateway</td>
            </tr>
        </tbody>
    </table>


  <table>
        <thead>
            <th colspan="4">Router Interfaces (Remote Office)</th>
        </thead>
        <tbody>
            <tr>
                <td>Interface</td>
                <td>Network</td>
                <td>IP Address</td>
                <td>Description</td>
            </tr>
             <tr>
                <td>g0/0</td>
                <td>10.10.10.0/30</td>
                <td>10.10.10.2</td>
                <td>WAN to Branch</td>
            </tr>
            <tr>
                <td>g0/1</td>
                <td>192.168.30.0/24</td>
                <td>192.168.30.1</td>
                <td>Remote access network gateway</td>
            </tr>
            <tr>
                <td>g0/2</td>
                <td>192.168.40.0/24</td>
                <td>192.168.40.1</td>
                <td>Remote access network gateway</td>
            </tr>
        </tbody>
    </table>

  <table>
        <tbody>
            <th colspan="4">BR-SW-1 Interfaces</th>
            <tr>
                <td>Interface</td>
                <td>Mode</td>
                <td>VLAN</td>
                <td>Description</td>
            </tr>
            <tr>
                <td>fa0/1</td>
                <td>Access</td>
                <td>Data</td>
                <td>Uplink to Branch Router (g0/1)</td>
            </tr>
            <tr>
                <td>fa0/2</td>
                <td>Access</td>
                <td>10</td>
                <td>MGMT-PC-A</td>
            </tr>
            <tr>
                <td>fa0/3</td>
                <td>Access</td>
                <td>10</td>
                <td>MGMT-PC-B</td>
            </tr>
            <tr>
                <td>fa0/7</td>
                <td>Access</td>
                <td>Data</td>
                <td>Client 1</td>
            </tr>
        </tbody>
    </table>

  <table>
        <tbody>
            <th colspan="4">BR-SW-2 Interfaces</th>
            <tr>
                <td>Interface</td>
                <td>Mode</td>
                <td>VLAN</td>
                <td>Description</td>
            </tr>
            <tr>
                <td>fa0/1</td>
                <td>Access</td>
                <td>20</td>
                <td>MGMT-PC-2</td>
            </tr>
            <tr>
                <td>fa0/2</td>
                <td>Access</td>
                <td>20</td>
                <td>MGMT-PC-1</td>
            </tr>
            <tr>
                <td>g0/2</td>
                <td>Access</td>
                <td>Data</td>
                <td>Uplink to Branch Router (g0/2)</td>
            </tr>
            <tr>
                <td>fa0/5</td>
                <td>Access</td>
                <td>Data</td>
                <td>Client 2</td>
            </tr>
        </tbody>
    </table>

  <table>
        <tbody>
            <th colspan="4">RE-SW-1 Interfaces</th>
            <tr>
                <td>Interface</td>
                <td>Mode</td>
                <td>VLAN</td>
                <td>Description</td>
            </tr>
            <tr>
                <td>fa0/1</td>
                <td>Access</td>
                <td>Data</td>
                <td>Uplink to Remote Router (g0/1)</td>
            </tr>
            <tr>
                <td>fa0/2</td>
                <td>Access</td>
                <td>30</td>
                <td>MGMT-PC-C</td>
            </tr>
            <tr>
                <td>fa0/3</td>
                <td>Access</td>
                <td>30</td>
                <td>MGMT-PC-D</td>
            </tr>
            <tr>
                <td>fa0/6</td>
                <td>Access</td>
                <td>Data</td>
                <td>Client 3</td>
            </tr>
        </tbody>
    </table>

  <table>
        <tbody>
            <th colspan="4">RE-SW-2 Interfaces</th>
            <tr>
                <td>Interface</td>
                <td>Mode</td>
                <td>VLAN</td>
                <td>Description</td>
            </tr>
            <tr>
                <td>fa0/1</td>
                <td>Access</td>
                <td>40</td>
                <td>MGMT-PC-4</td>
            </tr>
            <tr>
                <td>fa0/4</td>
                <td>Access</td>
                <td>40</td>
                <td>MGMT-PC-3</td>
            </tr>
            <tr>
                <td>fa0/2</td>
                <td>Access</td>
                <td>Data</td>
                <td>Client 4</td>
            </tr>
            <tr>
                <td>g0/1</td>
                <td>Access</td>
                <td>Data</td>
                <td>Uplink to Remote Router (g0/2)</td>
            </tr>
        </tbody>
    </table>

  <table>
        <tbody>
            <th colspan="5">Host IP Addresses</th>
            <tr>
                <td>Network</td>
                <td>Device Name</td>
                <td>LAN</td>
                <td>Subnet</td>
                <td>IP Address</td>
            </tr>

  <tr>
                <td rowspan="6">Branch Network</td>
                <td>MGMT-PC-A</td>
                <td>BR-SW-1</td>
                <td>172.16.10.0/24</td>
                <td>172.16.10.5</td>
            </tr>

  <tr>
                <td>MGMT-PC-B</td>
                <td>BR-SW-1</td>
                <td>172.16.10.0/24</td>
                <td>172.16.10.20</td>
            </tr>

<tr>
                <td>Client 1</td>
                <td>BR-SW-1</td>
                <td>192.168.10.0/24</td>
                <td>192.168.10.20</td>
            </tr>

  <tr>
                <td>MGMT-PC-1</td>
                <td>BR-SW-2</td>
                <td>172.16.30.0/24</td>
                <td>172.16.30.25</td>
            </tr>

  <tr>
                <td>MGMT-PC-2</td>
                <td>BR-SW-2</td>
                <td>172.16.30.0/24</td>
                <td>172.16.30.20</td>
            </tr>

  <tr>
                <td>Client 2</td>
                <td>BR-SW-2</td>
                <td>192.168.20.0/24</td>
                <td>192.168.20.2</td>
            </tr>

  <tr>
                <td rowspan="6">Remote Network</td>
                <td>MGMT-PC-C</td>
                <td>RE-SW-1</td>
                <td>172.16.100.0/24</td>
                <td>172.16.100.5</td>
            </tr>

  <tr>
            <td>MGMT-PC-D</td>
            <td>RE-SW-1</td>
            <td>172.16.100.0/24</td>
            <td>172.16.100.20</td>
        </tr>

  <tr>
            <td>Client 3</td>
            <td>RE-SW-1</td>
            <td>192.168.30.0/24</td>
            <td>192.168.30.2</td>
        </tr>

  <tr>
            <td>MGMT-PC-3</td>
            <td>RE-SW-2</td>
            <td>172.18.10.0/24</td>
            <td>172.18.10.4</td>
        </tr>

  <tr>
            <td>MGMT-PC-4</td>
            <td>RE-SW-2</td>
            <td>172.18.10.0/24</td>
            <td>172.18.10.2</td>
        </tr>

  <tr>
            <td>Client 4</td>
            <td>RE-SW-2</td>
            <td>192.168.40.0/24</td>
            <td>192.168.40.10</td>
        </tr>
        </tbody>
    </table>
            
       

            


