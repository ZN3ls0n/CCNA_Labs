# Lab Objective

This lab is part of a multi-phase enterprise network simulation. The goal is to implement access controls that:
<ul>
<li>Restrict management access to authorized devices only</li>
<li>Prevent unauthorized devices from connecting to the network</li>
<li>Control traffic flow between data and management subnets</li>
</ul>
The lab simulates two network connected sites - a branch office and a remote office - with routers and access switches. The login information for switches and routers remain the same.

# Network Design
Each site includes:
<ul>
<li>1 router</li>
<ul>
<li>Each router has subinterfaces, allowing PCs in the Management VLAN to perform SSH operations</li></ul>
<li>2 access switches</li>
<li>3 PCs</li>
<ul>
<li>2 PCs are in Management VLANs</li>
<li>1 PC is in Data VLAN</li>
</ul>
</ul>

## VLAN Structure
<ul>
<li>Management VLAN</li>
<ul>
<li>Used for SSH access to routers</li>
</ul>
<li>Data VLAN</li>
<ul>
<li>Used for regular client communication</li>
</ul>
<li>Native VLAN</li>
<ul>
<li>Temporary setup for client communication, but will be changed in future labs</li>
</ul>
</ul>

# Security Configuration

## Access Controls Lists
Access control lists are configured on the branch and remote routers to enforce the following design goals.

### Access Control List Design Goals
<ul>
<li>Allow Management PCs to:</li>
<ul>
  <li>Reach their default gateway</li>
  <li>Access routers via SSH, and not Telnet</li>
</ul>
<li>Allow Data PCs:</li>
<ul>
<li>Reach their gateway</li>
<li>Communicate with other data devices and Internet (future lab)</li>
</ul>
<li>Deny:</li>
<ul>
<li>Management VLAN to Data VLAN access</li>
<li>Data VLAN to SSH access to routers</li>
<li>All Telnet traffic</li>
</ul>
</ul>

<pre><code>Extended IP access list DATA-Router
    10 permit icmp 192.168.10.0 0.0.0.255 host 192.168.10.1
    20 deny tcp 192.168.10.0 0.0.0.255 host 192.168.10.1 eq 22
    30 permit ip 192.168.10.0 0.0.0.255 any
Extended IP access list MGMT-Router
    10 permit tcp host 172.16.10.5 host 172.16.10.1 eq 22
    20 permit tcp host 172.16.10.20 host 172.16.10.1 eq 22
    30 permit icmp host 172.16.10.5 host 172.16.10.1
    40 permit icmp host 172.16.10.20 host 172.16.10.1
Extended IP access list 101
    10 deny tcp any any eq telnet
</code></pre>

## Port Security
Port security is configured on access switch ports connected to PCs to prevent unauthorized device access. 
### Port Security Configuration Guidelines
<ul>
<li>Any unused ports must be administratively shutdown by using:
  <pre><code>conf t
  int {interface}
  shutdown</code></pre></li>
<li>Port security is enabled on access switches' ports connected to client or management PCs</li>
<li>There is no port security enabled on trunk links between the switch and router. </li>
<li>Each port should limit to one configured MAC address</li>
</ul>

## Network Configuration
## Trunk Lines
Trunk lines are used to carry Management VLAN and Data VLAN traffic between ther access switch and router. All trunk lines use 802.1q encapsulation. 
