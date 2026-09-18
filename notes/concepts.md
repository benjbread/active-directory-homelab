# Concept Notes – Active Directory Home Lab

Plain-English notes on every concept used in this lab, written in my own words.
Each card connects the concept to what a user would experience and how I would troubleshoot it.

## Concept index

- [-] NAT (Network Address Translation) — Steps 1, 8
- [-] DHCP (Dynamic Host Configuration Protocol) — Steps 1, 9
- [-] DNS (Domain Name System) — Steps 1, 6, 12
- [-] APIPA (169.254.x.x addresses) — Steps 1, 5
- [-] Static vs. dynamic IP addressing — Steps 1, 5
- [x] File Hash / SHA 256 - Step 2
- [ ] Subnet mask (/24) — Step 5
- [ ] Default gateway — Steps 5, 9
- [-] SRV records — Steps 1, 12
- [ ] Active Directory Domain Services (AD DS) — Step 6
- [ ] Domain, forest, and domain controller — Step 6
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

**In one sentence (my words):** Allows multiple devices in the same private network to access the internet using one shared public IP.

**Everyday analogy:** At the bar with your friends you all get on one tab instead of paying seperatly.

**Where it lives in my lab:**
This lives in the DC VM, part of the "Remote Access" role. NIC 1 faces the internet and NIC 2 is internal facing.

**What a user would notice if it broke:**
Users would lose the ability to access the internet but still have access to anything on the private network (logging in, reaching the DC, and shared drives still work)

**Command to check it:**
<!-- Leave blank for now. You'll fill this in during Step 12. -->

**Interview question it answers:** How do computers with private IP addresses reach the internet?

**My answer:**
It gets translated by NAT into the shared public IP used by the router (in this lab the DC is the router), and then the IP that made the request is stored so it can be properly returned.

**Source(s):**
https://www.geeksforgeeks.org/computer-networks/advantages-and-disadvantages-of-nat/
https://www.reddit.com/r/techsupport/comments/yln3j6/question_about_network_address_translation_nat/

---

## DHCP (Dynamic Host Configuration Protocol)

**In one sentence (my words):** This is a network management protocol that will automatically assign IPs (within set range) to clients and apply relevant network configuration settings.

**Everyday analogy:** This is like when you start looking for a match on Fortnite and it automatically identifies what location you're in (for server), your rank, etc. and places you in the proper matchmaking queue. Without it, its more like Minecraft where you manually enter the servers IP and Port before joining.

**Where it lives in my lab:**
This lives on the DC, and for my lab the range will be 172.16.0.100-200. The two options that points clients to the DC are the 'router' field and 'DNS' field.

**What a user would notice if it broke:**
If DHCP broke, the client would find themselves with an 169.254 address, meaning they couldn't get an IP from the DHCP server.

**Command to check it:**
<!-- Leave blank for now. You'll fill this in during Step 12. -->

**Interview question it answers:** A user's IP address starts with 169.254. What does that tell you, and what do you check first?

**My answer:**
This tells me the user's device isn't able to get an IP assigned by the DHCP server, usually mean DHCP is broken and isn't handing out IPs.

**Source(s):**
https://www.whatismyip.com/169-254-ip-address/
https://www.geeksforgeeks.org/computer-networks/dynamic-host-configuration-protocol-dhcp/

---

## DNS (Domain Name System)

**In one sentence (my words):** DNS translates human readable names to the numerical IP addresses (or vice versa).

**Everyday analogy:** Like when you find a phone number from a name (via phone book or google)

**Where it lives in my lab:**
This lives on the DC and the DC uses its loopback address as its own DNS server while clients use the DC's Internal facing IP.

**What a user would notice if it broke:**
"An Active Directory Domain Controller (AD DC) for the domain 'mydomain.com' could not be contacted.", meaning even if can connect to internet they cannot retrieve SRV records to confirm. Also, websites wouldn't load by name.

**Command to check it:**
<!-- Leave blank for now. You'll fill this in during Step 12. -->

**Interview question it answers:** A computer can browse the internet but can't join the domain. What's the most likely cause?

**My answer:**
The DNS is set to one that doesn't contain the correct SRV records to contact the domain.

**Source(s):**
https://www.cloudflare.com/learning/dns/what-is-dns/

---

## File Hash (SHA 256)

**In one sentence (my words):** Creates a 256 bit hash value for an input of any size using a mathematical algorithm.

**Everyday analogy:** This is like a human fingerprint

**Where it lives in my lab:** This is used for hash checking. In our lab I checked the hash of my Windows 11 Evaluation edition and checked it against the one provoided.

**What a user would notice if it broke:** If the hash's didn't match this could mean a few things: 1. file download corrupted, 2. file tampered with, 3. file isn't complete 

**Command to check it:** Get-FileHash .\filename.ext (uses SHA 256 by default)

**Interview question it answers:** What is the difference between SHA-256 and Encryption?

**My answer:** With encryption you can still recover the data, while with a file hash you can only verify integrity.

**Source(s):**
https://www.quora.com/What-is-a-hash-mismatch

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
