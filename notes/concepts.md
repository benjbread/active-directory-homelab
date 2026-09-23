# Concept Notes – Active Directory Home Lab

Plain-English notes on every concept used in this lab, written in my own words.
Each card connects the concept to what a user would experience and how I would troubleshoot it.

## Concept index

## Concept index

- [ ] NAT (Network Address Translation) — Steps 1, 8 *(draft, fixes pending)*
- [ ] DHCP (Dynamic Host Configuration Protocol) — Steps 1, 9 *(draft, fixes pending; also covers APIPA and static vs. dynamic addressing)*
- [ ] DNS (Domain Name System) — Steps 1, 6, 12 *(draft, fixes pending; also covers SRV records)*
- [x] File hash (SHA-256) — Step 2
- [x] VirtualBox network modes (NAT vs. Internal Network) — Step 3
- [ ] Server Core vs. Desktop Experience — Step 4
- [X] Default gateway — Steps 5, 9
- [ ] Subnet mask (/24) — Step 9
- [X] Active Directory Domain Services (AD DS) — Step 6
- [X] Domain, forest, and domain controller — Step 6
- [ ] Organizational Unit (OU) — Steps 7, 14
- [ ] Security groups — Steps 7, 14
- [ ] Domain Admins and least privilege — Steps 7, 22
- [ ] PowerShell execution policy — Step 10
- [ ] Domain join — Step 12
- [ ] Group Policy (GPO) — Steps 15, 19, 21
- [ ] Account lockout vs. disabled account — Steps 15, 16, 20
- [ ] Share vs. NTFS permissions — Step 19
- [ ] Event Viewer and Security logs (event ID 4740) — Step 16
- [ ] Delegation of control — Step 22

---

## NAT (Network Address Translation)

**In one sentence (my words):**
Allows multiple devices in the same private network to access the internet using one shared public IP.

**Everyday analogy:**
At the bar with your friends you all get on one tab instead of paying seperatly.

**Where it lives in my lab:**
This lives in the DC VM, part of the "Remote Access" role. NIC 1 faces the internet and NIC 2 is internal facing.

**What a user would notice if it broke:**
Users would lose the ability to access the internet but still have access to anything on the private network (logging in, reaching the DC, and shared drives still work)

**Command to check it:**
<!-- Leave blank for now. You'll fill this in during Step 12. -->

**Interview question it answers:** 
How do computers with private IP addresses reach the internet?

**My answer:**
It gets translated by NAT into the shared public IP used by the router (in this lab the DC is the router), and then the IP that made the request is stored so it can be properly returned.

**Source(s):**
https://www.geeksforgeeks.org/computer-networks/advantages-and-disadvantages-of-nat/
https://www.reddit.com/r/techsupport/comments/yln3j6/question_about_network_address_translation_nat/

---

## DHCP (Dynamic Host Configuration Protocol)

**In one sentence (my words):** 
This is a network management protocol that will automatically assign IPs (within set range) to clients and apply relevant network configuration settings.

**Everyday analogy:** 
This is like when you start looking for a match on Fortnite and it automatically identifies what location you're in (for server), your rank, etc. and places you in the proper matchmaking queue. Without it, its more like Minecraft where you manually enter the servers IP and Port before joining.

**Where it lives in my lab:**
This lives on the DC, and for my lab the range will be 172.16.0.100-200. The two options that points clients to the DC are the 'router' field and 'DNS' field.

**What a user would notice if it broke:**
If DHCP broke, the client would find themselves with an 169.254 address, meaning they couldn't get an IP from the DHCP server.

**Command to check it:**
<!-- Leave blank for now. You'll fill this in during Step 12. -->

**Interview question it answers:** 
A user's IP address starts with 169.254. What does that tell you, and what do you check first?

**My answer:**
This tells me the user's device isn't able to get an IP assigned by the DHCP server, usually mean DHCP is broken and isn't handing out IPs.

**Related: APIPA (169.254.x.x)**
- **What it is:** Allows a device to automatically assign itself an IP address.
- **When a machine assigns it to itself:** When the device can't reach a DHCP server.
- **What it tells a help desk tech:** 1. Either the DHCP server is unreachable or misconfigured. 2. Device is on the wrong network 3. DHCP scope ran out of addresses.
- **Where I saw it in this lab:** Before I set the static IP for NIC 2 on the DC.

**Related: Static vs. dynamic addressing**
- **Static:** A non changing IP, used on the DC's NIC 2 so that clients always reach the right thing.
- **Dynamic:** A changing IP from the DHCP server, used on the clients.
- **Why the DC must be static:** The DC is the DHCP server so it can't get an address assigned from itself. Also, clients will be using it for routing and DNS so they must always be able to reach it at the same place.
- **What breaks if a server's static IP changes:** Its connection to client devices.

**Source(s):**
https://www.whatismyip.com/169-254-ip-address/
https://www.geeksforgeeks.org/computer-networks/dynamic-host-configuration-protocol-dhcp/

---

## DNS (Domain Name System)

**In one sentence (my words):**
DNS translates human readable names to the numerical IP addresses (or vice versa).

**Everyday analogy:** 
Like when you find a phone number from a name (via phone book or google)

**Where it lives in my lab:**
This lives on the DC and the DC uses its loopback address as its own DNS server while clients use the DC's Internal facing IP.

**What a user would notice if it broke:**
"An Active Directory Domain Controller (AD DC) for the domain 'mydomain.com' could not be contacted.", meaning even if can connect to internet they cannot retrieve SRV records to confirm. Also, websites wouldn't load by name.

**Command to check it:**
<!-- Leave blank for now. You'll fill this in during Step 12. -->

**Interview question it answers:** 
A computer can browse the internet but can't join the domain. What's the most likely cause?

**My answer:**
The DNS is set to one that doesn't contain the correct SRV records to contact the domain.

**Related: SRV records**
- **What they are:** A record in the DNS.
- **What they tell a client:** Tells clients where to find specific services without relying on default ports.
- **Where they live in this lab:** They live on the DC as it acts as our networks DNS.
- **Why 8.8.8.8 can't provide them:** While other DNS IPs might allow access to the internet they will lack the SRVs to get access to local services.
- **What a user sees if the client can't find them:** They will experience connection timeouts, "server not found: errors, or set up failures all because they cannot automatically discover the specific hostname and port to run local services.

**Source(s):**
https://www.cloudflare.com/learning/dns/what-is-dns/

---

## File Hash (SHA 256)

**In one sentence (my words):** 
Creates a 256 bit hash value for an input of any size using a mathematical algorithm.

**Everyday analogy:** 
This is like a human fingerprint

**Where it lives in my lab:** 
This is used for hash checking. In our lab I checked the hash of my Windows 11 Evaluation edition and checked it against the one provoided.

**What a user would notice if it broke:** 
If the hash's didn't match this could mean a few things: 1. file download corrupted, 2. file tampered with, 3. file isn't complete 

**Command to check it:** 
Get-FileHash .\filename.ext (uses SHA 256 by default)

**Interview question it answers:** 
What is the difference between SHA-256 and Encryption?

**My answer:** 
With encryption you can still recover the data, while with a file hash you can only verify integrity.

**Source(s):**
https://www.quora.com/What-is-a-hash-mismatch

---

## VirtualBox network modes (NAT vs. Internal Network)

**In one sentence (my words):**
NAT connects a single VM to the internet hidden behind the host's network address, while an Internal Network lets multiple VMs reach each other while staying isolated from the host and the internet.

**Everyday analogy:**
NAT is like using the company mailroom to send and receive your mail. The Internal Network is like the company intercom: it works inside the building but not outside.

**Where it lives in my lab:**
The DC's Adapter 1 uses VirtualBox NAT, which gives the DC internet access through my host. The DC's Adapter 2 and the client's adapter both use the Internal Network named `intnet`, so they share an isolated virtual network. The client never uses VirtualBox NAT directly. It reaches the internet through the DC's own Windows NAT (Remote Access role), which forwards its traffic out Adapter 1.

**What a user would notice if it broke:**
- **Adapter 1 set wrong:** The DC and all clients lose internet, but domain logins and DHCP still work because they stay on the internal network.
- **Internal network name mismatch** (e.g., `intnet` vs. `intnet2`): The client gets a 169.254.x.x (APIPA) address and can't reach the DC, the domain, or the internet.

**Command to check it:**
- In VirtualBox: Settings → Network, and confirm each adapter's mode and network name.
- Inside the VM: `ipconfig`. The VirtualBox NAT adapter shows a 10.0.2.x address (VirtualBox's default NAT range). The internal adapter shows 172.16.0.x, or 169.254.x.x if it can't reach DHCP.

**Interview question it answers:** Why would you put lab or test VMs on an isolated internal network instead of connecting them directly to your real network?

**My answer:**
Isolation keeps lab services from affecting the real network. For example, my DC runs a DHCP server. On my home network, it could hand out wrong addresses to real devices and break their connections. Isolation also contains risky activity, like malware testing, so it can't spread. The most important takeaway is that its like a separate network for testing that wont affect our real network.

**What I got wrong at first:**
I thought the client used VirtualBox NAT to reach the internet. It actually uses the DC's Windows NAT, which then sends traffic out the DC's VirtualBox NAT adapter. There are two separate NAT layers.

**Source(s):**
VirtualBox User Manual – Virtual Networking chapter (virtualbox.org/manual)

---

## Server Core vs. Desktop Experience

**In one sentence (my words):** 
Server core uses only CLI so its better for performance and security. Desktop Experience is easier to navigate for beginners since it uses a GUI which can be useful for documentation screenshots.

**Everyday analogy:**
Server Core is like a high-performance sports car stripped of features to maximize speed and efficiency, while Desktop Experience is the fully loaded luxury SUV equipped with all the comfort features and a heavy touchscreen dashboard.

**Where it lives in my lab:**
In my lab I used the Desktop Experience as documentation will be easier for screenshots and it will be faster for me as its more familiar.

**Trade-offs:**
The Desktop Experience is going need more RAM and disk use because of its larger codebase. This larger codebase also introduces the need for more patches meaning more reboots. The last main downside is more code = more to attack, basically the larger codebase creates more opportunity's for attackers to find a weakness. Server Core does have its own issues, mainly being its harder to learn and some software requires a desktop to operate.

**How Server Core is managed without a desktop:**
<!-- PowerShell remoting and Windows Admin Center. -->
The Windows Admin Center gateway translates actions from its web-based GUI into PowerShell commands and WMI queries and executes them remotely via the Windows Remote Management service (WinRM).

**Interview question it answers:** Why might a company run its servers without a graphical desktop?

**My answer:**
A server running without a GUI has less hardware requirements, better overall security, faster runtime, and since it has less code it needs fewer patches meaning fewer reboots.

**What I got wrong at first:**
None.

**Source(s):**
https://learn.microsoft.com/en-us/windows-server/manage/windows-admin-center/configure/use-powershell

---

## Default gateway

**In one sentence (my words):** The device that client devices will use to send data to external networks or the internet (anything outside the private network basically).

**Everyday analogy:**
Imagine trying to leave a building with many doors (some leading to the entrance, some to the back, some to the garage, etc.), the default gateway would be the one that lets you access things outside that building (a.k.a the entrance/exit door).

**Where it lives in my lab:**
In the DC the INTERNET adapter's gateway is assigned from the VirtualBox's built-in NAT DHCP. We leave the _INTERNAL adapter blank since the DC already has the default gateway and adding one leading into the internal network would mean no access to the internet. Clients will use the DC's DHCP service to get their gateway assigned, which will route them through the DC to the internet.

**What a user would notice if it broke:**
A user would still be able to access local services (things on the local private network) but nothing outside of it (can't access the internet).

**Command to check it:**
`ipconfig /all`

**Interview question it answers:** A user can reach devices on the local network but not the internet. What's one likely cause?

**My answer:**
One cause could be that the default gateway is misconfigured. You can check this by running `ipconfig /all` and checking the default gateways value. If its empty then the DHCP server isn't properly assigning the routing field. If its there but wrong then the DHCP server is misconfigured and giving out the wrong gateway. The last option would be its correct but the gateway device itself is down or misrouting. You can ping the gateway IP, and if it replies but the internet doesn't then you know the client config is correct and the issue is upstream.

**What I got wrong at first:**
None.

**Source(s):**
https://www.geeksforgeeks.org/computer-networks/default-gateway-in-networking/

---

## Active Directory Domain Services (AD DS)

**In one sentence (my words):**
Active Directory Domain Services is used for things like managing a network's authentication, locating computers and services, and applying GPOs.

**Everyday analogy:**
This would be like entering a company where they check your work ID at the entrance and allow you access to the part of the building you're supposed to work in.

**Where it lives in my lab:**
In my lab the DC runs AD DS. Using this I created the root forest mydomain.com, and the NetBIOS name was set to MYDOMAIN by default from the forest.

**What it gives me that standalone computers don't:**
1. Authenticating users and computers
2. Locating computers by name
3. Applying Group Policy Objects (GPOs)
4. Discovering and locating local services
5. Storing certain config data

**What a user would notice if it broke:**
If AD DS stopped working a user would no longer be able to authenticate (so they might claim their password isn't working, can't log in, etc.), they wouldn't be able to discover or locate local services, and GPOs wouldn't be in effect so they wouldn't have their typical permissions. Users may still get into their machines on cached credentials from a previous sign-in, so the first complaint is often "I can't reach the shared drive" rather than "I can't log in." In my lab the same machine also runs DNS and DHCP, so if the DC is down, all three go down together.

**Command to check it:**
- `whoami` – displays the currently logged in user
- `nslookup -type=SRV _ldap._tcp.dc._msdcs.mydomain.com` – returns the SRV record for the domain
- `dcdiag` – runs a health check on a domain controller

**Interview question it answers:** What does Active Directory actually do for an organization?

**My answer:**
Active Directory is the central directory of an org's users, computers, and groups. Instead of each device keeping its own separate accounts, they are all stored in AD. This gives the company the power to enforce Group Policy Objects (GPOs), locate devices by human-readable names, and have single sign-on where one account works on any domain machine. It gives admins the ability to do things like disable an account everywhere from one server, instead of separately on every machine that user appears on.

**What I got wrong at first:**
None.

**Source(s):**
https://learn.microsoft.com/en-us/training/paths/active-directory-domain-services/

---

## Domain, forest, and domain controller

**In one sentence each (my words):**
- **Domain:** An area of a network organized by a single authoritative database.
- **Domain controller:** The DC is the authority over AD objects, authentication, and changes. A Domain can have multiple DCs each with a copy of the AD and syncing changes to each other.
- **Forest:** The top of the hierarchy

**How they relate:**
Forests hold 'trees', and trees are just collections of one or more domains. A forest can have one tree and still be a forest. A Domain can have multiple domain controllers but a domain controller can't belong to multiple domains. An organization might have multiple DCs for a domain so that if one goes down they have redundancy and services will remain running.

**Everyday analogy:**
If a domain is like an organizations building where workers have ID to get into their respective offices. Then a Forest would be like if that org had multiple buildings that all operate under the same company policies. The buildings can't belong to multiple organizations but the organization can have multiple buildings.

**Where it lives in my lab:**
In my lab the forest is named `mydomain.com` since that's the name of the root domain, the domain is named `mydomain.com`, the NetBIOS name is `MYDOMAIN`, and the hostname is `DC`.

**Why the forest matters for security:**
By default a forest administrator won't have permissions in other forests. This is because the forest acts as the 'security boundary'. For sharing permissions between forests you must establish a 'forest trust'. They allow for the establishment of a trust relationship between two forests, enabling users in one forest to access resources in another, provided the appropriate permissions are assigned.

**Interview question it answers:** What's the difference between a domain and a forest?

**My answer:**
A domain is an area of a network while a forest is one or more trees of domains. A forest acts as the top in the hierarchy, meaning it's the 'security boundary'. For forest-level admins this means everything under the forest is in their scope of control. Most small companies run one domain per forest since the forest acts as the boundary to help isolate environments. You'd add more domains when an area of the network is needed that will operate under a different set of rules, for example, when a company expands internationally and different laws apply to how work needs to be conducted.

**What I got wrong at first:**
None.

**Source(s):**
https://learn.microsoft.com/en-us/training/paths/active-directory-domain-services/

---

## Blank card (copy this for each new concept)

```
## CONCEPT NAME (Full Name)

**In one sentence (my words):**

**Everyday analogy:**

**Where it lives in my lab:**

**What a user would notice if it broke:**

**Command to check it:**

**Interview question it answers:**

**My answer:**

**What I got wrong at first:**

**Source(s):**

---
```
