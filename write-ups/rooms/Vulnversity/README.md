06/16/2025

Vulnversity
Note: Box seemed very unstable and constantly dropping packets. Ping reponse is less than 70% even after waiting 10 mins. Grrr

''TASK 1

Fire up the box and wait a bit for pings to return
Create working folder and nmap folder

Setup IP variable for simplicity
Export IP=<ip address>
10.10.101.248

''TASK 2

Q. Scan the box; 
#Nmap scan
#nmap -sC -sV -oN nmap/initial $IP

Results


Q. How many ports are open?
A> 4

Q. What version of the squid proxy is running on the machine?
A> 

Q. How many ports will Nmap scan if the flag -p-400 was used?

Q. What is the most likely operating system this machine is running?


Q. What port is the web server running on?

Q. What is the flag for enabling verbose mode using Nmap?

''TASK 3

Q. What is the directory that has an upload form page?
A. 

''TASK 4

Q. What common file type you'd want to upload to exploit the server is blocked? Try a couple to find out.
A. 

Q. What extension is allowed after running the above exercise?
A. 

Q. What is the name of the user who manages the webserver?
A. 

Q. What is the user flag?
A. 

''TASK 5

Q. On the system, search for all SUID files. Which file stands out?
A. 

Q. What is the root flag value?
A. 
