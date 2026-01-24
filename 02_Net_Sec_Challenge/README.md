# Net Sec Challenge (Try Hack Me) write-up ⚔️

In my journey to become a certified junior penetration tester I came across this CTF. Its difficulty is set as medium, and the room is intended to be completed within 60 minutes, which I did not achieve 😔, but more on this later. As a result, the host IP address changes throughout this write-up. Rest assured that, for the sake of this exercise, this is the same machine.

## Introduction ℹ️

> _Use this challenge to test your mastery of the skills you have acquired in the Network Security module. All the questions in this challenge can be solved using only `nmap`, `telnet`, and `hydra`._

## Challenge Questions ❓

As stated above we can only use the specified tools to answer questions below, so let's get started.

### Q1: What is the highest port number being open less than 10,000?

I connected to TryHackMe’s network via VPN and booted Kali on a virtual machine, which caused me to miss the 60-minute requirement. But let me explain further.

I ran a common `nmap` query: `nmap -sS -sV -T4 10.48.158.36 -p-` and I allowed the scan to continue; however, after half an hour, I terminated it because it showed no progress. I tried to modify this command, to narrow down the scan but with no joy. Finally, I retried the original command using TryHackMe’s AttackBox and oh boy... Scan was finished within 15 seconds 🙄. To this day I didn't find out why `nmap` on TryHackMe's network is so slow, but will look into it.

![nmap](images/nmap.png)

**Answer:** 8080

### Q2: There is an open port outside the common 1000 ports; it is above 10,000. What is it?

If we look back on the scan result, we definitely can find one.

**Answer:** 10021

### Q3: How many TCP ports are open?

All the ports we scanned were TCP so we just need to count them up.

**Answer:** 6

### Q4: What is the flag hidden in the HTTP server header?

For this question we can utilize `nmap` scripts and get header by sending `nmap --script=http-headers 10.48.174.141 -p 80`

![Http header](images/header.png)

**Answer:** THM{web_server_25352}

### Q5: What is the flag hidden in the SSH server header?

Our answer we can see on the `nmap` scan result from **Q1**, but there are other ways. While still working with nmap, there yet another script to get it `nmap -sV --script=banner 10.48.174.141 -p 22`. This approach would also work on the HTTP server header question.

![SSH header](images/ssh_header.png)

Another way we can tackle this one, is to use a good'ol `telnet`. By simply executing `telnet 10.48.174.141 22`

![SSH header telnet](images/telnet.png)

**Answer:** THM{946219583339}

### Q6: We have an FTP server listening on a nonstandard port. What is the version of the FTP server?

FTP server was set up on the 10021 port instead of standard 21. Probably to make it more secure 😉.

Since in my initial scan I used `-sV` option, we already got version of FTP server. For less scrolling (back to the first screenshot) we execute `nmap -sV 10.48.174.141 -p 10021`

![FTP_version](images/ftp_version.png)

**Answer:** vsftpd 3.0.5

### Q7: We learned two usernames using social engineering: **eddie** and **quinn**. What is the flag hidden in one of these two account files and accessible via FTP?

First we create `users.txt` file with given usernames. For the password database I will use **rockyou.txt** because TryHackMe CTF creators often uses this database for accounts passwords.

This is the moment when `hydra` come into play. All we need to do is to hit FTP server with this `hydra -L users.txt -P rockyou.txt 10.48.158.36 ftp -s 10021` and wait.

![Hydra delivers](images/hydra.png)

Now we got both passwords and can scour the FTP server for our flag. Unfortunately logging in Kali to ftp set on port higher than 1024 ftp gets us to passive mode, so we won't get any output for commands like `dir`. I tried using `passive`, but I get _Illegal PORT command_.

So I took other approach, just downloaded whole contents of FTP server from both accounts with simple `wget -m ftp://USER:PASSWORD@10.48.174.141:10021` and there it was

![ftp flag](images/ftp_flag.png)

**Answer:** THM{321452667098}

### Q8: Browsing to http://MACHINE_IP:8080 displays a small challenge that will give you a flag once you solve it. What is the flag?

This was a trial-and-error type of challenge. A simple `nmap -sN -T4 10.48.158.36` command did the trick.

![scan challenge](images/nmap_sn.png)

![challenge result](images/challenge.png)

**The flag is:** THM{f7443f99}

---

And that concludes this challenge. I hope you enjoyed it as much as I did. See you in the next one. 👋
