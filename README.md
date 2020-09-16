# Boot2Root

> Résumé: Ce projet est une introduction à la pénetration d’un système.

> Après tout vos efforts vous allez enfin pouvoir vous amuser !
> Ce projet est donc une base pour vous faire comprendre comment vous devez procéder
> pour pénétrer un systéme sur lequel vous avez les droits légalement parlant.
>Je vous invite fortement à utiliser toutes les méthodes disponibles pour casser cet iso
>vraiment. La correction sera limité mais votre capacité à pouvoir exploiter votre iso sera
>grandement récompensée pour vous surtout au delà de votre note.

We have to find 2 exploit methods to validate the mandatory part. 1 bonus per other valid methods

![screen_page](img/screen_full_page.png)

## Mandatory part

### 1st solution

*[writeup1.md](writeup1.md)*

1. Find the server ip, scan local network -> **nmap**
    - 192.168.1.22
2. Port scan on the server -> **nmap**
    - http
    - https
    - ftp
    - ssh
3. List https domains -> **dirb**
    - https://192.168.1.22/forum/
    - https://192.168.1.22/phpmyadmin/ 
    - https://192.168.1.22/webmail/
4. Forum login -> credentials found on a forum post
    - `lmezard`
    - `q\]Ej?*5K5cy*AJ`
5. Webmail login -> mail found on the forum user space
    - `laurie@borntosec.net`
    - `q\]Ej?*5K5cy*AJ`
6. PhpMyAdmin login -> credentials found on a mail
    - `root`
    - `Fg-'kKXBj87E:aJ$`
7. Inject php backdoor page in the forum to execute commands -> PhpMyAdmin **SQL request**
    - `https://192.168.1.22/forum/templates_c/backdoor.php`
8. Inject python reverse shell -> **backdoor** guest + **netcat** host
    - shell on server with user `www-data`
9. List kernel version + list popular vulnerabilities
    - Dirty COW (**CVE-2016-5195**)
10. Upload custom Dirty COW exploit + compile + run
    - new user `easywin` with password `easywin`, with root rights

*From 8 to 10 are automated with the scripts `reverse_shell_inject.py` + `exploit_dirtycow.py`*

**ROOT!** :checkered_flag:

### 2nd solution

*[writeup2.md](writeup2.md)*

**Starting at the 1st solution point 8**

8. Inject python reverse shell -> **backdoor** guest + **netcat** host (or scripts `reverse_shell_inject.py` + `listen_reverse_shell.py`)
    - shell on server with user `www-data`
9. FTP login -> credentials found on `/home/LOOKATME`
    - files `README` and `fun`
10. Extract `fun` tar archive -> `tar xvf`
    - many pcap files in the directory `ft_fun`
11. ADD all pcap files in good order in a c file and compile it -> `pcap_files_to_c.py` + **gcc**
    - a program print a password
13. SSH connection -> user in README and SHA256 of the password
    - `laurie`
    - `330b845f32185747e4f8ca15d40ca59796035c89ea809fb5d30f4da83ecf45a4`
14. Inspect lauries's home
    - files `README` and `bomb`
15. Reverse the binary `bomb` on laurie's home -> **gdb** or **Ghidra**
    - all files and exploit scripts in `laurie` directory
    - a password
16. User login `thor` -> user in README and found password (warning subject error)
    - `thor`
    - `Publicspeakingisveryeasy.126241207201b2149opekmq426135`
17. Inspect thor's home
    - files `README` and `turtle`
18. Reproduce the turtle instructions with a python script -> `thor/turtle.py`
    - word : `SLASH`
19. User login `zaz` -> user in README and md5 of the found word
    - `zaz`
    - `646da671ca01bb5d84dbb5fb2238dc8e`
20. Inspect zaz's home
    - binary file `exploit_me` with root suid
21. Exploit binary `strcpy` overflow -> **gdb** + ret2libc method
    - root shell
22. (Optional) Add a backdoor binary to easily open a `root` shell on a `zaz` ssh session

**ROOT!** :checkered_flag:

## Bonus part

### Bootloader init program overwrite

1. At the boot, spam `Shift` to access to the grub boot menu
2. List boot partitions with `Tab`
    - `live`
3. Enter `live init=/bin/bash`
    - root shell

### Research

- [x] Other kernel vuln
    - linux-exploit-suggester-2
    - manually
- [x] SUID binaries vuln
    - linux-smart-enumeration
    - manually
- [x] open port vuln
    - metasploit

### Some links

https://github.com/diego-treitos/linux-smart-enumeration
https://github.com/jondonas/linux-exploit-suggester-2
https://github.com/rebootuser/LinEnum

https://book.hacktricks.xyz/linux-unix/privilege-escalation
https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS