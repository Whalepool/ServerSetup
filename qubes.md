# Whalepool Qubes Desktop Setup    
    
Qubes, found at https://www.qubes-os.org/ - Is a good linux desktop OS. 
For those involved in crypto, Qubes is a good operationg system. Using security by separation of concerns.  

Here's some useful code from my experiences with Qubes.    
    
#### Tips  
- Backup / Clone template vm's before making any modifications to them   
- Use and create disposable vm's as much as possible, non persistent filesystems are good   
- Install i3 for more efficient workflow  
- Create custom desktop launchers for what you need   
- Update the stock template vm's 
   
#### Useful links
qvm-screenshot-tool : https://github.com/evadogstar/qvm-screenshot-tool   
qubes i3 : https://www.qubes-os.org/doc/i3/   
laurent tools : https://git.zx2c4.com/laurent-tools/tree/tools
    
---   

# Useful Commands / Snippets   
   
### Install i3
First things first. Not working anywhere without i3   
```bash  
sudo qubes-dom0-update i3 i3-settings-qubes

# Check xrandr settings are working
xrandr

# eg, move monitor placement 
xrandr --output DVI-0 --left-of DisplayPort-1   
```     
    
Some i3 config things that are good   
```bash  
hide_edge_borders vertical  
bindsym Control+Shift+X [class="^.*"] border toggle   
for_window [class="mpv"] border pixel 0   
```   
  
### Enable full screen mode for selected vms   
```bash  
sudo vim /etc/qubes/guid.conf  
```  
and declare what you want to have full screen permissions  
```bash  
VM: {   
	work: {   
		allow_fullscreen = true;   
	};  
	home: {  
		allow_fullscreen = true;   
	};  
}  
```        

### dom0 overview commands   
```bash   
# top/htop equivilant 
# Displays real time information about Xen system and domains   
xentop   

# vm Network information
qvm-ls --net

# vm Disk usage information 
qvm-ls --net 

# vm General summary 
qvm-ls --all 
   
# Keep an eye on cpu temps 
watch sensors   

# Get block id's 
sudo blkid 
```    

### Block devices     
```bash     
# List block devices  
qvm-block list   
  
# Attached a block device, example, nvme0n1 
qvm-block attached some-app-vm dom0:nvme0n1  
```     

### Devices     
```bash    
# Example of adding a mic device    
qvm-device mic attach some-app-vm dom0:mic     
# Detach device     
qvm-device mic detach some-app-vm dom0:mic      
```       
   
### Updating stock templates   
```bash    
# Update to the newest fedora template  
qubes-dom0-update qubes-template-fedora-29   
```    
   
### Other Essential dom0 installs   
```bash    
qubes-dom0-update redshift 
```    
    
### Create a disposable vm from a template   
```bash    
# Create the custom-dvm app-vm from the template vm
# Labels color options https://github.com/QubesOS/qubes-core-admin/blob/5c08d0e2e3df358ffc79cf217f5998b84b0249c8/core/qubes.py#L925-L935   
# red, orange, yellow, green, gray, blue, purple, black   
qvm-create --template fedora-29 --label red my-custom-dvm   
  
# Tell Qubes this app vm is a template for dispoable vms  
qvm-prefs my-custom-dvm template_for_dispvms true   
   
# Set as the default disposable vm if you want  
qvm-prefs default_disvm my-custom-dvm   
   
# Launch something with your new custom disposable vm  
qvm-run --dispvm=my-custom-dvm gnome-terminal  
```   
   
### Custom Desktop launcher   
```bash   
cd ~/.local/share/applications  
vim my-launcher.desktop  
```    
This is some sample code for a simple desktop launcher file contents    
```bash    
[Desktop Entry]   
Encoding=UTF-8     
Version=1.0    
Type=Application   
Terminal=false   
# Custom vm 
Exec=qvm-run -a --dispvm=my-custom-dvm 'google-chrome'    
# Normal VM   
# Exec=qvm-run -a some-appvm google-chrome  
Name=chrome.dispm      
```  
Additional desktop shortcuts I recommend:   
- qubes-qube-manager  
    
### Essentials to install on a fedora template vm    
```bash    
sudo dnf config-manage set-enabled rpmfusion-free    
sudo vim /etc/yum.repos.d/rpmfusion-nonfree.repos    
sudo vim /etc/yum.repos.d/google-chrome.repo     
    
sudo dnf install ffmpeg    
# Browsing   
# google-chrome    
    
# Media    
# vlc, mpv, gimp    
     
# Networking    
# nfs-utils, openvpn, filezilla    
    
# Social    
# telegram-desktop, discord, qtox    
    
# Downloads   
# deluge, youtube-dl    
    
# File browsing   
# nemo     
    
# Management    
# keepassxc    
    
# Essentials    
# zsh, git, unzip, mlocate    
    
    
# Dotnet    
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc    
wget -q https://packages.microsoft.com/config/fedora/27/prod.repo    
sudo mv prod.repo /etc/yum.repos.d/microsoft-prod.repo    
sudo chown root:root /etc/yum.repos.d/microsoft-prod.repo    
sudo dnf update      
sudo dnf install dotnet-sdk-2.2    
```      
     
### Mount other local linux servers      
```bash      
# Mount your other local server     
sudo mount -t nfs 192.168.1.1:/path/to/shae local-folder-name       
```   
   
### Setup a vpn net-vm    
Create a new qube, type, AppVM, template, whatever, networking, sys-firewall, advanced, 'provides network'       
```bash   
cd /rw/config   
sudo mkdir vpn   

# Copy the openvpn-ip.zip to the current appvm   
cd ~/QubesIncoming/from-app-vpm/  

# Move the .crt, .pem and .ovpn file to the vpn folder, eg:    
sudo mv ca.rsa.2048.crt /rw/config/vpn/   
sudo mv crl.rsa.2048.pem /rw/config/vpn/   
sudo mv UK\ London.ovpn /rw/config/vpn/openvpn-client.ovpn  
   
# Back to the vpn dir
cd /rw/config/vpn   
  
# Edit the vpn client file
sudo vim openvpn-client.ovpn   

# Append to file...
redirect-gateway def1   
auth-user-pass pass.txt   
script-security 2   
up 'qubes-vpn-handler.sh up'   
down 'qubes-vpn-handler.sh down'  
  
# Save it   
# Create the password file with your vpn login user/pass
sudo vim pass.txt  
  
# EG:
username-on-line1  
password-on-line2  
  
# Save it 
# Create the qubes-vpn-handler.sh file  
  
sudo vim qubes-vpn-handler.sh   

# Paste in the following   
#!/bin/bash
set -e
export PATH="$PATH:/usr/sbin:/sbin"

case "$1" in

up)
# To override DHCP DNS, assign DNS addresses to 'vpn_dns' env variable before calling this script;
# Format is 'X.X.X.X  Y.Y.Y.Y [...]'
if [[ -z "$vpn_dns" ]] ; then
    # Parses DHCP foreign_option_* vars to automatically set DNS address translation:
    for optionname in ${!foreign_option_*} ; do
        option="${!optionname}"
        unset fops; fops=($option)
        if [ ${fops[1]} == "DNS" ] ; then vpn_dns="$vpn_dns ${fops[2]}" ; fi
    done
fi

iptables -t nat -F PR-QBS
if [[ -n "$vpn_dns" ]] ; then
    # Set DNS address translation in firewall:
    for addr in $vpn_dns; do
        iptables -t nat -A PR-QBS -i vif+ -p udp --dport 53 -j DNAT --to $addr
        iptables -t nat -A PR-QBS -i vif+ -p tcp --dport 53 -j DNAT --to $addr
    done
    su - -c 'notify-send "$(hostname): LINK IS UP." --icon=network-idle' user
else
    su - -c 'notify-send "$(hostname): LINK UP, NO DNS!" --icon=dialog-error' user
fi

;;
down)
su - -c 'notify-send "$(hostname): LINK IS DOWN !" --icon=dialog-error' user
;;
esac    
   
# Save it 
# Make it executable
sudo chmod +x qubes-vpn-handler.sh     
   
# Make the qubes ip change hook script
cd /rw/config   
sudo vim qubes-ip-change-hook  
   
# Enter the following details   
#!/bin/bash
#    Block forwarding of connections through upstream network device
#    (in case the vpn tunnel breaks):
iptables -I FORWARD -o eth0 -j DROP
iptables -I FORWARD -i eth0 -j DROP
ip6tables -I FORWARD -o eth0 -j DROP
ip6tables -I FORWARD -i eth0 -j DROP
   
#    Block all outgoing traffic
iptables -P OUTPUT DROP
iptables -F OUTPUT
iptables -I OUTPUT -o lo -j ACCEPT
   
#    Add the `qvpn` group to system, if it doesn't already exist
if ! grep -q "^qvpn:" /etc/group ; then
     groupadd -rf qvpn
     sync
fi
sleep 2s
   
#    Allow traffic from the `qvpn` group to the uplink interface (eth0);
#    Our VPN client will run with group `qvpn`.
iptables -I OUTPUT -p all -o eth0 -m owner --gid-owner qvpn -j ACCEPT
  
# Save it, make it executable
sudo chmod +x qubes-ip-change-hook  
  
# Edit the rc.local to auto run the vpn file 
sudo vim rc.local   
   
# Add the following
#!/bin/bash
VPN_CLIENT='openvpn'
VPN_OPTIONS='--cd /rw/config/vpn/ --config openvpn-client.ovpn --daemon'

su - -c 'notify-send "$(hostname): Starting vpn connection..." --icon=network-idle' user
groupadd -rf qvpn ; sleep 2s
sg qvpn -c "$VPN_CLIENT $VPN_OPTIONS"
```   
   
### Script to auto start vm, attach & mount block device   
Useful little script for opening for example a 'bitcoind' vm, with auto start/attach/mount if not already running.
Can then call this script from a .desktop launcher  
```bash    
#!/bin/bash       
list=`qvm-ls --raw-data`    
is_bitcoind_running=`awk '/bitcoind\|Running/' <<< "$list"`    
   
echo "is_bitcoind_running: $is_bitcoind_running"   
   
if [ -z $is_bitcoind_running ]
then 
	echo "bitcoind is not running"    
	echo "starting bitcoind.... "   
	qvm-start -v bitcoind    
   
	echo "attaching block device..."
	qvm-block -v attach bitcoind dom0:nvme0n1    
    
	echo "mounting nvme drive on bitcoind vm"
	qvm-run -a bitcoind 'sudo mount /dev/xvdi1 /home/user/bitcoind'    
    
	echo "opening gnome terminal"   
	qvm-run -a bitcoind 'gnome-terminal'    
else
	echo "bitcoind is running, opening gnome terminal"    
	qvm-run -a bitcoind 'gnome-terminal'   
fi   
```
