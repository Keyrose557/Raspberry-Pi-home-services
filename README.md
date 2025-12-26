# PiQ Setup

I want to deploy a raspberry Pi to provide some services on my home network. Maybe in the future Ill add more services. For now PiHole and PiVPN will do.

## Preparation

Im using **Windows 11** and the **Terminal** application to do this project and have already <u> reserved a static IP </u> on my router.

### SSH Key Gen

- Generate your ssh key pair on your **host** device

```bash
ssh-keygen
```

You'll be asked to create a password for your private SSH key.

- Use the **`cat`** command to copy and paste your public key.

```bash
# file path on windows machine

cd C:\Users\*username*\.ssh

# dir will list the files in the .ssh directory

dir

# you should see two key pairs, use the CAT on the public key indicated by the ".pub" file

cat c:\Users\*username*\.ssh\id_ed25519.pub

# Copy and paste what is displayed, you'll need this moving forward

```

### Raspberry Pi Imager

The OS being used is Debian 12 Bookworm, you can find this in other **Raspberry Pi OS (other)**. Im selecting this OS for the foundation of our server to reduce compatability issues moving forward.

![OS](/media/bookworm.png)

### OS Customisation settings

Im giving this raspberry pi hostname **piQ.local**

![hostname](/media/hostname.png)

Using a more secure method than password authentication is good practice, so lets try it.

- Enable SSH option **public SSH key only**

![ssh](/media/OS%20customization.png)

### SSH into Pi

```bash
ssh *pi
```

![ssh](/media/ssh.png)


### Update the PI

```bash
# You'll be prompted if you want to continue with the updates.

sudo apt update && sudo apt full-upgrade

```

## Our Pie is ready for services

### Pi-hole


Lets start with Pi-hole for network wide ad blocking. Review the documentation from the git repo.

![pihole](/media/Vortex-R.png)


https://github.com/pi-hole/pi-hole/blob/master/README.md

```bash

sudo curl -sSL https://install.pi-hole.net | bash

```

I leaving the deafult options on, except for the upstream DNS. I selected quad9 filtered ECS, DNSSEC. I selected this one because its GRC compliant.

The installation GUI will give you the credentials and IP address to admin webpage.

### Adding block lists

Ill be adding these blocking list

 https://github.com/hagezi/dns-blocklists?tab=readme-ov-file#pro

Just copy and paste.

![blocklist](/media/dnslist.gif)

- tools>update gravity> select update and your finished

## PiVPN

### PiVPN install

https://github.com/pivpn/pivpn

```bash 
sudo curl -L https://install.pivpn.io | bash

```

I wont be configuring ip6 so i selected **no**

I have dchp ip address reserved so I select **yes**

user: `keyrose`

vpn service `wiregaurd`

port `51820`

- Youll need to create a port **fowarding rule** on your route for this port

pihole `yes`

public IP `yes`

unattended security upgrades `yes`

Following the installtion , will lead to a reboot

### PiVPN setup (mobile)

```bash
# create clients
pivpn add

# 1. select IP from ranges provided, this is user preference

# 2. I left name as default

# 3. Generate QR code to add devices

pivpn -qr

# 4 download app wireguard to enable VPN access for mobile devices 

# help command

pivpn help
```
below is the documenation 
https://docs.pivpn.io/

### PiVPN setup (desktop/laptop)

Adding devices require the `.conf` file made by PiVPN in order to work.

1. Install wireguard app

    - https://www.wireguard.com/install/

2. Copy and paste the `.conf` into the app

    I used `scp`command to copy the file from the piQ to the windows machine. However it required the installation of OpenSSH Server. I used the command below.
    
    - Windows Machine 

    ```powershell
    Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
    Start-Service sshd
    Set-Service -Name sshd -StartupType Automatic
    netsh advfirewall firewall add rule name="OpenSSH Server" dir=in action=allow protocol=TCP localport=22
    ```
    - Raspberry Pi

    ```bash
    scp /home/keyrose/configs/piQ.conf keyrose@192.168.*.*:/Users/keyrose
    ```
    The windows machine should now have a copy of the `.conf` to add a tunnel using wiregaurds gui

