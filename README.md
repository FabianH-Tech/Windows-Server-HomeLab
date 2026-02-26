# **Windows Server HomeLab**

Documentation and configuration files for my Windows Server home lab running on my Thinkpad T14 Gen 3 on VirtualBox.

**The Goal**

    Building a complete "Branch Office in a Box" Windows Server infrastructure to gain hands-on experience with enterprise networking and systems administration. The goal was to simulate a real company network with a Domain Controller (DC826) and a File Server (FS1030) where clients receive automatic IPs, users access centralized storage, and all servers can be managed remotely.

**The Tools**

    Virtualization: 

    - Oracle VirtualBox

    Operating Systems:

    - Windows Server 2025 (Desktop Experience) - Evaluation copy

    - Windows 11 Pro

**Server Roles & Features**

    - Active Directory Domain Services (AD DS)

    - DNS Server

    - DHCP Server

    - File Server

    - Group Policy Management

**Networking**
| Component | Details |
|-----------|---------|
| **Virtual Network** | NAT Network `ad_lab` (10.0.1.0/24) |
| **DNS Server** | DC826 @ 10.0.1.10 (Static) |
| **File Server** | FS1030 @ 10.0.1.11 (Static) |
| **DHCP Scope** | 10.0.1.100 - 10.0.1.200 |
| **DHCP Lease** | 8 days |
| **Gateway** | 10.0.1.1 (VirtualBox NAT) |

**Management Tools:**

    - Server Manager

    - Active Directory Users and Computers

    - DNS Manager

    - DHCP Console

    - PowerShell

    - Command Prompt

**The Process**

**Phase 1: Network Setup**

    - Created a NAT Network in VirtualBox named ad_lab with CIDR 10.0.1.0/24

    - Disabled VirtualBox's built-in DHCP to prepare for Windows Server DHCP

**Phase 2: Domain Controller (DC826) Setup**

    - Created VM with 4GB RAM, 50GB Virtual Hard Disk

    - Installed Windows Server 2025 (Desktop Experience)

    - Configured static IP: 10.0.1.10/24 with gateway 10.0.1.1 and DNS 10.0.1.10

    - Installed features: AD DS, DNS, and DHCP roles via Server Manager

    - Promoted to domain controller with a new forest named homelab.local

    - Authorized DHCP server in AD

**Phase 3: DHCP Configuration (DC826 Local Server)**

    - Created DHCP scope "Office LAN" with range 10.0.1.100 - 10.0.1.200

    - Set scope options:

        Scope excludes 10.0.1.1 - 10.0.1.99 for future server implementations

        Router:(Defaul Gateway) 10.0.1.1

        DNS Servers: 10.0.1.10

    - Activated and authorized the scope

**Phase 4: File Server (FS1030) Setup**

    - Created second VM with 4GB RAM, 50GB Virtual Hard Disk

    - Installed Windows Server 2025 (Desktop Experience)

    - Configured static IP: 10.0.1.11/24 with gateway 10.0.1.1 and DNS 10.0.1.10

    - Attempted to join domain homelab.local—encountered DNS resolution issues

    - Completed troubleshooting the DNS resolution issue which ended up being a simple NAT Network not connected on Virtualbox to the File Server, preventing access to the DNS server. 

**Phase 5: Major Troubleshooting - DNS Connectivity**

The Situation: After configuring DNS on DC826 and setting FS1030's DNS to 10.0.1.10, I couldn't resolve domain names. Test-NetConnection showed port 53 was unreachable, and netstat revealed DNS wasn't bound to any IP address. I spent significant time troubleshooting:

    - DNS service status (Utilized command: sc query dnscache)

    - Windows Firewall rules for port 53 and enable inbound ICMPv4-IN rule

    - Verified DNS connection (Utilized command: Test-NetConnection 10.0.1.10 -Port 53) test failed

    - DNS interface bindings in DNS Manager (Configured listening on the DNS server 10.0.1.10)

    - DNS server registration with ipconfig /registerdns

**The Actual Problem**

    - After an hour of deep-dive troubleshooting, I discovered the root cause was embarrassingly simple—FS1030 wasn't connected to the correct NAT network in VirtualBox. The VM was completely isolated from DC826.

**The Fix:**

    - Shut down FS1030

    - VirtualBox → FS1030 Settings → Network

    - Changed "Attached to" from incorrect setting to NAT Network named "ad_lab"

    - Started FS1030 - all connectivity worked immediately

    - Verified through these two commands: ping 10.0.1.10 & nslookup homelab.local 10.0.1.10
