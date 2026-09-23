# Build Log – Active Directory Home Lab

Step-by-step record of building a Windows Server Active Directory environment in VirtualBox and resolving common help desk tickets, including the problems I hit and how I solved them.

## Lab environment

| Component | Details |
|---|---|
| Host | Windows 11 · 8 GB RAM · 4 CPU cores · virtualization enabled |
| Hypervisor | Oracle VirtualBox 7.2.18 |
| Domain controller VM | Windows Server 2025 (or 2022) · 2 GB RAM · 2 vCPU · name: DC |
| Client VM | Windows 11 Enterprise · 4 GB RAM for install, 3 GB after · 2 vCPU · name: CLIENT1 |
| Domain | mydomain.com |
| Internal network | 172.16.0.0/24 (VirtualBox internal network) |
| DC internal IP | 172.16.0.1 (static) |
| DHCP scope | 172.16.0.100 – 172.16.0.200 |

## Progress

**Setup**
- [X] 01 – Plan the lab
- [X] 02 – Install VirtualBox and download ISOs

**Domain controller**
- [X] 03 – Create the DC virtual machine
- [X] 04 – Install Windows Server and Guest Additions
- [X] 05 – Configure network adapters, static IP, and server name
- [X] 06 – Install AD DS and promote to domain controller
- [ ] 07 – Create admin OU and domain admin account
- [ ] 08 – Install and configure RAS/NAT
- [ ] 09 – Install and configure DHCP
- [ ] 10 – Bulk-create users with PowerShell

**Client**
- [ ] 11 – Create and install the Windows 11 client
- [ ] 12 – Verify networking and join the domain
- [ ] 13 – Log in as a domain user and verify

**Help desk tickets**
- [ ] 14 – Department OUs and security groups
- [ ] 15 – Password and account lockout policy
- [ ] 16 – Ticket: "I'm locked out"
- [ ] 17 – Ticket: "I forgot my password"
- [ ] 18 – Ticket: New hire onboarding
- [ ] 19 – Ticket: Department shared drive
- [ ] 20 – Ticket: Employee offboarding
- [ ] 21 – Ticket: "My policy isn't applying"
- [ ] 22 – Delegate help desk permissions

**Portfolio**
- [ ] 23 – Final README, diagram, and résumé bullets

## Troubleshooting summary

<!-- One row per problem you hit. This table becomes your interview stories. -->

| Step | Symptom | Cause | Fix |
|---|---|---|---|
| | | | |

---

## Step 01 – Plan the lab

**Date: 9/18/2026**
**Time spent: 1 Hour**

**Goal:** Plan the lab network, confirm my computer can run two VMs, and set up documentation before building anything.

**What I did:**
1. Checked my hardware via Task Manager
2. Created network diagram to plan and decide how many VMs are needed.
3. Researched requirements for Windows 11 Enterprise VM and Windows Server 2022 VM to both be on laptop.

**Hardware check results:**
- Host OS: Windows 11
- RAM: 8 GB (plan: DC at 2 GB; client at 4 GB for install, then 3 GB)
- CPU: 4 cores, virtualization enabled
- Free disk: 112 GB

**Why it matters (my own words):**
A working Windows network needs four core services. In this lab, the DC provides all four:
- **AD DS** - For managing directory data like user accounts, computers, and setting access to network resources.
- **DNS** - Maps computer names to IP addresses, enabling name resolution for computers and users.
- **DHCP** - Automates the assignment and management of IP addresses and related network configurations for client.
- **RAS/NAT** - Routes internal traffic to the internet. NAT lets clients share the DC's internet-facing IP instead of their private 172.16.x.x addresses.

**Proof:**
- `screenshots/step01-host-specs.png`
- `diagram/network_diagram.png`

**Problems & fixes:**
8 GB of RAM was the only problem really but it will end up being more of just a slowdown. While creating VM's I will only have the one being made running and then the client drops from 4GB to 3GB RAM after its install. The DC stays at 2GB the whole time. I can't avoid running both VMs at once forever, the DC has to be on when the client joins the domain.

**What I got wrong at first → what I learned:**

1. **"Fallback IP"**
   - **I thought:** DHCP gave clients a "fallback IP" that pointed them to the DC.
   - **Actually:** There's no fallback. The DHCP scope hands every client the DC's static internal IP (172.16.0.1) as both its router (default gateway) and its DNS server.
   - **Why it matters:** If the DC's IP ever changed, every client would still point at the old address, and users would lose internet access, name lookups, and the ability to find the domain. That's why servers get static IPs.

2. **The DC getting its own address from DHCP**
   - **I thought:** The internal IP was set within DHCP
   - **Actually:** The DC is the DHCP server therefor it cannot have an IP assigned to it by DHCP.
   - **Why it matters:** Any server that other machines depend on (DHCP, DNS, domain controllers) needs a static IP, so clients can always find it at the same address.

3. **Leaving DNS off the DC's list of jobs**
   - **I thought:** DNS was a job included in AD DS.
   - **Actually:** It's installed with AD DS but is its own service/job.
   - **Why it matters:** Because DNS is a separate service, it can break even when AD DS is running fine. Without DNS, clients can't find SRV records, so users can't log in. When a domain join or login fails, I should check DNS settings before Active Directory.

**Help desk / security connection:**

**Ticket 1 – No network access**
- **Symptom:** User's `ipconfig` shows 169.254.x.x and they can't reach anything.
- **Means:** Their computer couldn't reach a DHCP server (APIPA).
- **Check:** Cable/Wi-Fi first, then whether the DHCP server is running.

**Ticket 2 – Can't join the domain**
- **Symptom:** "An Active Directory Domain Controller (AD DC) for the domain 'mydomain.com' could not be contacted."
- **Means:** The DNS could not find SRV's containing 'mydomain.com'.
- **Check:** The DNS is properly set to the DC's DNS using 'ipconfig /all', if so check DHCP status.

---

## Step 02 – VirtualBox Prep

**Date:** 9/18/2026
**Time spent:** 30 minutes

**Goal:** Install VirtualBox and download the ISOs for Windows 11 Enterprise and Windows Server 2022.

**What I did:**
1. Installed VirtualBox 7.2.18 and confirmed the version (Help → About).
2. Downloaded the Windows Server 2022 and Windows 11 Enterprise evaluation ISOs from Microsoft's Evaluation Center to `C:\ISOs`.
3. Ran `Get-FileHash` on the Windows 11 ISO and compared it to the SHA-256 hash on Microsoft's download page. Result: [MATCHED / DID NOT MATCH].

**Why it matters (my own words):**
Downloading from Microsoft instead of a third-party site ensures the ISOs haven't been modified or bundled with malware. Checking the hash confirms the file wasn't corrupted or tampered with, so later install problems won't come from a bad ISO. The Server evaluation must also activate online within 10 days, which the DC will do through its internet-facing (NAT) NIC.

**Proof:**
- `screenshots/step02-virtualbox-version.png`
- `screenshots/step02-iso-folder.png`
- `screenshots/step02-hash-check.png`

**Problems & fixes:**
None.

**Help desk / security connection:**
- Hash checking protects integrity, one of the three parts of the CIA triad (confidentiality, integrity, availability).
- VMs are used to isolate workloads, analyze malware safely, and enable rapid recovery from attacks. In this lab, snapshots let me roll back instantly when I break something.

---

## Step 03 – Create the DC virtual machine

**Date:** 9/18/2026
**Time spent:** 30 minutes

**Goal:** Create the DC VM, set its hardware, and configure its two NICs to use NAT and Internal Network respectively.

**What I did:**
1. Created a VM named `DC` using the Windows Server 2022 ISO, and checked "Skip Unattended Installation" so I could install Windows manually.
2. Configured 2048 MB of RAM, 2 CPU cores, and a 50 GB disk (not pre-allocated, so it only grows as space is used).
3. Set Shared Clipboard and Drag'n'Drop to Bidirectional.
4. Configured two network adapters: Adapter 1 on NAT and Adapter 2 on Internal Network (`intnet`).

**Why it matters (my own words):**
Two network adapters let the DC 1) connect to the internet and 2) host a private network for the client VMs. The internal network is isolated, so lab services like the DC's DHCP server can't affect my real home network.

**Proof:**
- `screenshots/step03-dc-vm-settings.png`

**Problems & fixes:**
None.

**What I got wrong at first → what I learned:**
1. **Which setting must match between the DC and the client**
   - **I thought:** The client's "router" and "DNS" fields had to match the DC.
   - **Actually:** Those are Windows settings that DHCP hands out. At the VirtualBox level, the internal network *name* (`intnet`) must match. It works like plugging both machines into the same switch.
   - **Why it matters:** If the names don't match (e.g., `intnet2`), the client lands on a separate isolated network, can't reach DHCP, and ends up with a 169.254.x.x (APIPA) address.

**Help desk / security connection:**
Real servers often have multiple network adapters to keep networks separate. For example, one faces users and another is only for administration. Separating networks like this is a basic security practice (network segmentation).

---

## Step 04 – Install Windows Server and Guest Additions

**Date:** 9/18/2026
**Time spent:** 15 minutes

**Goal:** Install Windows Server 2022 Standard Evaluation (Desktop Experience) and install Guest Additions.

**What I did:**
1. Started the DC VM and selected Windows Server 2022 Standard Evaluation (Desktop Experience).
2. Set the Administrator password to something easy to type, since I'll use it often in the lab. (In a real environment, this password should be strong.)
3. Installed Guest Additions from the CD image, then fully shut down and restarted the VM. Verified that the mouse was smooth and the resolution adjusted when resizing the window.
4. Took a VM snapshot named "Fresh Install + Guest Additions."

**Why it matters (my own words):**
Server Core is often preferred for production servers because Desktop Experience uses more RAM and disk, needs more patches, and has a larger attack surface. For this lab, I'm using Desktop Experience because it's faster to navigate and easier to screenshot for documentation. Guest Additions makes mouse movement and window resizing work properly in the VM. The snapshot gives me a clean rollback point if a later step goes wrong.

**Proof:**
- `screenshots/step04-winver.png`
- `screenshots/step04-snapshot.png`

**Problems & fixes:**
None.

**What I got wrong at first → what I learned:**
1. **Server Core's benefits**
   - **I thought:** Server Core's advantages were a smaller attack surface and better privilege separation.
   - **Actually:** The attack surface point is right. The other main benefits are fewer patches and reboots, and lower RAM and disk use.
   - **Why it matters:** Knowing the real trade-offs lets me explain why a company would choose one over the other.

**Help desk / security connection:**
Without Guest Additions, the mouse lags and the resolution is stuck. That's the same thing you'd see on a real PC missing its graphics driver. Checking drivers is a standard help desk fix for display and mouse problems.

---

## Step 05 – Configure network adapters, static IP, and server name

**Date: 9/19/2026**
**Time spent: 30 Minutes**

**Goal: Configure network adapters name, set IPv4 properties for Internal adapter, and server name**

**What I did:**
1. Went to "about" section of settings and renamed device "DC".
2. Opened network connections and identified adapter IP starting with 169.254.x.x as Internal and the one with 10.0.2.x is Internet, then renamed them "_Internal" and "Internet" respectivly.
3. Opened IPv4 properties of Internal adapter and set static IP (172.16.0.1), subnet (255.255.255.0), and DNS (127.0.0.1, loopback).
4. Restarted VM and confirmed settings applied with `hostname` and `ipconfig /all`.

**Why it matters (my own words):**
Renaming the server "DC" is important later as it makes it easier to distinguish against other machines in our lab. The same applies for renaming the adapters as being able to quickly know which adapter is which down the line will save lots of time. The Internal Network adapter's IPv4 properties were important to set as the DC can't get an address from its own DHCP server so it needs a static one. This IP is then used for clients default gateway and DNS server so it must not be changing on them. We don't configure it to have a default gateway as NIC 1 already does and NIC 2 wouldn't allow requests to reach the internet. We use the loopback address for DNS as our DC will run our DNS once AD is installed.

**Proof:**
- `screenshots/step05-ipconfig.png`

**Problems & fixes:**
None.

**What I got wrong at first → what I learned:**
1. **Why the internal adapter has no gateway**
   - **I thought:** It was blank because the DC acts as the default gateway for the clients.
   - **Actually:** The DC's own internet traffic already exits through INTERNET, which has its own gateway. A second gateway on _INTERNAL could send traffic into the isolated network, where it can't reach the internet.
   - **Why it matters:** A machine should have only one default gateway.

**Help desk / security connection:**
The `ipconfig /all` fields a help desk tech reads first when a user says "the internet is down,":
1. Media State: This will say "Media disconnected" if the cable is unplugged or WiFi is off.
2. IPv4 Address: A 169.254.x.x means the device cannot reach the DHCP server to get a valid lease.
3. Default Gateway: If this is empty the computer has no path to send traffic outside the local private network.

---

## Step 06 – Install AD DS and promote to domain controller

**Date: 9/22/2026**
**Time spent: 30 Minutes**

**Goal:** Install the Active Directory Domain Services role and promote the server to a domain controller for a new forest, mydomain.com.

**What I did:**
1. Took a snapshot named "Before AD DS."
2. Opened Server Manager and select "Add roles and features" option.
3. Added Active Directory Domain Services role and install to DC.
4. Promoted DC to domain controller and added new forest "mydomain.com" and DRSM password, NetBIOS name autofills to "MYDOMAIN".
5. Verified with `whoami` and `nslookup -type=SRV _ldap._tcp.dc._msdcs.mydomain.com`.

**Why it matters (my own words):**
<!-- What does promoting a server to a DC actually give me? Why did DNS install automatically alongside AD DS? What is the DSRM password for? -->
Promoting to a DC allows me to run Active Directory and its services, like authenticating users, locating computers by name, applying group policies, discovering local services, and storing certain config data. DNS is automatically installed alongside it and it what allows us to resolve computer names to addresses. We also set a DSRM (Directory Services Restore Mode) password since now our user account lives on the AD not the server so if it ever goes offline we still need a way to get in and attempt to restore it.

**Proof:**
- `screenshots/step06-whoami-srv.png`
- `screenshots/step06-server-manager.png`
- `screenshots/step06-multihomed-fix.png`

**Problems & fixes:**
**Problem 1: the DNS delegation warning**
- **Symptom:** During the promotion to domain controller, a warning says a delegation for this DNS can't be created.
- **Cause:** A delegation is a pointer from a parent DNS zone to yours. So the parent of mydomain.com is the real .com zone which I don't control.
- **Fix:** None needed for the lab, but a real company avoids it by using a domain they own or an internal-only name.

**Problem 2: the multi-homed DC**
- **Symptom:** nslookup shows dc.mydomain.com resolving to both 172.16.0.1 and 10.0.2.15.
- **Cause:** By default, Windows registers every adapter's IP in DNS. Since my DC has two adapters, both got registered, and DNS hands them out in rotation.
- **Fix:** 1. Uncheck "Register this connection's addresses in DNS" for the unwanted adapter.
2. Limit the DNS server to the internal IP in the Server Manager by going to DNS properties for DC.
3. Delete record of host with wrong IP to clear stale records.
  

**What I got wrong at first → what I learned:**
1. **What changed about the Administrator account after promotion**
   - **I thought:** I stated it was because the AD DS role/features were added but that's the cause not effect.
   - **Actually:** Before it was stored in the server but was added to the domain, now it lives in domain
   - **Why it matters:** Now that its stored in the domain, if the domain were to become unreachable for whatever reason then we now need the DSRM password to get in since we can't access the admin account.

**Help desk / security connection:**
<!-- The nslookup SRV command is the exact check for the "AD DC could not be contacted" error. What does it prove if it returns the DC? -->
`nslookup -type=SRV _ldap._tcp.dc._msdcs.[YourDomain].[Extension]` 

`nslookup -type=SRV`: Instructs the system to query DNS specifically for Service records.
`_ldap`: The name of the service you are looking for (Lightweight Directory Access Protocol).
`_tcp`: The transport protocol used by the service.
`dc`: dc is a fixed label meaning "domain controllers".
`_msdcs`: A Microsoft-specific DNS zone used by Active Directory for internal locator services (Microsoft Domain Controller Service).
`[YourDomain].[Extension]`: Placeholder for your organization's internal Active Directory domain name (in my case `mydomain.com`).

If DNS passed the test, you now know DNS is not the reason the DC cannot be contacted. The "AD DC could not be contacted" error is now almost certainly being caused by a network, protocol, or authentication barrier between the client and that specific DC host. If it fails, then you know DNS name resolution for Active Directory isn't working properly.

---

## Blank entry

```
## Step NN – Title

**Date:**
**Time spent:**

**Goal:**

**What I did:**
1.
2.
3.

**Why it matters (my own words):**

**Proof:**
- `screenshots/stepNN-description.png`

**Problems & fixes:**
- Symptom:
- What I checked:
- Cause:
- Fix:

**What I got wrong at first → what I learned:**

**Help desk / security connection:**

---
```
