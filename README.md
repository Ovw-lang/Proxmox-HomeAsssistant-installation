# Proxmox on NUC with Home Assistant
Proxmox VE installation on a NUC with home Assistant

## Table of contents
* [Installation](#Installation)
* [Configuration](#Configuration)
* [Update](#Update)
* [HA Installation](#HA)


## Installation

* Download newest proxmox installation image from ```https://pve.proxmox.com/wiki/Installation```
* Use Etcher ```https://etcher.io/``` to install Proxmox image on a bootable USB drive

## Configuration

For newer version first look at: ```https://gist.github.com/whiskerz007/53c6aa5d624154bacbbc54880e1e3b2a```

# Disable Commercial Repo
```sed -i "s/^deb/\#deb/" /etc/apt/sources.list.d/pve-enterprise.list```
```apt-get update```

# Add PVE Community Repo
```echo "deb http://download.proxmox.com/debian/pve $(grep "VERSION=" /etc/os-release | sed -n 's/.*(\(.*\)).*/\1/p') pve-no-subscription" > /etc/apt/sources.list.d/pve-no-enterprise.list```
```apt-get update```

# Remove nag (disable 'no subscription' message when logon)
```echo "DPkg::Post-Invoke { \"dpkg -V proxmox-widget-toolkit | grep -q '/proxmoxlib\.js$'; if [ \$? -eq 1 ]; then { echo 'Removing subscription nag from UI...'; sed -i '/data.status/{s/\!//;s/Active/NoMoreNagging/}' /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js; }; fi\"; };" > /etc/apt/apt.conf.d/no-nag-script```
```apt --reinstall install proxmox-widget-toolkit```

# Update
Go to Updates and make sure that all newest updates is installed. 

<a name="ha"></a>
# Home Assistant VM Deployment
