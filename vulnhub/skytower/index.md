**skytower is boot2root machine from vulnhub and considered an easy machine.**

**This is my methodolgy** 

**1- Enumeration**

**2- Exploitation**

**3- Post Exploitation**

**Enumeration**

We begin with nmap scan to see open ports and services running.![login](https://user-images.githubusercontent.com/113348039/192632548-a278517d-0597-4537-947d-773a932f7f91.png)


`nmap -sC -sV -p- -T5 10.0.2.16
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-27 09:01 EDT
Nmap scan report for 10.0.2.16
Host is up (0.00086s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE    SERVICE    VERSION
22/tcp   filtered ssh
80/tcp   open     http       Apache httpd 2.2.22 ((Debian))
3128/tcp open     http-proxy Squid http proxy 3.1.20`

![nmapScan](https://user-images.githubusercontent.com/113348039/192632294-b57cf905-5618-4951-8eb6-0ba6c5f30976.png)


-sC: for default scripts

-sV: for version 

-p-: to scan all 65535 ports

* Enumerating HTTP 

it looks like it is login page 

![login](https://user-images.githubusercontent.com/113348039/192632658-46490dff-b6e8-4adb-ad4e-8493aabc1cc4.png)

I like to view page source because if there is something hidden press ctrl+u 
But nothing interesting.

![sourceCode](https://user-images.githubusercontent.com/113348039/192632788-d8cfe648-2419-4961-8892-3dd9b88476d7.png)

Let's find subdirectories, I use gobuster dir mode to find subdirectories.
No Subdirectories interesting

![gobuster](https://user-images.githubusercontent.com/113348039/192632995-f4c93e3b-83a2-41b9-bd0e-6840c7951ba9.png)

I tried to test if there was broken authentication
I typed anything to see whether the error message is verbose
But it wasn't verbose , I tried some credentials but all failed.
![authentication1](https://user-images.githubusercontent.com/113348039/192633177-d13e4d0a-4721-4261-875e-9312f2a52a8e.png)
![authentication2](https://user-images.githubusercontent.com/113348039/192633201-8c352223-ff99-4def-9221-5049abc4968d.png)



Let's try basic sql injection payload sql error occured which means there is sql injection vulnerability
![sqlinjection](https://user-images.githubusercontent.com/113348039/192633290-4261b337-80f0-4666-b209-a1a9db0bac97.png)
![SqlError](https://user-images.githubusercontent.com/113348039/192633384-6518d90d-014b-43b4-8333-3f3b41ce5e3b.png)

**Exploiting Sql Injection**

I tried some payloads til i write the right payload that exploit the vulnerability

   ' || 1=1#
![sqlPayload](https://user-images.githubusercontent.com/113348039/192633504-94c61129-bb41-4665-9d59-2ce84e45e926.png)
![PayloadResult](https://user-images.githubusercontent.com/113348039/192633571-3b84886d-174d-4af3-96e5-e8a5d151e1c0.png)


And Here is the credentials for user called john, looks like he is an actual user on the box.

This page said that you must login via ssh to access the account details and since ssh port(22) is filtered we can't directly access the account via ssh. 

So we will use portforwarding technique to connect to port 22 indirectly.

We will use squid proxy to create a tunnel on the target machine that forward the data received from port 1122 on the local machine to port 22 on the target machine.

username:john ,password:hereisjohn
![sshError](https://user-images.githubusercontent.com/113348039/192633844-1884277a-f280-40e5-96ec-a001a46a6452.png)

A strange string appear which is "Funds have been withdrawn" and the connection closed 

After a lot of time thinking and searching about the error I remembered that there is a hidden file in every user directory called .bashrc

> The .bashrc file is a script file that’s executed when a user logs in. The file itself contains a series of configurations for the terminal session. This includes setting up or enabling: coloring, completion, shell history, command aliases, and more.

So I decided to delete this file from the target machine, Fortunately ssh has the ability to execute commands on the target without login shell.

execute the command to delete .bashrc file then connect with ssh 

AND Finally we got a shell
![SHELL](https://user-images.githubusercontent.com/113348039/192634080-6d856eae-64b4-4119-bb3d-873f28bb07ba.png)



**Post Exploitation**
**Privilege Escalation**
* Method 1

Users on host: john , sara , william 
![Users](https://user-images.githubusercontent.com/113348039/192634528-073b0438-927c-4a8d-8e70-b068d42608be.png)


Go back to login page which is vulnerable to sql injection and try to get the password for these accounts
![SaraCreds](https://user-images.githubusercontent.com/113348039/192634637-161e0129-b3c9-4fee-b952-eed45e4f7210.png)
![SaraCreds2](https://user-images.githubusercontent.com/113348039/192634656-faca388e-4c4b-4a15-8ea1-d0d7f2e405ca.png)
![williamCreds](https://user-images.githubusercontent.com/113348039/192634673-72e84de7-73e4-4626-94c3-7e3d1f901117.png)

User:sara , password:ihatethisjob 

user:william , password:senseable 

* Method 2

If you type netstat -ano to find the connections on the machine 

![netstat](https://user-images.githubusercontent.com/113348039/192634747-bb5ae53a-04f5-45ab-84a8-94eb03aedd78.png)


the result indicate that port 3306 is open which is mysql, let's try to inspect login.php to find the credentials

![RootCreds](https://user-images.githubusercontent.com/113348039/192635518-7ebe47f8-5ca5-4d9b-bb1b-f1f0ca9ec081.png)

Let's connect to root with mysql 

![mysql](https://user-images.githubusercontent.com/113348039/192635899-43460e06-b28e-4513-ba84-6c4917d039b5.png)

try to show databases and access the columns of login table and see the users 

![mysql2](https://user-images.githubusercontent.com/113348039/192635956-77de720f-9a77-4f80-8fe8-26d460fe05a5.png)
![mysql3](https://user-images.githubusercontent.com/113348039/192635977-48549d0d-de18-45bb-9f8e-7f8d4058d7a1.png)

Let's do like we did with john (delete .bashrc) then connect to sara

![SaraShell](https://user-images.githubusercontent.com/113348039/192636078-06ad2e5f-7af7-4503-b0c8-2890204e8940.png)

See what sara can execute with sudo command  

> sudo command gives you safe, elevated privileges to run important commands

![SaraSudo](https://user-images.githubusercontent.com/113348039/192636126-f54e383b-9474-45e3-bb47-5fdad9d74400.png)

From the result sara has access to any thing in accounts directory you can abuse that by using dot dot slash attack which is known as directory traversal to see root's flag.

![Root'sPassword](https://user-images.githubusercontent.com/113348039/192636545-2f64c9cc-ae1b-4ba5-a24e-7ce113159d32.png)

user:root , password:theskytower

Type `su root` and write the password and got root shell.

![RootShell](https://user-images.githubusercontent.com/113348039/192636849-f62a4e97-c7ee-4f8f-9270-d8a34c5d0ca8.png)


