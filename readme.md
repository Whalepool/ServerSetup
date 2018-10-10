# Whalepool Ubuntu Server Setup  
```bash
apt-get update 
apt-get upgrade  
apt-get install software-properties-common git vim unzip build-essential curl libcurl4-gnutls-dev libexpat1-dev fontconfig unzip libxml2-dev dh-autoreconf libssl-dev sendmail-bin
```   
  
----  
  
# Envionment setup 

#### Set hostname  
`sudo hostname xxxx`   
   
#### Set vim as the default editor   
`sudo update-alternatives --config editor` 

#### Create a new user  
`sudo adduser wp`    

Add to the sudo group     
`usermod -aG sudo wp`    

Add the users 'wp' to the visudoers with no password required to use the sudo command  
`wp      ALL=(ALL) NOPASSWD: ALL`  


#### ultimate vim  

```bash
su wp 
git clone --depth=1 https://github.com/amix/vimrc.git ~/.vim_runtime
sh ~/.vim_runtime/install_awesome_vimrc.shp
```

#### zsh  
```bash
apt-get install git-core zsh  
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"  
```   
[zsh themes](https://github.com/robbyrussell/oh-my-zsh/wiki/themes) - eg: mrtazz, norm 
  
#### termite terminfo  
```bash  
wget https://raw.githubusercontent.com/thestinger/termite/master/termite.terminfo
tic -x termite.terminfo  
```  
  
#### more
[setup passwordless ssh login to the server](https://serverfault.com/questions/241588/how-to-automate-ssh-login-with-password)      
[dejavu ttf fonts](https://askubuntu.com/questions/454703/cant-start-some-apps-problem-might-have-to-do-with-ttf-fonts#454707)   

----  


# Server setup 
- [Security](#security)
- [Python](#python)  
- [RabbitMQ](#rabbitmq)
- [Mongodb](#mongodb)
- [Github](#github)
- [Web Dev Environment](#web-dev-environment)
  
# xserver setup
- [i3 wm](#i3-wm)
- [Linux Friendly Network File Sharing](#linux-friendly-network-file-sharing)
- [Windows Friendly File Sharing](#windows-friendly-file-sharing)
- [Secure Browsing & Connection](#secure-browsing-and-connection)
- [Essential Apps](#essential-apps)
- [Useful Commands](#useful-commands)
  
----

# Security 

#### Install firewall 
`sudo apt-get install ufw`  

Allow ssh/http  
```bash
sudo ufw allow ssh
sudo ufw allow http  
``` 

Enable the firewall, disable incoming, enable outgoing  
```bash 
sudo ufw enable  
sudo ufw default deny incoming  
sudo ufw default allow outgoing  
```  
  
Check the status  
`sudo ufw status verbose`   
  
#### Disable Root login  
Edit `/etc/ssh/sshd_config` to:
```
PermiteRootLogin no
```  
 
Restart ssh  
`sudo service ssh restart`   
  
  
#### Harden network with sysctl settings  
Edit `/etc/sysctl.conf`    
Add the following lines:  
  
```
# IP Spoofing protection
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Ignore ICMP broadcast requests
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Disable source packet routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0 
net.ipv4.conf.default.accept_source_route = 0
net.ipv6.conf.default.accept_source_route = 0

# Ignore send redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# Block SYN attacks
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 5

# Log Martians
net.ipv4.conf.all.log_martians = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Ignore ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0 
net.ipv6.conf.default.accept_redirects = 0
```  
    
Optional, disable pings   
``` 
# Ignore Directed pings
net.ipv4.icmp_echo_ignore_all = 1
```  
  
Reload sysctl  
`sudo sysctl -p`   
   
For more security setup stuff you can check [here](https://www.thefanclub.co.za/how-to/how-secure-ubuntu-1604-lts-server-part-1-basics)   
    
----   
  
# Python

#### [Pyenv](https://github.com/pyenv/pyenv)    
```bash  
sudo apt-get install zlib1g-dev bzip2 libbz2-dev libreadline-dev libssl-dev libsqlite3-dev python3-dev tk-dev libyaml-dev`  
pyenv versions
pyenv install [version]
pyenv global [version]
```  
Add pyenv to the PATH variable in cron [deploying with pyenv](https://github.com/pyenv/pyenv/wiki/Deploying-with-pyenv)
  
#### more
**VRC for Python** ~ *Automatically mock your HTTP interactions to simplify and speed up testing*  
[Install VCR](https://vcrpy.readthedocs.io/en/latest/installation.html)    
**Pocha for Python tests** ~    
[Pocha for tests](https://github.com/rlgomes/pocha) 
    
----   
  
# RabbitMQ 

#### Erlang  
*Erlang is a general-purpose, concurrent, functional programming language*  
[download erlang](https://packages.erlang-solutions.com/erlang/)  
```bash
sudo apt-get install libwxbase3.0-0v5 libwxgtk3.0-0v5
sudo apt-get install -f 
sudo dpkg -i erlang-solutions_1.0_all.deb
```


#### RabbitMQ  
[download rabbitmq](https://www.rabbitmq.com/install-debian.html)
```bash
wget .deb...
sudo dpkg -i .deb
```
   

#### RabbitMQ Config    
- for >= 3.7 use [sample config](https://raw.githubusercontent.com/rabbitmq/rabbitmq-server/master/docs/rabbitmq.conf.example)  
= put in /etc/rabbitmq/  
- edit to enable guest from any ip login and listeners tcp default  
- edit: default user = guess  
- edit: default_pass = a5zYbwGXkXcwBd5C   
- change the password: `sudo rabbitmqctl change_password guest a5zYbwGXkXcwBd5C`  
  
  
#### RMQ Management Console   
`sudo rabbitmq-plugins enable rabbitmq_management`  
check on http://192.168.1.X:15672/  


#### Delayed Message Exchange    
Get the version: `sudo rabbitmqctl status`  
[Download correct version from rabbitmq.com](https://www.rabbitmq.com/community-plugins.html)  
Move .ez to plugins folder eg: /usr/lib/rabbitmq/lib/rabbitmq_server-3.7.2/plugins   
enable plugin: `sudo rabbitmq-plugins enable rabbitmq_delayed_message_exchange` 


----   
  
# Mongodb 
  
#### MongoDB
[Instructions](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)  
  
  
#### Robomongo
Connection: address:port - 192.168.1.X : 27017  


#### Robomongo Desktop Launcher 
[https://help.ubuntu.com/community/UnityLaunchersAndDesktopFiles](https://help.ubuntu.com/community/UnityLaunchersAndDesktopFiles)  
  
#### Pymongo
`sudo apt-get install python-dev`  
    
----   
    

# Github 

#### pub/prv key pair with github  
`cd ~/.ssh && ssh-keygen`  
[Add to github](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)

#### git config  
```bash  
git config --global user.email "email@email.com"  
git config --global user.name "username"  
git config --global core.editor "vim"  
```  
  
#### more 
[contribute to github anonymously through tor](https://stackoverflow.com/questions/10274879/how-to-contribute-on-github-anonymously-via-tor)  

---   
  
# Web Dev Environment

####  APR  
[Download APR](https://apr.apache.org/download.cgi)  
```bash
./configure
make
sudo make install
```

#### APR Utils    
[Download APR UTILS](https://apr.apache.org/download.cgi)  
```bash  
./configure --with-apr=/usr/local/apr/bin/apr-1-config   
make   
sudo make install 
```  

#### PCRE (NOT PCRE2 !!!)  
[ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/](ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/)  
```bash  
./configure --prefix=/usr/local/pcre   
make   
sudo make install   
```   

#### HTTPD  
[Download HTTPD](https://httpd.apache.org/download.cgi)  
```bash  
./configure --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr --with-pcre=/usr/local/pcre/bin/pcre-config --enable-so `  
make   
sudo make install   
``` 

Set  HTTPD 'ServerName'~ *Avoids apache server name error/notice on startup*  
`sudo vim /usr/local/apache2/conf/httpd.confg`


Set the root web directory to be that in your home directory  
```bash
sudo rm /usr/local/apache2/htdocs/index.html
cd ~
ln -s /usr/local/apache2/htdocs/ www
sudo chown [username]:[username] /usr/local/apache2/htdocs   
sudo /usr/local/apache2/bin/apachectl -k start  
```
  
#### PHP
[download php](https://secure.php.net/downloads.php)  



# Linux Friendly Network File Sharing 

**NFS** ~ *Enable remote mounting your server from your other linux machines*  
[digialocean.com how to setup an nfs mount on ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-16-04)   

**Server Side**  
Install support for NFS kernel server  
`sudo apt-get install nfs-kernel-server`    

Edit `/etc/exports` and setup the following line  
`/home/[username]/www [local-ip](rw,sync,no_root_squash,no_subtree_check)`    

Reload the new exports file  
`sudo exportfs -ra`

**Client Side**   
Install NFS support files common to client and server  
`sudo apt-get install nfs-common`  
  
 Make a directory in your home directory  
 `mkdir ~/remote-server`  

 Mount your remote server  
 `sudo mount [ip address]:/path/on/remote/server /path/on/local/machine`  
    
----   
  
# Windows Friendly File Sharing 

**Samba** ~ *SMB/CIFS file, print, and login server for Unix*  
Install Samba  
`sudo apt-get install samba`  
  
Setup a Samba user   
`sudo smbpasswd -a [username]`   
  
Backup original samba config file  
`sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.orig`  
  
Edit the config file  
`sudo vim /etc/samba/smb.conf`  
  
Add to the end of the file:  
```
[www]
path = /home/[username]/www
valid users = [username]
read only = no
```
  
Restart samba  
`sudo service smbd restart`
  
# Secure Browsing and Connection 
   
**Tor Browser**     
[https://github.com/TheTorProject/gettorbrowser](https://github.com/TheTorProject/gettorbrowser)    
        
**Private Internet Access VPN**  
Download the openvpn-ip files from here [https://www.privateinternetaccess.com/openvpn/openvpn-ip.zip](https://www.privateinternetaccess.com/openvpn/openvpn-ip.zip)    
`sudo apt-get install openvpn`    
Extract the ovpn+crl+ca file to /libs/openvpn-ip.zip to /etc/openvpn     
`cd /etc/openvpn`    
`sudo openvpn --config uk-manchester.ovpn`    
      
**IPTables** ~ *to force either vpn connection or no connection*  
Set the ip to be that of the ovpn file     
Check the lan communication ip    
    
----   
  
# i3 wm  

#### Install   
`sudo apt-get install i3 xorg-init dmenu` 
  
#### Configure the monitors   
If there is a second/third monitor and it hasn't been identified or placed properly, firstly identify the monitor device names with:   
`xrandr`  
Make sure the secondary monitors are not turned off:  
`xrandr --output Display-1 --auto --off`   
Set the monitors and placement:  
`xrandr --output Display-1 --right-of Display-2`  
     
####  i3 config additions  
`vim .config/i3/confg`   
  
```
# Make the default terminal gnome-terminal
bindsym $mod+Return exec termite 

# Launch Chrome
bindsym $mod+Shift+Return exec chromium --incognito 

# Take a screenshot
bindsym $mod+Print exec screenshot

# Disable the screensaver
exec --no-startup-id xset s off
exec --no-startup-id xset s noblank
exec --no-startup-id xset -dpms

# Command to lock the computer 
bindsym $mod+Control+l exec i3lock -c 000000 -f

# Set some display variables from xrandr 
set $display1 Display-1
set $display2 Display-2 

# Make preset the workspaces to each screen
workspace 0 output $display1
workspace 1 output $display1
workspace 2 output $display1
workspace 3 output $display1
workspace 4 output $display2
workspace 5 output $display2
workspace 6 output $display2
workspace 7 output $display2
workspace 8 output $display2
workspace 9 output $display2


```   

#### Reload i3    
reload i3 with   
`mod+shift+r` to reload the configuration file   
`mod+shift+r` to restart i3 inplace with existing layout/sesion   



# Essential Apps 
  
**Red shift** ~ *Adjusts the color temperature of your screen according to your surroundings*  
[Github](https://github.com/jonls/redshift)  
  
**KeepassXC** ~ *password manager*    
[https://keepassxc.org/](https://keepassxc.org/)    
  
  
**Usernet**  
Create email address with cock.li     
Ceate account on usenetbucket.com    
  
**Sabnzbd**  
Sign in/install [sabnzbd.org](https://sabnzbd.org/wiki/installation/install-ubuntu-repo) - access at http://127.0.0.1:8080/sabnzbd    
  
**Sonarr**   
Install [sonarr.tv](https://sonarr.tv/#home) - radarr.video - configure them with the nzb service  - access at http://localhost:8989/    
  
  
**Delgue** ~ *torrent downloader* 
Install [link](https://dev.deluge-torrent.org/wiki/Installing/Linux/Ubuntu)   
  
  
**Nano** ~ *a small i3 compatible gui file browser*  
`sudo apt-get install nano` 
  

**Telegram** ~ *messaging client*  
Download [link](https://desktop.telegram.org/)  
  
   
**qTox** ~ *messaging client*  
Download [link](https://github.com/qTox/qTox)  
  

**Teamspeak** ~ *voice chatting client*  
Download [link](https://www.teamspeak.com/en/downloads)  

   
**Filezilla** ~ *ftp client*  
`sudo apt-get install filezilla`  
  

**ckb-next** ~ *Corsair keyboard support*  
May or may not be needed depending on your keyboard, but if you have a corsair keyboard that isn't supported this is the package you'll want  
Download [link](https://github.com/ckb-next/ckb-next)  
  

**youtube-dl** ~ *Download and rip youtube vidoes/extract audio etc*  
Download [link](https://rg3.github.io/youtube-dl/download.html)  
Example of extracting the audio as mp3:  
` youtube-dl -x --audio-format="mp3" [url]`  
  
   
**Maim** ~ *quick and easy screenshots*   
`sudo apt-get install maim`  
To add a screenshot command, `mkdir /home/$user/Pictures/Screenshots` and add to `/home/$user/bin`:  
```bash  
#!/bin/bash
maim -s ~/Pictures/Screenshots/$(date +%s).png
```   
  
  
**Chromium** ~ *open source version of chrome*  
`sudo apt-get install chromium-browser`  
  
 Recommended extensions:   
 - [Adblock](https://chrome.google.com/webstore/detail/adblock/gighmmpiobklfepjocnamgkkbiglidom)  
 - [GitHub Flavoured Markdown](https://chrome.google.com/webstore/detail/github-flavored-markdown/faelggnmhofdamhdegcdhhemfokkfngk)  
 - [Keep Awake](https://chrome.google.com/webstore/detail/keep-awake/bijihlabcfdnabacffofojgmehjdielb)  
 - [Lightshot Screenshot](https://chrome.google.com/webstore/detail/lightshot-screenshot-tool/mbniclmhobmnbdlbpiphghaielnnpgdp)  
 - [Reddit Enhancement Suite](https://chrome.google.com/webstore/detail/reddit-enhancement-suite/kbmfpngjjgdllneeigpgjifpgocmfgmb)  
  

# Useful Commands

**ffmpeg ogv to mp4**  
`ffmpeg -i lag.ogv -c:v libx264 -preset veryslow -crf 22 -c:a libmp3lame -qscale:a 2 -ac 2 -ar 44100 lag.mp4` 
  

**Updatedb verbose mode**   
`sudo updatedb.mlocate -v | tee updatedb.log`  
  
   
 # Useful Links 

 - Fake name generator [link](https://www.fakenamegenerator.com/gen-random-ch-br.php)