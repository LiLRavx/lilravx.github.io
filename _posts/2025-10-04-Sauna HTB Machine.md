---
layout: post
title: Sauna Writeup
date: 2025-09-28 18:00:00 +0300
categories: [Active Directory - CRTP, DCsync] # Add your new categories here
tags: [Replication, DCsync,AS-REP Roasting, AD,WinRM,WinPEAS]
image: "photos/Sauna/bg.jpg"
---

❗Important Commands in this article ❗ 

```jsx
1. GetNPUsers.py EGOTISTICAL-BANK.LOCAL/ -usersfile users_ready -dc-ip 10.10.10.175 -no-pass
2. upload /opt/pref/winPEAS.ps1 C:\\Users\\FSmith\\Documents\\winPEAS.ps1
3. Get-ObjectAcl | Select ObjectSID,ActiveDirectoryRights | Format-List
4. secretsdump.py svc_loanmgr:Moneymakestheworldgoround!@10.10.10.175
5. [secretsdump.py](http://secretsdump.py/) svc_loanmgr:Moneymakestheworldgoround!@10.10.10.175
```

Sauna is an easy difficulty Windows machine that features Active Directory enumeration and exploitation, these two attacks we can take a benefit from them when we do a Security assessment to the companies and entities, you will find some of users have wrong permission that leads to these attacks.

with this article am going to correlate everything with realistic scenarios that i personally faced in my technical assessment tasks at my work.

after the initial enumeration of the machine , I found it is a DC and many of interesting ports that opened to the public such as port 88

for this port (88) -  this is responsible for Kerberos authentication, 

while this port is responsible for Kerberos authentication its a good point to exploit it by finding if there is a user have permission “**Do not require Kerberos pre-authentication”.**

**why we are looking for this permission ?** 

because in the first step of the authentication with Kerberos the user sends a message requests     AS-REQ asking for verifying username and password from the KDC in the domain controller in order to granting a ticket, the DC will check the user and  password and then it will send AS-RES, if the permission “**Do not require Kerberos pre-authentication”**  enabled on the user that requested the ticket, ****That means the KDC will send the AS-REP (TGT) directly to the user without verifying the password first. This AS-REP contains both the user’s NTLM hash and the TGT encrypted by the KRBTGT account.

 In this attack, we focus on extracting the NTLM hash part to crack the user’s password.

![](/photos/Sauna/second.png)

![](/photos/Sauna/third.png)

Then i need the users list for Active directory, the good point in the Active directroy that All administrator have the same concept about creating users in the active directory, the conecpt is, don’t add the full username to login the user name have to be first letter of his username and then family name, or user name and first or and second  letter of the family name.

for example

my name is yazan saleh  it can be Y.saleh or Yazan.sh

we can use tool such as username-anarchy  to create our own list, for the Sauna’s machine i got all the usernames from web page they posted the team member, so  generate a list for each name 

![](/photos/Sauna/fourth.png)

now we need to start our hypothisi for the vulnarability on kerberos port, we have our list ready, and we have ip of the machine 

we can use [NPGetusers.py](http://NPGetusers.py) from impact scripts, if you didn’t find it online you can install impacet using pip

```jsx
GetNPUsers.py EGOTISTICAL-BANK.LOCAL/ -usersfile users_ready -dc-ip 10.10.10.175 -no-pass
```

![](/photos/Sauna/fifth.png)

cracking the user by finding the module on hashcat table that start with $krb4asrep$23$ 

![](/photos/Sauna/sexith.png)

now we can connect on the machine using evil-winrm port 5985

by using evil-winrm plugins we can upload our winpeas to Check the **Local Windows Privilege Escalation** 

![](/photos/Sauna/sevnth.png)

to upload files using evil-winrm you have to head to the directory that contains the files you need to uplaod to the victem machine, then lunch your connection from their as shows in the figure below:

![](/photos/Sauna/egiht.png)

then use upload plugin to upload it their.

```jsx
upload /opt/pref/winPEAS.ps1 C:\\Users\\FSmith\\Documents\\winPEAS.ps1
```

After running the script i found the svc_loanmanager services with password that added on auto login, and this services is part of admin group.

![](/photos/Sauna/nine.png)

then i enumerated the ACL for the following users  using power view and AD module

![](/photos/Sauna/ten.png)

by command 

```jsx
Get-ObjectAcl | Select ObjectSID,ActiveDirectoryRights | Format-List
```

![](/photos/Sauna/eleven.png)

**DCsync** abuses Active Directory replication: an attacker impersonates a domain controller and requests replication from a real DC to obtain credential material (password hashes, Kerberos keys, etc.). This lets the attacker harvest account credentials without directly compromising the domain controller’s NTDS.dit file.

**How it works (short):**

![](/photos/Sauna/twleve.png)

- AD replication allows DCs to exchange directory changes.
- If an account has the appropriate replication privileges, an actor can request those changes and receive sensitive secrets (NT hashes, AES keys) for domain accounts.
- This is not possible for arbitrary domain users — specific permissions are required.

**Required permissions / privileges**

(typically granted to high-privilege groups such as Domain Admins / Enterprise Admins / Domain Controllers, or explicitly delegated):

- `DS-Replication-Get-Changes`
- `DS-Replication-Get-Changes-All`
- `DS-Replication-Get-Changes-In-Filtered-Set`

![](/photos/Sauna/thirteen.png)

and found  permission `DS-Replication-Get-Changes` on the user SVC_Loanmgt

we can lunch our attack to get a copy of the ntds.dit file

![](/photos/Sauna/forteen.png)

