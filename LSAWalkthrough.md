<h1>LazySysAdmin 1.0 Walkthrough</h1>

Start things off with an Nmap scan (of course).

`nmap -Pn -sT -sV -O 192.168.17.139 -p1-65000`

![Image of Nmap](https://blu0.github.io/LSAWalkthrough/LSAnmap.png)

A few different places to start, but I'll take a look at the website first.

![Image of Site](https://blu0.github.io/LSAWalkthrough/LSAsite.png)

Nothing really jumps out after looking around for a bit, so time for some more enumeration.

I'll go with trusty old Nikto here.

`nikto -h 192.168.17.139`
![Image of Nikto](https://blu0.github.io/LSAWalkthrough/LSAnikto.png)

So looking through the results, there are quite a few things to check here.
Robots.txt shows the same directories as the Nikto scan.
The `/old/` and `/test/` directories show nothing.
Poked at `/myphpadmin/` but it requires a password. A few guesses proved useless... meh, moving on for now.
There is a WordPress site, but nothing seems to load. Who cares? I’m going to add `/wp-admin/` anyway and see if there is a login

Voila!

![Image of Wp-admin](https://blu0.github.io/LSAWalkthrough/LSAwp.png)

WordPress looks the most viable way in and would be a quick reverse shell if I can work my way in.
It's time for some WPScan to see what usernames are there.
![Image of WPscan](https://blu0.github.io/LSAWalkthrough/LSAwpscane.png)
![Image of WPscan2](https://blu0.github.io/LSAWalkthrough/LSAwpscanadmin.png)

So we have Admin and nothing else. Ok then.
Trying some WPScan Brute Force, I scraped the site with Cewl
`cewl -m 5 http://192.168.17.139/wordpress -w lazycewl.txt`
Then ran it through some mutations with John The Ripper
`john --wordlist=/root/lazycewl --rules --stdout > lazymutate.txt`
And even tried rockyou for a few minutes before cancelling it
wpscan –u http://192.168.17.139/wordpress –wordlist lazymutate.txt –username admin
Maybe the password is somewhere else...

Time to check the SMB port and see if there is any relevant data to be found.
`enum4linux -a 192.168.17.139`

