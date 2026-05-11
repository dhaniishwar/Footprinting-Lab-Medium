**Platform:** HackTheBox Academy

**Difficulty:** Medium

**OS Used:** Linux

**Target IP:** 10.129.202.41

**Data:** 2026-05-08

---
**GOAL:**

&nbsp;&nbsp;&nbsp;&nbsp;The goal of this lab was to footprint a Windows machine and find the password of a user named HTB stored inside a database.

---
**Tools Used:**

 &nbsp;&nbsp;&nbsp;&nbsp; - Nmap 
 &nbsp;&nbsp;&nbsp;&nbsp; - Showmount 
 &nbsp;&nbsp;&nbsp;&nbsp; - Smbclient 
 &nbsp;&nbsp;&nbsp;&nbsp; - Xfreerdp3 
 &nbsp;&nbsp;&nbsp;&nbsp; - SSMS (SQL Server Management Studio) 

---
**Port Scanning:**

&nbsp;&nbsp;&nbsp;&nbsp;Running one scan that does everything at once takes a long time. A smarter appoach is to split it into two scans, first scan sweeps all 65535 ports as fast as possible.

>I will not recommend you to use this comment in real life scenario because it will make lot of noise in the network.
&nbsp;
   ```
   nmap -p- --min-rate 5000 -T4 10.129.202.41
   ```

&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Now that we know exactly which ports are open, we run a deeper scan only on thoes ports

&nbsp;
   ```
   nmap -sV -sC -p 111,135,139,445,2049,3389,5985,47001 10.129.202.41
   ```


   <img width="789" height="637" alt="m-1" src="https://github.com/user-attachments/assets/a41bca63-9733-41af-9fea-548e91c386b5" />
   
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp; The most unusual one was NFS(Network File System). NFS is file sharing protocal that is normally used on Linux systems, so seeing it open on windows machine immediately caught my attention.

---
**Enumeration on NFS :**

 ```bash
 showmount -e 10.129.202.41
  ```

 ```bash
 sudo mkdir target-nfs
 ```

    
   <img width="200" height="90" alt="m-2" src="https://github.com/user-attachments/assets/ae2a7c85-5219-465b-85df-3f59b83165e7" />
   
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;We used 'showmount' to ckeck what NFS share were available on the target and found a share called '/TechSupport' that was accessible by everyone - no authentication required.We can get the share folder on our local mechine by mounting it and send to the folder name 'target-nfs'.


```
sudo mount -t nsf 10.129.202.41:/TechSupport ./target-nfs -o nolock
```
```
cd target-nfs
ls -al
```


<img width="441" height="42" alt="m-3" src="https://github.com/user-attachments/assets/9a232a05-dcae-45e5-b947-c970452f435f" />

&nbsp;
<img width="475" height="484" alt="m-4" src="https://github.com/user-attachments/assets/c983fd3b-22e4-424d-8af9-3ede4e8e938f" />

&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;We found 70 of ticket files inside. Most of the files were empty, but
&nbsp;

<img width="462" height="564" alt="m-5" src="https://github.com/user-attachments/assets/e2abe98c-389b-4a7f-a622-3f4e778020da" />

&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Only one file,ticket4238791283782.txt, had a size of 1305 bytes, which told us immediately that it was the only file worth reading.
&nbsp;

```
cat ticket4238791283782.txt
```

<img width="671" height="630" alt="m-6" src="https://github.com/user-attachments/assets/6164b84f-bd5d-4ab5-90d4-52da547cf037" />

&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Inside the file, there was a chat conversation between a support operator and a user named alex, where alex had pasted his SMTP configuration file directly into the chat. That config file contained his username and plaintext password.
&nbsp;

 &nbsp;&nbsp;&nbsp;&nbsp;**Credential :** alex:lol123!mD

---
**Enumeration on SMB :**

&nbsp;&nbsp;&nbsp;&nbsp; SMB(Server Message Block) is the windows file sharing protocol. We can list all available shares by 'SMBclient'.
&nbsp;

```
smbclient -L \\\\10.129.202.41 -U alex
```

<img width="629" height="180" alt="m-x" src="https://github.com/user-attachments/assets/3aed0c54-89a8-4be5-9a9d-4c5a7c677e8c" />

&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Usually, the standard Windows share names end with '$'. We found two non-default shares Users and devshare.Let's connect to devshare by username alex
&nbsp;

```
smbclient \\\\10.129.202.41\\devshare -U alex
```

 <img width="629" height="163" alt="m-7" src="https://github.com/user-attachments/assets/7b165303-234e-498f-a655-27548548ecef" />

 &nbsp;
 &nbsp;&nbsp;&nbsp;&nbsp; We found a single file called important.txt. Inside it,
 &nbsp;
 

&nbsp;&nbsp;&nbsp;&nbsp; **Credential :** sa:87N1ns@slls83
&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;sa is the default built-in superuser account in Microsoft SQL Server. sa(System Administrator) account, the most powerful account in a Microsoft SQL Server database.

---
**Access RDP :**

&nbsp;&nbsp;&nbsp;&nbsp;RDP (Remote Desktop Protocol) is a Microsoft protocol that allows you to connect to a Windows computer over a network and see its full graphical desktop on your screen, just like you are physically sitting in front of that machine. Connect to RDP using alex credential.
>The ! character breaks bash commands. So use single quotes around passwords in the terminal.

```
xfreerdp3 /v:10.129.202.41 /u:alex /p:'lol123!mD'
```

<img width="402" height="274" alt="m-9" src="https://github.com/user-attachments/assets/baaec0d6-4d85-4059-be12-efb325b4bd12" />

&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Now we could see the Windows desktop of the machine with SQL Server Management Studio (SSMS) sitting right there on the desktop.
&nbsp;

<img width="519" height="399" alt="m-10" src="https://github.com/user-attachments/assets/edfcaf58-4e31-4085-b3c2-d9c2ca131029" />

---
**Getting Inside The Database :**

&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;We opened SSMS by right-clicking and running it as Administrator. A UAC (User Account Control) prompt appeared asking for the Administrator password.
&nbsp;

<img width="517" height="404" alt="m-11" src="https://github.com/user-attachments/assets/5dfa2ed8-f7b2-45b6-9c02-197339ccd443" />

&nbsp;
>We can use the password of sa (87N1ns@slls83).

<img width="512" height="401" alt="m-12" src="https://github.com/user-attachments/assets/028ce51c-385d-4175-a366-99cf1fae70f6" />

&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Make the sittings in SQL Server as shown in the above image to connected to the local SQL Server.
&nbsp;

 <img width="511" height="401" alt="m-13" src="https://github.com/user-attachments/assets/aafa465b-c8df-49b5-b8b8-9085d3f775b7" />

 &nbsp;
&nbsp;&nbsp;&nbsp;&nbsp; We found a database called accounts containing a table called dbo.devsacc with lot of usernames and passwords (Including HTB password).
&nbsp;&nbsp;&nbsp;&nbsp;

<h3 align="center"> Goal Achieved </h3>

---
**Summary:**

&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp; Enumerated an NFS share which contained a support ticket with SMTP credentials for user alex. Used those credentials to access an SMB share containing a file with  SA credentials. RDP'd into the machine using alex's credentials, then opened SQL Server Management Studio as Administrator using the SA password to access the accounts database and retrieve the HTB user credentials.
