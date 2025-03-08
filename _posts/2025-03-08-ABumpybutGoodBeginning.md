---
title: A Bumpy but Good Beginning
date: 2025-03-08 -0800 # Yes, write out the date, it doesnt automatically pick it up. -800 might be whats messing up the date and making it a day earlier so consider removing.
categories: [LBFH]
tags: [linux basics for hackers, lbfh, projects, linux, objectives, hacking, dns, dhcp, file manipulation]     # TAG names should always be lowercase
author: <author_id>
description: An unexpected start and LBFH chapters one through four.
toc: false
---


<h3>A (Long) Preface</h3>

  To start, I'll be honest, it's been too long of a gap in between posts, but I also need to appreciate leaving myself the flexibility in the future to have longer gaps in between. Part of it is that I need to break these posts down into smaller chunks, and part of is that I ran into some issues already. These adjustments and formatting will come to me as I continue to do this, all in due time.

  Very shortly after my last post, one of the nodes on my Proxmox cluster was corrupted. It landed in grub rescue mode, and while I briefly messed around with recovery, I actually didn't care all that much to recover it because I had recently decided to redo my Proxmox cluster anyway. The previous cluster formation was my first Proxmox installation, and I wanted to go back and change some things. First, I wanted to avoid creating VMs on the primary node. Second, I wanted to avoid using shared storage. Third, I wanted to have a minimum of one extra node, and fourth, I wanted upgrade the CPUs to 4590Ts for extra cores and performance.

  I purchased 3 4590T CPUs and replaced the previous 2 core CPUs, I believe all three tinys had 4570Ts. Even if I didn't replace them, I don't think the CPUs had ever had the thermal paste refreshed anyway, which they were probably in desperate need of. I have a Meanwell LRS-350-24 power supply that I would like to use to power three of the M73 tinys with future plans of duplicating that for another three M73 tinys in the cluster. Overall I have 7 or 8 Lenovo tinys lying around to use for this. Eventually I'd like to upgrade the RAM on each machine from 8GB to 16GB as well, but that will be a process over time.

  All of that said, I'm getting too far off topic for this post because this isn't about my plans for my homelab and cluster, it's about progressing my technical knowledge with the previously laid out goals and sources. What the above boils down to is that with the corruption of my second node, I no longer had the VM for the first book I'm tackling, Linux Basics for Hackers (LBFH), a Kali environment for me to follow along with.

  In between troubleshooting and upgrading my Proxmox cluster, I've been digging into LBFH. As previously mentioned, I have a decent amount of Linux experience in a few different distributions. I'm writing this on a Fedora distribution, but I've also used Ubuntu and other Debian distros, as well as Arch. I am in no ways an expert, but deepening my knowledge in Linux is a core goal of mine. I mention my experience to provide context for my next point, which is that the initial chapters, one through four, went quite quickly and covered a lot of what I know. Some of it provided a bit more context for things I'm already familiar with, and some of it I didn't know.

  In my current studies, I feel particularly sensitive to rehashing things I already know. Over time while doing my own studies in tech, I've found that I often rehash things I have already learned, whether it stems from a distrust in my foundational knowledge or just thinking that I'm not ready for advancing my studies, I'm not entirely sure. I've found that because of this I sometimes find myself in Tutorial Hell as well. I watch video after video, read blog post after blog post, often feeling like I need just *a little bit more* reading before I should dive in hands on. I've noticed this pattern more and more in the past year or two, and am firmly set to balance it out by trusting myself to move forward (much) more quickly. In reality, if I'm swinging the pendulum back the other way to move forward more quickly and get more experience hands on, this is an inopportune time for my Proxmox node to fail and to lose the VM I planned on using for this book. I'll get it back up and running shortly enough, I'm making sure of that and holding myself accountable. In the meantime, for these initial chapters, for the few commands here and there that I haven't used much or at all, I just practiced it directly on my Fedora machine, adapting the commands where needed for the Fedora environment.

  Because of my goal to move more quickly and knowing much of what was covered, the following is a very brief breakdown of the chapters I covered, with little in the way of additional writing or studying. I want to note that this post is *not* how I foresee future posts and my technical posts looking, but a way for me to swing the pendulum back while dealing with a (temporary) lack of a Kali VM. I have skipped over some commands and details in the chapters where there was no value to me rehashing them and repeating my past mistakes. The following is my coverage of the chapters with a list at the end of all commands covered in my writing. It's quick and dirty, but that's a muscle I need to flex more anyway. I need to trust myself more, and I need to be okay breaking some things in test environments in the process.


<h3>Linux Basics for Hackers</h3>
**CHAPTER 1: GETTING STARTED WITH THE BASICS**
Commands:
- `locate` will go through entire filesystem and locate every instance of that word
	- Uses a database that is typically only updated once a day, so will not find things from minutes or hours ago
- `whereis` searches for binary files and will return the location of the binary, will also return man page related to the binary
- `which` only returns the location of the binaries in the PATH variable in Linux
	- **PATH variable** will be covered more in chapter seven, but it holds the directories in which the OS looks for the commands you execute in the command line.
	- `which` takes the parameter given and the OS looks to the PATH variable to see in which directories it should look look for the parameter
	- Directories at a minimum include /usr/bin but also include /usr/sbin/
	- I still have more questions about what this means and how it functions
- `find` is the most powerful and flexible of the search utilities. You can begin your search in any designated directory and can look for filename, date of creation or modification, the owner, group, permissions, and the size.
	- syntax: find *directory* *options* *expression*
	- `find / -type f -name apache2`
		- This searches for apache2 in the root directory, -f looks for ordinary files
	- A search going through every directory is slow, but it can be sped up by searching only the directories you think the file may be in
		- If we were looking for a configuration file, it would make sense to start our look in /etc
		- `find /etc -type f -name apache2`
	- find only brings back **exact** name matches, trying this in my own system you have to include the file extension too or it doesnt match
- **wildcards**:
	- `?` replaces a single character - ?at would find cat, hat, rat, etc
	- `[ ]` place multiple characters in brackets and it will find matches for any of the characters. `[c,b]at` would find bat and cat
	- `*` fills in for an unlimited amount of characters. `*at` would find chat, cat, bat, what, etc
- `ps` displays information about processes running on the machine
	- `ps aux` displays *all* processes running in a system
- `cat` obviously displays the contents of a file, but adding **>** creates a file with the filename you follow it with
	- `cat > hackingskills` would create a file called hacking skills. Exit the created file with **ctrl+d**
	- `cat >> hackingskills` will append more content to the file
	- using `cat > hackingskills` again will overwrite the contents of the file
- `touch` also does file creation if the filename doesnt already exist. If the file already exists, it was created to just *touch* a file to change the date it was created or modified, etc.
- `mkdir`
- `cp file /diffdirectory/filecopyhere`
- `mv` this can be used to move a file or rename a file
	- Linux doesnt have a command intended specifically for renaming a file, mv can be used for this purpose
- `rm`
- `rmdir`


**CHAPTER 2: TEXT MANIPULATION**
Nearly everything is a file
**NIDS**: Network intrusion detection system (like Snort)
- I've used Snort previously in a Black Hills Information Security SOC skills class and TryHackMe rooms. I'll be excited to go back to it and use it more.
- `head` Displays the beginning of a file, by default displays the first 10 lines, but you can change it with the dash switch (-)
	- `head -20 /etc/snort/snort.conf`
- `tail` same thing but displays end of file
- `nl` nl for number lines, displays the file with each line numbered
	- `nl /etc/snort/snort.conf`
- `grep`
**Challenge**: use nl, grep, tail, and head, display the 5 lines before block of text that we are looking for
- I used /etc/systemd/logind.conf as a random example file, trying to find 'OnlyUsers' text
- `nl /etc/systemd/logind.conf | grep OnlyUsers`
	- OnlyUsers appears on line 20
- `head -n+21 /etc/systemd/logind.conf`
	- This displays the first 21 lines of the file
	- adding on `| tail -n 6` gives us the line we want and the 5 lines before it
- `sed` search for occurrences of a word or text pattern and then perform an action on it
	- `sed` is stream editor, works like Find and Replace does in Windows
	- Lets use the above file as another example, and we want to replace OnlyUsers with onlyusers
	- `sed s/OnlyUsers/onlyusers/g /etc/systemd/logind.conf > /etc/systemd/logind2.conf`
		- `s` tells sed to substitute
		- `g` tells sed to do it globally, meaning it will change all instances of OnlyUsers
		- the first is what we want to replace, then a slash followed by what we want to replace it with
		- the last part references the file and tells it to save the changes in a new file we've labled as logind2.conf
		- If we wanted to replace a specific instance, if we wanted to replace the 2nd instance of something, we would replace the `g` with a `2`
- `more` cat is limited, its only useful for smaller files since it spits out the entire file contents
	- `more` will display the first page of a file and let you scroll through one page at a time using enter
- `less` this is the same as `more`, but with more functionality. Scroll through one page at a time but you can also filter for terms
	- A cheeky Linux saying is "Less is more"
	- When you are looking through a file with less, you can hit `/` followed by the term to begin a search
	- It will highlight all occurrences of your search term, then you can press `n` to cycle to the next instance
	- `less /etc/systemd/logind.conf`
		- `/Hand`
		- `n` to scroll through each instance
**Chapter 2 exercises**:
- Some exercises to go through Kali Linux password files, find certain line numbers, use more, less, grep, etc


**CHAPTER 3: ANALYZING AND MANAGING NETWORKS**
- `ifconfig` Can change IP address using ifconfig
	- `ifconfig eth0 192.168.110.10 netmask 255.255.255.0 broadcast 192.168.1.255` (fill in whatever numbers here)
	- Can also use `ifconfig` to change your MAC. First bring the ethernet interface down, change the address, then bring it back up
		- `ifconfig eth0 down`
		- `ifconfig eth0 hw ether 00:11:22:33:44:55`
		- `ifconfig eth0 up`
- **Assigning a new IP Address from the DHCP server**
	- the Linux DHCP server runs a daemon (a process that runs in the background) called **dhcpd** or the dhcp daemon.
		- This keeps logs of which IP addresses it allocated to which machine. **This makes it a great resource for forensic analysts to trace hackers** with after an attack.
		- Usually to connect to the internet from a LAN, you must have a DHCP-assigned IP
			- This means that after setting a static IP address, you must return and get a new DHCP-assigned IP address. Lets look at how to do this without having to shut your system down and restart it.
			- To request an IP address from DHCP, call the DHCP server with the command `dhclient` followed by the interface
				- Kali is built on debian so it uses `dhclient`, others distros use different commands for this
			- `dhclient eth0` this starts the DORA process with DHCP
- **Manipulating DNS**
	- Examining DNS with `dig`
	- dig helps with key reconnaissance on targets. It could include the IP address of the target's nameserver (the server that translates the target's name to an IP address), target's email server, and potentially any subdomains and IP addresses.
	- `dig hackers-arise.com ns` ns here stands for nameserver, we find the NS in the ANSWER SECTION that this command retrieves for us
		- Also note in the response shown on page 34 that the ADDITIONAL SECTION that this dig query reveals the IP address of the DNS server for hackers-arise.com
			- You can use dig to get email server info using `mx` option (short for mail exchange server)
			- `dig hackers-arise.com mx` in the AUTHORITY SECTION is the email server for hackers-arise.com
	- **The most common Linux DNS server is the Berkeley Internet Name Domain (BIND)**
		- In some cases, Linux users refer to DNS as BIND, but dont get confused as both DNS and BIND map domain names to IP addreses
- **Editing DNS**
	- Edit the /etc/resolv.conf
	- For example: `nameserver 8.8.8.8`
	- You can also do this entirely from the command line without opening the file by doing something like
		- `echo "nameserver 8.8.8.8"> /etc/resolv.conf`
		- The `>` character redirects it to the specified file and replaces the content currently in the resolv.conf file
		- You can place multiple in there and it will use the DNS sources in the order provided in the file
		- **Note: if you are using DHCP and the DHCP server provides DNS, it will replace the contents of this file when it renews the DHCP address
- **Mapping your own IP address**:
	- Local file /etc/hosts acts as a local DNS rather than letting the DNS server decide
		- You can thus map sites like microsoft.com to whatever IP you want
		- This can be useful as a hacker to hijack a TCP connection on your local network and redirect traffic to a different web server with a tool such as **dnsspoof**
	- The author uses Leafpad as a text editor
	- `leafpad /etc/hosts`
		- For example you could map bankofamerica.com to your own local IP address of 192.168.181.131
			- **Make sure you use tab between the name and IP address, *not* a space.**
**Chapter 3 summary exercises**
- Change your IP
- Change MAC address
- Check wireless interfaces
- Reset your IP to a DHCP address
- Find the nameserver and email server of a website
- Add Google's DNS 


**CHAPTER 4: ADDING AND REMOVING SOFTWARE**
`apt-cache search keyword`
- search for apps already installed
`apt-get install packagename`
`apt-get remove packagename`
- Does not remove configuration files
`apt-get purge packagename`
- Removes all configuration files along with the program
`apt autoremove packagename`
- Remove the other libraries installed along with the package by using autoremove. Packages are often broken down into smaller parts and libraries commonly used by other programs that are smaller chunks installed along with the overall package
`apt-get update` - update simply updates the list of packages available for download from the repository
`apt-get upgrade` - upgrading will upgrade the package to the latest version in the repository
Repositories - the servers that hold the software for particular distributions of Linux are known as repositories
	Your repository list lives in **/etc/apt/sources.list**
	If you are trying to install a package in Kali, for example, that apt-get doesnt find, you could always add other repository sources to sources.list to have apt-get check. The sources.list will go in order.
	Many Linux distributions divide repositories into separate categories. For instance, Debian breaks its repository down into the following:
		**Main**: Contains supported open source software
		**Universe**: Contains community-maintained open source software
		**Multiverse**: Contains software restricted by copyright or other legal issues
		**Restricted**: Contains proprietary device drivers
		**Backports: Contains packages from later releases
		**The author does not recommend using testing, experimental, or unstable repositories in your sources.list**
GUI-based installer
git installer: if the software you want isnt available in repositories, especially if its brand new, but may be available on github, you can install it with git clone.
  `git clone https://github.com/theaddress/here`
	You should be able to check that it was downloaded by using `ls -l`



**Commands Chapter 1 - Chapter 4:**
`locate
whereis
which
find
ps
cat
touch
mkdir
cp
mv
rm
rmdir
head
tail
nl
grep
sed
more
less
ifconfig
dhclient
dig ns
dig mx
apt-cache
apt-get install
apt-get remove
apt-get purge
apt autoremove
apt-get update
apt-get upgrade
git clone`

**Commands to learn more about and/or practice with more:**
`which
cat >
cat >>
find
nl
less
sed
dig
apt autoremove`
