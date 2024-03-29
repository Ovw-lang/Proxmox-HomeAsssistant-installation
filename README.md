# Proxmox with Home Assistant and auto backup Synology

![banner](https://user-images.githubusercontent.com/45032723/96386318-4f6b4d00-119a-11eb-8ac6-fb4025eae0eb.png) 

*Current hardware: NUC10FNK*

## Table of contents
* [Installation](#Installation)
* [Configuration](#Configuration)
* [Update](#Update)
* [Home Assistant VM Installation](#HA)
* [Home Assistant Add-ons]($HA_addons)
* [HA VM Automatic Backups to Synology](#backup)
* [Restore HA VM](#restore)

## Installation

* Download newest proxmox installation image from ```https://pve.proxmox.com/wiki/Installation```
* Use Etcher ```https://etcher.io/``` to install Proxmox image on a bootable USB drive

## Configuration

For newer version first look at: ```https://gist.github.com/whiskerz007/53c6aa5d624154bacbbc54880e1e3b2a```

### Disable Commercial Repo
```sed -i "s/^deb/\#deb/" /etc/apt/sources.list.d/pve-enterprise.list```

```apt-get update```

### Add PVE Community Repo
```
echo "deb http://download.proxmox.com/debian/pve $(grep "VERSION=" /etc/os-release | sed -n 's/.*(\(.*\)).*/\1/p') pve-no-subscription" > /etc/apt/sources.list.d/pve-no-enterprise.list```

apt-get update
```

### Remove nag (disable 'no subscription' message when logon)
```
echo "DPkg::Post-Invoke { \"dpkg -V proxmox-widget-toolkit | grep -q '/proxmoxlib\.js$'; if [ \$? -eq 1 ]; then { echo 'Removing subscription nag from UI...'; sed -i '/data.status/{s/\!//;s/Active/NoMoreNagging/}' /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js; }; fi\"; };" > /etc/apt/apt.conf.d/no-nag-script

apt --reinstall install proxmox-widget-toolkit
```

## Update
Go to Updates and make sure that all newest updates is installed. Reboot after installation is completed.

<a name="ha"></a>
## Deploy (Supported) Home Assistant OS with a simple script

*I'm using the script from whiskerz007. Before using this script, check his github page for the newest version: `https://github.com/whiskerz007/proxmox_hassos_install`*

### New Proxmox VM with Home Assistant

This script will create a new Proxmox VM with the latest version of Home Assistant. To create a new VM, run the following in a SSH session or the console from Proxmox interface

```
bash -c "$(wget -qLO - https://github.com/whiskerz007/proxmox_hassos_install/raw/master/install.sh)"
```

After script completes, click on the new VM (_the script will tell you the ID_), click on the `Hardware` tab for the VM and change the `Memory` and `Processors` settings to what you desire. The `Hard Disk` can be expanded by clicking on it, then click on the `Resize disk` button above (_Note: additional steps must be taken for storage to take effect in the VM after the first boot_). The network MAC address can be changed by selecting `Network Device` and clicking `Edit` above. Once all changes have been made, click `Start` above.

### Root Prompt

To get to the root prompt
- Open the console after the VM has been started
- When the messages slow down press the `Enter` key a couple of times until you see the following
```

Welcome to Home Assistant
homeassistant login:
```
- Login using `root`, no password is requested
- When you see the `hassio > ` prompt, type `login`
- You should now see a `# ` prompt.

### Add a serial port

By adding a serial port, you are able to use a different interface to interact with the VM. When you click on the down arrow next to `Console` you will be able to use `xterm.js` which enables you to `Right-Click` and get access to `Copy` and `Paste` functions. If the serial port was already added by the install script, no further actions are required to enable the functionality.
- Click on the VM in the list of containers at the left side panel
- Click `Hardware` tab located beside the list of containers
- Click `Add` located beside `Summary` tab, then click `Serial Port`
- `Serial Port` should be set to `0` in the input box, then click `Add`
- Start the VM, if it isn't already
- At the root prompt type `sed -i 's/$/ console=ttyS0/' /mnt/boot/cmdline.txt`
- A `Shutdown` and `Start` is required for the changes to take effect

### Show Current IP Address

To get the current IP address assigned to the VM from the Proxmox interface
- Click on the VM in the list of containers at the left side panel
- Click `Summary` tab located beside the list of containers
- Click `More` near `IPs` in the top left section
- You can find the assigned IP addresses on the line with the name similar to `enp0s18`

To get the current IP address assigned to the VM from the command line
- At the root prompt type `nmcli -g ip4.address d sh $(nmcli -g device c)`
- The response will be the IP address with subnet mask or nothing

**Note:** _If DHCP is configured and nothing is shown, check DHCP server and VM network settings_

### Configure Network for Static IP Address

To set a static IP address, use the following as an example
- At the root prompt type `nmcli c mod $(nmcli -g uuid c) ipv4.method manual ipv4.addresses "192.168.10.80/24" ipv4.gateway "192.168.10.254" ipv4.dns "192.168.10.254"`
- At the root prompt type `nmcli c up $(nmcli -g uuid c)`

### Configure Network for DHCP

To remove all static IP addresses and enable DHCP
- At the root prompt type `nmcli c mod $(nmcli -g uuid c) ipv4.method auto ipv4.addresses "" ipv4.gateway "" ipv4.dns ""`
- At the root prompt type `nmcli c up $(nmcli -g uuid c)`

### Default Interface Name

To get the default interface name
- At the root prompt type `nmcli -g device c`
- The response with be the interface name

### Change Hostname

To change the HassOS VM hostname
- At the root prompt type `hostnamectl set-hostname your-new-hostname`
- You can verify the change by logging out with `exit`, the last line printed will be `your-new-hostname login: `

### Resize Disk

To resize the disk after the first boot
- At the root prompt type `df -h /dev/sda8` and note the `Size`
- Shutdown the VM
- Resize the disk to the desired size
- At the root prompt type `sgdisk -e /dev/sda`
- At the root prompt type `reboot`
- Verify resize was successful by typing `df -h /dev/sda8` at the root prompt

<a name="backup"></a>
## Home Assistant Add-ons installed

### Official Add-ons 
1. ESPHome
2. File editor
3. Mosquitto broker
4. Samba share

### Non-official Add-ons
1. HACS ```https://hacs.xyz/```

#### HACS Add-ons
1. Lovelace Swipe Navigation ```https://github.com/maykar/lovelace-swipe-navigation```
2. Mini Media Player ```https://github.com/kalkih/mini-media-player```
3. Vacuum Card ```https://github.com/denysdovhan/vacuum-card```
4. Mini Graph Card ```https://github.com/kalkih/mini-graph-card```

<a name="backup"></a>
## Home Assistant automatic backups to Synology 

### Synology 
1. Create a shared folder in Synology.
2. Create a new user Proxmox and give this account write acces on the new shared folder.
3. Go to > File Services Enable SMB amd under advanced Enable SMBv3

### Proxmox
1. Go to > Proxmox Datacenter > Storage and add CIFS storage

![Add Storage](https://user-images.githubusercontent.com/45032723/96386286-fc919580-1199-11eb-9338-9a6bc0f65f17.png) 

2. Fill out the CIFS form:

![Image of backup](https://user-images.githubusercontent.com/45032723/96386234-8856f200-1199-11eb-9da8-95bb263be438.png)

```
ID: synology
Server: 192.168.10.10
Username: proxmox
Password: xxx
Share: Sysbackups
Max Backups: 2
Content: VZDump backup file, Disk image, ISO image, Container
```
3. Go to > Backup > Add > Storage Synology and use LZO(FAST) as compression

![backup](https://user-images.githubusercontent.com/45032723/96386645-c43f8680-119c-11eb-8f5e-fc319f73e3a8.png)

4. Select 'run now' for first backup and test run.

<a name="restore"></a>
## Home Assistant restore backup to Proxmox

1. Go to the Synology storage
2. Select the backup file and click on `Restore`

![Restore](https://user-images.githubusercontent.com/45032723/96386772-9f97de80-119d-11eb-8e0f-3b98a45f5ae8.png)
3. Choose an VM ID and click again on `Restore`

![Restore_again](https://user-images.githubusercontent.com/45032723/96386797-cfdf7d00-119d-11eb-9c21-88261c2c1043.png)

4. Restore progress is shown in the Task Viewer. 

![Restore_progress](https://user-images.githubusercontent.com/45032723/96386839-2b116f80-119e-11eb-85bb-bf0e0af4f4a0.png)
