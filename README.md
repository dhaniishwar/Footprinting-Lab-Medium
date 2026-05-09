**Platform:** HackTheBox Academy

**Difficulty:** Medium

**OS Used:** Linux

**Target IP:** 10.129.202.41

**Data:** 2026-05-08

---
**GOAL:**

>The goal of this lab was to footprint a Windows machine and find the password of a user named HTB stored inside a database.

---
**Port Scanning:**

>Running one scan that does everything at once takes a long time. A smarter appoach is to split it into two scans, first scan sweeps all 65535 ports as fast as possible.


   ```
   nmap -p- --min-rate 5000 -T4 10.129.202.41
   ```


>Now that we know exactly which ports are open, we run a deeper scan only on thoes ports


   ```
   nmap -sV -sC -p 111,135,139,445,2049,3389,5985,47001 10.129.202.41
   ```


   <img width="789" height="637" alt="m-1" src="https://github.com/user-attachments/assets/a41bca63-9733-41af-9fea-548e91c386b5" />
   

>The most unusual one was NFS(Network File System). NFS is file sharing protocal that is normally used on Linux systems, so seeing it open on windows machine immediately caught my attention.

---
**Enumeration on NFS :**

 ```bash
 showmount -e 10.129.202.41
  ```

 ```bash
 sudo mkdir target-nfs
 ```

    
   <img width="200" height="90" alt="m-2" src="https://github.com/user-attachments/assets/ae2a7c85-5219-465b-85df-3f59b83165e7" />
   

>We used 'showmount' to ckeck what NFS share were available on the target and found a share called '/TechSupport' that was accessible by everyone - no authentication required.

>We can get the share folder on owr local mechine by mounting it and send to the folder name 'target-nfs'.


```
sudo mount -t nsf 10.129.202.41:/TechSupport ./target-nfs -o nolock
```


<img width="441" height="42" alt="m-3" src="https://github.com/user-attachments/assets/9a232a05-dcae-45e5-b947-c970452f435f" />

<img width="475" height="484" alt="m-4" src="https://github.com/user-attachments/assets/c983fd3b-22e4-424d-8af9-3ede4e8e938f" />









    

   

   
   
