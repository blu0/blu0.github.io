
<h1>DerpNStink 1 Walkthrough</h1>

Author Description: "Mr. Derp and Uncle Stinky are two system administrators who are starting their own company, DerpNStink. Instead of hiring qualified professionals to build up their IT landscape, they decided to hack together their own system which is almost ready to go live..."

Difficulty: Beginner

`root@kali:~# nmap -Pn -v -sS -sV -p- 192.168.92.135`

![Image of Nmap](https://blu0.github.io/DNSWalkthrough/nmap.png)

Checked FTP (port 21) but no anonymous login is allowed, moved on to port 80.

The hell...?

![Image of Site](https://blu0.github.io/DNSWalkthrough/port80.png)

Robots.txt shows a couple of directories to investigate (`/php/` and `/temporary/`) but I need to enumerate more.

Viewed the source code of the main page and noticed this:

![Image of info](https://blu0.github.io/DNSWalkthrough/dns.png)

Edited my hosts file:

![Image of hosts](https://blu0.github.io/DNSWalkthrough/hosts.png)

Ran some Nikto and Dirbuster scans until I got some useful results:

![Image of dirbuster](https://blu0.github.io/DNSWalkthrough/dirbuster.png)

`nikto -h http://192.168.92.135/weblog/`

![Image of Nikto](https://blu0.github.io/DNSWalkthrough/nikto.png)

So there is a Wordpress site... ezpz.. enumerate users and plugins.

`wpscan --url http://192.168.92.135/weblog -e vp -e u`

![Image of wpscan](https://blu0.github.io/DNSWalkthrough/wpscan.png)

Trying admin:admin seems to login fine, but it doesn't appear to be an actual admin account.

![Image of admin](https://blu0.github.io/DNSWalkthrough/adminadmin.png)

The plugin found seems to have a [public exploit available](https://www.exploit-db.com/exploits/34681/)

Ran it with the [Pentest Monkey PHP Reverse Shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell)

`python 34681.py -t http://derpnstink.local/weblog/ -u admin -p admin -f php-reverse-shell.php`

![Image of exploit](https://blu0.github.io/DNSWalkthrough/exploitplugin.png)

Head to the location of the upload:

![Image of fire](https://blu0.github.io/DNSWalkthrough/fire.png)

Catch the reverse shell:

![Image of revshell](https://blu0.github.io/DNSWalkthrough/revshell.png)

At this point (or maybe sooner) I deviated from what I think the path was supposed to be and pwned the system using the 32-bit shellcode of [the Dirty Cow expoit](https://www.exploit-db.com/exploits/40616/).

![Image of cowroot](https://blu0.github.io/DNSWalkthrough/cowroot.png)

Found 2 flags:

![Image of flags](https://blu0.github.io/DNSWalkthrough/flags.png)

Poking around it looks like MySQL was flag2...? I kind of stopped really bothering at this point tho. (I already won, leave me alone!)

All in all, this was a fun machine.
