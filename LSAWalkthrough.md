<h1>LazySysAdmin 1.0 Walkthrough</h1>


Start with Nmap scan
`nmap -Pn -sT -sV -O 192.168.17.139 -p1-65000`

![Image of Nmap](https://blu0.github.io/LSAWalkthrough/LSAnmap.png)

Let’s take a look at the website

![Image of Site](https://blu0.github.io/LSAWalkthrough/LSAsite.png)

Nothing to really see off –hand, time for some more enumeration

Kicking things off with Nikto
`nikto -h 192.168.17.139`
![Image of Nikto](https://blu0.github.io/LSAWalkthrough/LSAnikto.png)

Robots.txt shows the same directories
`/old/` and `/test/` show nothing
Poked at `/myphpadmin/` but it requires a password. A few guesses proved useless
There is a Wordpress site, but nothing seems to load. Who cares? I’m going to add `/wp-admin/` anyway and see if there is a login

Voila!
![Image of Nikto](https://blu0.github.io/LSAWalkthrough/LSAwp-admin.png)

Now it's time for some WPScan to see what usernames are there
![Image of Nikto](https://blu0.github.io/LSAWalkthrough/LSAwpscane.png)
![Image of Nikto](https://blu0.github.io/LSAWalkthrough/LSAwpscanadmin.png)

