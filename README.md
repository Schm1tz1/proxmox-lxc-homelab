# Proxmox-LXC-Homelab
The following is for creating our Homelab LXC containers.

Network Prerequisites are:
- [x] Layer 2 Network Switches
- [x] Network Gateway is `192.168.1.5`
- [x] Network DNS server is `192.168.1.5` (Note: your Gateway hardware should enable you to a configure DNS server(s), like a UniFi USG Gateway, so set the following: primary DNS `192.168.1.254` which will be your PiHole server IP address; and, secondary DNS `1.1.1.1` which is a backup Cloudfare DNS server in the event your PiHole server 192.168.1.254 fails or os down)
- [x] Network DHCP server is `192.168.1.5`
- [x] A DDNS service is fully configured and enabled (I recommend you use the free Synology DDNS service)
- [x] A ExpressVPN account (or any preferred VPN provider) is valid and its smart DNS feature is working (public IP registration is working with your DDNS provider)

Other Prerequisites are:
- [x] Synology NAS, or linux variant of a NAS, is fully configured as per [SYNOBUILD](https://github.com/ahuacate/synobuild#synobuild)
- [x] Proxmox node fully configured as per [PROXMOX-NODE BUILDING](https://github.com/ahuacate/proxmox-node/blob/master/README.md#proxmox-node-building)
- [x] pfSense is fully configured on typhoon-01 including both OpenVPN Gateways VPNGATE-LOCAL and VPNGATE-WORLD.

Tasks to be performed are:
- [ ] 1.00 PiHole LXC - CentOS7
- [ ] 2.00 UniFi Controller - CentOS7
- [ ] 3.00 NextCloud LXC - Turnkey Debian

## About LXC Homelab Installations
This page is about installing Proxmox LXC's and VM's for your homelab network. Software tools like PiHole, cloud storage and stuff.

Proxmox itself ships with a set of basic templates and to download a prebuilt OS distribution use the graphical interface `typhoon-01` > `local` > `content` > `templates` and select and download the following templates:
*  `centos-7-default`;
*  `ubuntu-18.04-standard`; *and,*
*  `turnkey-nextcloud`.

---

## 1.00 PiHole LXC - CentOS7
Here we are going install PiHole which is a internet tracker blocking application which acts as a DNS sinkhole. Basically its charter is to block advertisments, tracking domains, tracking cookies and all those personal data mining collection companies.

### 1.01 Create a CentOS7 LXC for PiHole - CentOS7
Now using the web interface `Datacenter` > `Create CT` and fill out the details as shown below (whats not shown below leave as default):

| Create: LXC Container | Value |
| :---  | :---: |
| **General**
| Node | `typhoon-01` |
| CT ID |`254`|
| Hostname |`pihole`|
| Unprivileged container | `☑` |
| Resource Pool | Leave Blank
| Password | Enter your pasword
| Password | Enter your pasword
| SSH Public key | Add one if you want to
| **Template**
| Storage | `local` |
| Template |`centos-7-default_xxxx_amd`|
| **Root Disk**
| Storage |`typhoon-share`|
| Disk Size |`8 GiB`|
| **CPU**
| Cores |`1`|
| CPU limit | Leave Blank
| CPU Units | `1024`
| **Memory**
| Memory (MiB) |`256`|
| Swap (MiB) |`256`|
| **Network**
| Name | `eth0`
| Mac Address | `auto`
| Bridge | `vmbr0`
| VLAN Tag | Leave Blank
| Rate limit (MN/s) | Leave Default (unlimited)
| Firewall | `☑`
| IPv4 | `☑  Static`
| IPv4/CIDR |`192.168.1.254/24`|
| Gateway (IPv4) |`192.168.1.5`|
| IPv6 | Leave Blank
| IPv4/CIDR | Leave Blank |
| Gateway (IPv6) | Leave Blank |
| **DNS**
| DNS domain | Leave Default (use host settings)
| DNS servers | Leave Default (use host settings)
| **Confirm**
| Start after Created | `☑`

And Click `Finish` to create your PiHole LXC.

Or if you prefer you can simply use Proxmox CLI `typhoon-01` >  `>_ Shell` and type the following to achieve the same thing (note, you will need to create a password for PiHole LXC):
```
pct create 254 local:vztmpl/centos-7-default_20171212_amd64.tar.xz --arch amd64 --cores 1 --hostname pihole --cpulimit 1 --cpuunits 1024 --memory 256 --net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.1.5,ip=192.168.1.254/24,type=veth --ostype centos --rootfs typhoon-share:8 --swap 256 --unprivileged 1 --onboot 1 --startup order=1 --password
```

### 1.02 Install PiHole - CentOS7
First Start your `254 (pihole)` LXC container using the web interface `Datacenter` > `254 (pihole)` > `Start`. Then login into your `254 (pihole)` LXC by going to  `Datacenter` > `254 (pihole)` > `>_ Console and logging in with username `root` and the password you created in the previous step 1.1.

Now using the web interface `Datacenter` > `254 (pihole)` > `>_ Console` run the following command:
```
curl -sSL https://install.pi-hole.net | bash
```
The PiHole installation package will download and the installation will commence. Follow the prompts making sure to enter the prompts and field values as follows:

| PiHole Installation | Value | Notes
| :---  | :---: | :--- |
| PiHole automated installer | `<OK>` | *Just hit your ENTER key*
| Free and open source | `<OK>` | *Just hit your ENTER key*
| Static IP Needed | `<OK>` | *Just hit your ENTER key*
| Select UPstream DNS Provider | `Cloudfare` | *And tab key to highlight <OK> and hit your ENTER key*
| Pihole relies on third party .... | Leave Default, all selected | *And tab key to highlight <OK> and hit your ENTER key*
| Select Protocols | Leave default, all selected | *And tab key to highlight <OK> and hit your ENTER key*
| Static IP Address | Leave Default | *It should show IP Address: 192.168.1.254/24, and Gateway: 192.168.1.5. And tab key to highlight <Yes> and hit your ENTER key*
| FYI: IP Conflict | Nothing to do here | *And tab key to highlight <OK> and hit your ENTER key*
| Do you wish to install the web admin interface | `☑` On (Recommended) | *And tab key to highlight <OK> and hit your ENTER key*
| Do you wish to install the web server |  `☑` On (Recommended) | *And tab key to highlight <OK> and hit your ENTER key*
| Do you want to log queries? |  `☑` On (Recommended) | *And tab key to highlight <OK> and hit your ENTER key*
| Select a privacy mode for FTL |  `☑` 0 Show Everything  | *And tab key to highlight <OK> and hit your ENTER key*
| **And the installation script will commence ...**
| Installation Complete | `<OK>` | *Just hit your ENTER key*

Your installation should be complete.

### 1.03 Reset your PiHole webadmin password - CentOS7
Now reset the web admin password using the web interface `Datacenter` > `254 (pihole)` > `>_ Console` run the following command:
```
pihole -a -p
```
You can now login to your PiHole server using your preferred web browser with the following URL http://192.168.1.254/admin/index.php

### 1.04 Enable DNSSEC - CentOS7
You can enable DNSSEC when using Cloudfare which support DNSSEC. Using the PiHole webadmin URL http://192.168.1.254/admin/index.php go to `Settings` > `DNS Tab` and enable `USE DNSSEC` under Advanced DNS Settings. Click `Save`.

---

## 2.00 UniFi Controller - CentOS7
Rather than buy a UniFi Cloud Key to securely run a instance of the UniFi Controller software you can use Proxmox LXC container to host your UniFi Controller software.

For this we will use a CentOS LXC container.

### 2.01 Create a CentOS7 LXC for UniFi Controller - CentOS7
Now using the web interface `Datacenter` > `Create CT` and fill out the details as shown below (whats not shown below leave as default):

| Create: LXC Container | Value |
| :---  | :---: |
| **General**
| Node | `typhoon-01` |
| CT ID |`251`|
| Hostname |`unifi`|
| Unprivileged container | `☑` |
| Resource Pool | Leave Blank
| Password | Enter your pasword
| Password | Enter your pasword
| SSH Public key | Add one if you want to
| **Template**
| Storage | `local` |
| Template |`centos-7-default_xxxx_amd`|
| **Root Disk**
| Storage |`typhoon-share`|
| Disk Size |`8 GiB`|
| **CPU**
| Cores |`1`|
| CPU limit | Leave Blank
| CPU Units | `1024`
| **Memory**
| Memory (MiB) |`1024`|
| Swap (MiB) |`256`|
| **Network**
| Name | `eth0`
| Mac Address | `auto`
| Bridge | `vmbr0`
| VLAN Tag | Leave Blank
| Rate limit (MN/s) | Leave Default (unlimited)
| Firewall | `☑`
| IPv4 | `☑  Static`
| IPv4/CIDR |`192.168.1.251/24`|
| Gateway (IPv4) |`192.168.1.5`|
| IPv6 | Leave Blank
| IPv4/CIDR | Leave Blank |
| Gateway (IPv6) | Leave Blank |
| **DNS**
| DNS domain | Leave Default (use host settings)
| DNS servers | Leave Default (use host settings)
| **Confirm**
| Start after Created | `☑`

And Click `Finish` to create your UniFi LXC.

Or if you prefer you can simply use Proxmox CLI `typhoon-01` >  `>_ Shell` and type the following to achieve the same thing (note, you will need to create a password for UniFi LXC):
```
pct create 251 local:vztmpl/centos-7-default_20171212_amd64.tar.xz --arch amd64 --cores 1 --hostname unifi --cpulimit 1 --cpuunits 1024 --memory 1024 --net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.1.5,ip=192.168.1.251/24,type=veth --ostype centos --rootfs typhoon-share:8 --swap 256 --unprivileged 1 --onboot 1 --startup order=1 --password
```

**Note:** test CentOS UniFi package listing is available [HERE](https://community.ui.com/questions/Unofficial-RHEL-CentOS-UniFi-Controller-rpm-packages/a5db143e-e659-4137-af8d-735dfa53e36d).

### 2.02 Install UniFi - CentOS7
First Start your `251 (unifi)` LXC container using the web interface `Datacenter` > `251 (unifi)` > `Start`. Then login into your `251 (unifi)` LXC by going to  `Datacenter` > `251 (unifi)` > `>_ Console and logging in with username `root` and the password you created in the previous step 2.1.

Now using the web interface `Datacenter` > `251 (unifi)` > `>_ Console` run the following command:

```
yum install epel-release -y &&
yum install http://dl.marmotte.net/rpms/redhat/el7/x86_64/unifi-controller-5.8.24-1.el7/unifi-controller-5.8.24-1.el7.x86_64.rpm -y &&
systemctl enable unifi.service &&
systemctl start unifi.service
```

### 2.03 Move the UniFi Controller to your LXC Instance - CentOS7
You can backup the current configuration and move it to a different computer.

Take a backup of the existing controller using the UniFi WebGUI interface and go to `Settings` > `Maintenance` > `Backup` > `Download Backup`. This will create a `xxx.unf` file format to be saved at your selected destination on your PC (i.e Downloads).

Now on your Proxmox UniFi LXC, https://192.168.1.251:8443/ , you must restore the downloaded backup unf file to the new machine by going to `Settings` > `Maintenance` > `Restore` > `Choose File` and selecting the unf file saved on your local PC.

But make sure when you are restoring the backup you Have closed the previous UniFi Controller server and software because you cannot manage the APs by two controller at a time.

## 3.00 NextCloud - Turnkey Debian 9
Nextcloud helps store your files, folders, contacts, photo galleries, calendars and more on a server of your choosing. Access that folder from your mobile device, your desktop, or a web browser. Access your data wherever you are, when you need it.

Here we going to Turnkey Linux NextCloud prebuilt container.

### 3.01 Download the NextCloud LXC template - Turnkey Debian 9

First you need to add Ubuntu 18.04 LXC to your Proxmox templates if you have'nt already done so. Now using the Proxmox web interface Datacenter > typhoon-01 >Local (typhoon-01) > Content > Templates select ubuntu-18.04-standard LXC and click Download.

Or use a Proxmox typhoon-01 CLI >_ Shell and type the following:
```
wget  http://mirror.turnkeylinux.org/turnkeylinux/images/proxmox/debian-9-turnkey-nextcloud_15.2-1_amd64.tar.gz -P /var/lib/vz/template/cache && gz
```

### 3.02 Create the Turnkey Nextcloud LXC - Turnkey Debian 9
Now using the Proxmox web interface `Datacenter` > `Create CT` and fill out the details as shown below (whats not shown below leave as default):

| Create: LXC Container | Value |
| :---  | :---: |
| **General**
| Node | `typhoon-01` |
| CT ID |`121`|
| Hostname |`nextcloud`|
| Unprivileged container | `☑` |
| Resource Pool | Leave Blank
| Password | Enter your pasword
| Password | Enter your pasword
| SSH Public key | Add one if you want to
| **Template**
| Storage | `local` |
| Template |`debian-9-turnkey-nextcloud_15.2-1_amd64.tar.gz`|
| **Root Disk**
| Storage |`typhoon-share`|
| Disk Size |`8 GiB`|
| **CPU**
| Cores |`1`|
| CPU limit | Leave Blank
| CPU Units | `1024`
| **Memory**
| Memory (MiB) |`1024`|
| Swap (MiB) |`256`|
| **Network**
| Name | `eth0`
| Mac Address | `auto`
| Bridge | `vmbr0`
| VLAN Tag | `70`
| Rate limit (MN/s) | Leave Default (unlimited)
| Firewall | `☑`
| IPv4 | `☑  Static`
| IPv4/CIDR |`192.168.70.121/24`|
| Gateway (IPv4) |`192.168.70.5`|
| IPv6 | Leave Blank
| IPv4/CIDR | Leave Blank |
| Gateway (IPv6) | Leave Blank |
| **DNS**
| DNS domain | `192.168.70.5`
| DNS servers | `192.168.70.5`
| **Confirm**
| Start after Created | `☐`

And Click `Finish` to create your Nextcloud LXC. The above will create the Nextcloud LXC without any of the required local Mount Points to the host.

If you prefer you can simply use Proxmox CLI `typhoon-01` > `>_ Shell` and type the following to achieve the same thing PLUS it will automatically add the required Mount Points (note, have your root password ready for Nextcloud LXC):

**Script (A):** Including LXC Mount Points
```
pct create 121 local:vztmpl/debian-9-turnkey-nextcloud_15.2-1_amd64.tar.gz --arch amd64 --cores 1 --hostname nextcloud --cpulimit 1 --cpuunits 1024 --memory 1024 --nameserver 192.168.70.5 --searchdomain 192.168.70.5 --net0 name=eth0,bridge=vmbr0,tag=70,firewall=1,gw=192.168.70.5,ip=192.168.70.121/24,type=veth --ostype debian --rootfs typhoon-share:8 --swap 256 --unprivileged 0 --onboot 1 --startup order=2 --password --mp0 /mnt/pve/cyclone-01-nextcloud,mp=/mnt/nextcloud --mp1 /mnt/pve/cyclone-01-backup,mp=/mnt/backup --mp2 /mnt/pve/cyclone-01-books,mp=/mnt/books --mp3 /mnt/pve/cyclone-01-audio,mp=/mnt/audio 
```

**Script (B):** Excluding LXC Mount Points:
```
pct create 121 local:vztmpl/debian-9-turnkey-nextcloud_15.2-1_amd64.tar.gz --arch amd64 --cores 1 --hostname nextcloud --cpulimit 1 --cpuunits 1024 --memory 1024 --nameserver 192.168.70.5 --searchdomain 192.168.70.5 --net0 name=eth0,bridge=vmbr0,tag=70,firewall=1,gw=192.168.70.5,ip=192.168.70.121/24,type=veth --ostype debian --rootfs typhoon-share:8 --swap 256 --unprivileged 0 --onboot 1 --startup order=2 --password
```

### 3.03 Setup Nextcloud Mount Points - Turnkey Debian 9
If you used Script (B) in Section 3.02 then you have no Moint Points.

Please note your Proxmox Nextcloud LXC MUST BE in the shutdown state before proceeding.

To create the Mount Points use the web interface go to Proxmox CLI Datacenter > typhoon-01 > >_ Shell and type the following:
```
pct set 121 -mp0 /mnt/pve/cyclone-01-nextcloud,mp=/mnt/nextcloud &&
pct set 121 -mp1 /mnt/pve/cyclone-01-backup,mp=/mnt/backup &&
pct set 121 -mp2 /mnt/pve/cyclone-01-books,mp=/mnt/books &&
pct set 121 -mp3 /mnt/pve/cyclone-01-audio,mp=/mnt/audio 
```

### 3.04 Install Nextcloud - Turnkey Debian 9
First start LXC 121 (nextcloud) with the Proxmox web interface go to `typhoon-01` > `121 (nextcloud)` > `START`.

Then with the Proxmox web interface go to `typhoon-01` > `121 (nextcloud)` > `>_ Console` and type your root login and password. The Turnkey installation script will autostart requiring user input. Its best to create and record two different complex passwords (i.e oTL&9qe/9Y&RV style) ready for the Nextcloud installation.

| Nexcloud Passwords | Value | Notes
| :---  | :---: | :---
| MySQL Password | `complex password` | *This password is for your Nextcloud SQL - make a record of it*
| Nextcloud Password | `complex password` | *This password is for your Nextcloud admin account - make a record of it*

Now follow the onscreen prompts and type your input as follows:

| First boot configuration | Value | Notes
| :---  | :---: | :---
| **MySQL Password**
| Enter | `insert MySQL password in the box` | *Add a complex password and record it i.e oTL&9qe/9Y&RV*
| **Nextcloud Password**
| Enter | `insert Nextcloud password in the box` | *Add a complex password and record it i.e oTL&9qe/9Y&RV*
| **Nextcloud Domain**
| Enter the domain to serve Nextcloud | `*` | *Enter a asterix for all. Not required at this stage*
| **Initialize Hub Services**
| Select | `<Skip>` | *Not required*
| **System Notifications and Critical Security Alerts**
| Select | `<Skip>` | *Not required at this stage*
| **Security Updates**
| Security updates? | `<Install>` |

Your Nextcloud console will commence downloading security updates and complete the Nextcloud installation. On completion a dialoque box will appear listing your Nextcloud appliance services. Click/Select `<Quit>`



---
nano /var/www/nextcloud/config/config.php
sudo nextcloud.occ config:system:set trusted_domains 1 --value=192.168.1.*

<?php
$CONFIG = array (
  'passwordsalt' => '56e122543f3aa183ea12583e3b88aa0d',
  'secret' => '6ca4a03b07ac120fe34d4ed586d9b4e5b543e1b62eaa40757fcbfaba3c03bf21791fb3bff94b08299aa4eb407743d7d6',
  'trusted_domains' => 
  array (
    0 => 'localhost',
    1 => '*',
  ),
  'datadirectory' => '/var/www/nextcloud/data',
  'dbtype' => 'mysql',
  'version' => '16.0.1.1',
  'overwrite.cli.url' => 'http://localhost',
  'dbname' => 'nextcloud',
  'dbhost' => 'localhost',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'nextcloud',
  'dbpassword' => '2a66e17d8497543144154d325772bdc3',
  'installed' => true,
  'memcache.local' => '\\OC\\Memcache\\Redis',
  'redis' => 
  array (
    'host' => '/var/run/redis/redis.sock',
    'port' => 0,
    'timeout' => 0.0,
  ),
  'filelocking.enabled' => true,
  'memcache.locking' => '\\OC\\Memcache\\Redis',
  'instanceid' => '4c69191b6cbfa',
  'updater.secret' => '$2y$10$w8o1iEVTxqSuOvliGBuTjOCtR5FGq8zH1qN9Xn8u7W/mE9COrKrCC',
  'maintenance' => false,
);
