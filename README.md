**Platform:** HackTheBox Academy

**Difficulty:** Medium

**OS Used:** Linux

**Target IP:** 10.129.202.41

**Data:** 2026-05-08


**GOAL:**

&nbsp;&nbsp;&nbsp;&nbsp;The goal of this lab was to footprint a Windows machine and find the password of a user named HTB stored inside a database.

**Port Scanning:**

&nbsp;&nbsp;&nbsp;&nbsp;Running one scan that does everything at once takes a long time. A smarter appoach is to split it into two scans, first scan sweeps all 65535 ports as fast as possible.

   ```
   nmap -p- --min-rate 5000 -T4 10.129.202.41
   ```

&nbsp;&nbsp;&nbsp;&nbsp;Now that we know exactly which ports are open, we run a deeper scan only on thoes ports

   ```
   nmap -sV -sC -p 111,135,139,445,2049,3389,5985,47001 10.129.202.41
   ```

   <img width="789" height="637" alt="m-1" src="https://github.com/user-attachments/assets/a41bca63-9733-41af-9fea-548e91c386b5" />

&nbsp;&nbsp;&nbsp;&nbsp;The most unusual one was NFS(Network File System). NFS is file sharing protocal that is normally used on Linux systems, so seeing it open on windows machine immediately caught my attention.

**Enumeration on NFS**

    ```
    showmount -e 10.129.202.41

    sudo mkdir target-nfs
    ```

    <img width="200" height="90" alt="m-2" src="https://github.com/user-attachments/assets/7b703c07-a34d-4dcb-89eb-0c7b694ba6c3" />


    

   

   
   
