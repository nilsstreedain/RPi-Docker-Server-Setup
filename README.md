# RPi-Docker-Server-Setup

Before getting started:
- Create a fresh install of Raspberry Pi OS with ssh enabled (add an empty file named ssh to the boot folder)
- Connect the Raspberry Pi to your network (make sure to use a trunk port and assign a static IP)
- ssh into the Pi

## Prepare Raspberry Pi

Change default user password:

```bash
passwd
```

Update Raspberry Pi

```bash
sudo apt update
```

```bash
sudo apt full-upgrade
```

Open raspberry pi configuration

```bash
sudo raspi-config
```

Change Raspberry Pi Hostname (raspi-config > System Options > Hostname)

Set Raspberry Pi Country (raspi-config > Localisation Options > WLAN Country)

Finish > Would you like to reboot now? > Yes

## Setup non-default user

### Enable root SSH (If headless)

Enable root password
```bash
sudo passwd root
```

Edit SSH congif:
```bash
sudo nano /etc/ssh/sshd_config
```

Edit "#PermitRootLogin prohibit-password" to:
```
PermitRootLogin yes
```

Restart SSH:
```bash
sudo systemctl restart sshd
```

### Rename pi user

Login to root account
```bash
ssh root@raspberrypi
```

Change pi username (Replace nilsstreedain with the username you'd like to use):
```bash
usermod -l nilsstreedain pi
```

Change home directory name (Replace nilsstreedain with the username you'd like to use):
```bash
usermod -m -d /home/nilsstreedain nilsstreedain
```

Logout of root account
```bash
logout
```

## SSH Authentication With Public/Private Key Pair Instead of Password

### On Raspberry pi:

Create public key directory:
```bash
mkdir ~/.ssh && chmod 700 ~/.ssh
```

### On Mac:

Generate Public/Private Key Pair:
```bash
ssh-keygen -b 4096
```

Upload Public key from Mac to Linux (Replace nilsstreedain with the username you'd like to use):
```bash
scp ~/.ssh/id_rsa.pub nilsstreedain@raspberrypi:~/.ssh/authorized_keys
```

### Disable root password

Diable root password:
```bash
sudo passwd -l root
```

Just to be nice, also disable ssh for root. Edit SSH congif:
```bash
sudo nano /etc/ssh/sshd_config
```

Edit "PermitRootLogin yes" to:
```
#PermitRootLogin prohibit-password
```

Also disable ssh authentication with password. Change "#PasswordAuthentication yes" to:
```
PasswordAuthentication no
```

Restart SSH:
```bash
sudo systemctl restart sshd
```

## Setup Auto-Updtaes

Install Unattanded Upgrades:
```bash
sudo apt-get install unattended-upgrades
```

Start Unattended Upgrades:
```bash
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

## Setup Firewall

Install Uncomplicated Firewall:
```bash
sudo apt install ufw
```

Allow TCP on port 22 for SSH:
```bash
sudo ufw allow 22/tcp
```

Enable Firewall:
```bash
sudo ufw enable
```

## Setup VLANs:

Install vlan package:
```bash
sudo apt install vlan
```

Create network interface config file for vlans:
```bash
sudo nano /etc/network/interfaces.d/vlans
```

Configure vlan interfaces by adding:
```
auto eth0.16
iface eth0.16 inet manual
  vlan-raw-device eth0
 ```

Restart pi's networking:
```bash
sudo systemctl restart networking
```

Test config for an IP in each vlan:
```bash
hostname -I
```

## Setup Docker

Install dependencies:
```bash
sudo apt-get install curl git
```

Install Docker:
```bash
bash -c "$(curl -fsSL https://get.docker.com)"
```

Test Docker:
```bash
sudo docker run --rm hello-world
```

Install Docker Compose:
```bash
sudo apt-get -y install docker-compose
```

## Setup Pi-Hole

Create a directory to setup Pi-Hole with Auto-Updating Blocklists:
```bash
mkdir pihole pihole/etc-pihole-updatelists && cd pihole
```

Copy the pihole-updatelists config file to configure pihole-updatelists:
```bash
sudo wget https://raw.githubusercontent.com/nilsstreedain/RPi-Docker-Server-Setup/main/pihole/pihole-updatelists/pihole-updatelists.conf -O etc-pihole-updatelists/pihole-updatelists.conf
```

Copy the docker-compose file to configure cloudflared, pi-hole, pihole-updatelists, and their respective networking:
```bash
sudo wget https://raw.githubusercontent.com/nilsstreedain/RPi-Docker-Server-Setup/main/pihole/docker-compose.yml -O docker-compose.yml
```

Run docker-compose:
```bash
sudo docker-compose up -d
```

Set Pi-Hole password:
```bash
sudo docker exec -it pihole sudo pihole -a -p
```

### Updating Pi-Hole and other Docker containers
When you need to update Pi-Hole, ssh into the raspberry pi and navigate to ~/pihole :
```bash
cd ~/pihole
```

Pull the latest Pi-Hole docker updates:

```bash
sudo docker pull jacklul/pihole
```
<!--
```bash
sudo docker pull pihole/pihole && sudo docker pull jacklul/pihole
```
-->

Then re-run docker-compose to build and run the new updated containers:
```bash
sudo docker-compose up -d --force-recreate --build
```
