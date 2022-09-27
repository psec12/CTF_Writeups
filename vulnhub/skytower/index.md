** skytower is boot2root machine from vulnhub and considered an easy machine.

** This is my methodolgy 

* ** 1- Enumeration **
* ** 2- Exploitation **
* ** 3- Post Exploitation **

** Enumeration
We begin with nmap scan to see open ports and services running.
`nmap -sC -sV -p- -T5 10.0.2.16
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-27 09:01 EDT
Nmap scan report for 10.0.2.16
Host is up (0.00086s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE    SERVICE    VERSION
22/tcp   filtered ssh
80/tcp   open     http       Apache httpd 2.2.22 ((Debian))
3128/tcp open     http-proxy Squid http proxy 3.1.20`

-sC: for default scripts
-sV: for version 
-p-: to scan all 65535 ports

* Enumerating HTTP 

it looks like it is login page 

I like to view page source because if there is something hidden press ctrl+u 
But nothing interesting.

Let's find subdirectories, I use gobuster dir mode to find subdirectories.
No Subdirectories interesting

I tried to test if there was broken authentication
I typed anything to see whether the error message is verbose
But it wasn't verbose , I tried some credentials but all failed.

Let's try basic sql injection payload sql error occured which means there is sql injection vulnerability

** Exploiting Sql Injection
I tried some payloads til i write the right payload that exploit the vulnerability

   ' || 1=1#
   
And Here is the credentials for user called john, looks like he is an actual user on the box.

This page said that you must login via ssh to access the account details and since ssh port(22) is filtered we can't directly access the account via ssh. 

So we will use portforwarding technique to connect to port 22 indirectly.
We will use squid proxy to create a tunnel on the target machine that forward the data received from port 1122 on the local machine to port 22 on the target machine.

username=john
password=hereisjohn

A strange string appear which is "Funds have been withdrawn" and the connection closed 
After a lot of time thinking and searching about the error I remembered that there is a hidden file in every user directory called .bashrc

* The .bashrc file is a script file that’s executed when a user logs in. The file itself contains a series of configurations for the terminal session. This includes setting up or enabling: coloring, completion, shell history, command aliases, and more.

So I decided to delete this file from the target machine, Fortunately ssh has the ability to execute commands on the target without login shell.

execute the command to delete .bashrc file then connect with ssh 

AND Finally we got a shell

** Post Exploitation
** Privilege Escalation
* Method 1
Users on host: john , sara , william

Go back to login page which is vulnerable to sql injection and try to get the password 

User=sara
password=ihatethisjob 

user=william
password=senseable 

* Method 2
If you type netstat -ano to find the connections on the machine 

the result indicate that port 3306 is open which is mysql, let's try to inspect login.php to find the credentials

Let's connect to root with mysql 
try to show databases and access the columns of login table and see the users 






Let's do like we did with john (delete .bashrc) then connect to sara

Let's see what sara can execute with sudo command  
*sudo command gives you safe, elevated privileges to run important commands*

From the result sara has access to any thing in accounts directory you can abuse that by using dot dot slash attack which is known as directory traversal to see root's flag.

 

user=root
password=theskytower



