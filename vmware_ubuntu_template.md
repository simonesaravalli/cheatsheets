# How to create an Ubuntu Server template for VMware virtual machines

## Create and configure a new Ubuntu Server virtual machine

* Create a new virtual machine in VMware with 16 GB of HDD and 2 GB of RAM

* Install Ubuntu Server (for example 22.04) LTS, set IP via DHCP, create first Linux user and password

* Once the installation is completed, restart the virtual machine, then connect to it via terminal or SSH and install all available OS updates

```
sudo apt-get update
sudo apt-get upgrade
```

* Enable unattended upgrades to automatically install security updates

```
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

* Create a new Linux user that will be used by Ansible, from a control host, to connect to virtual machines created from the template, to automate the installation of OS updates. Add the new user to the sudo group

```
sudo adduser ansible
sudo usermod -aG sudo ansible
```

* Copy the necessary SSH public keys that will be used to connect to the new virtual machines created from the template without providing a password every time.

**IMPORTANT** if public/private keys have never been created before, create them now. Login on a control host and run

```
ssh-keygen -b 4096 -C "<linux_user>"
```

When prompted to provide a location for the SSH public and private keys, type something like /home/<user>/.ssh/<linux_user>_id_rsa

```
ssh-keygen -b 4096 -C "<linux_user>"
```

When prompted to provide a location for the SSH public and private keys, type something like /home/<user>/.ssh/ansible_id_rsa

Now it's time to copy the new keys on the virtual machine that will become the new Ubuntu Server template:

```
ssh-copy-id -i /home/<user>/.ssh/<linux_user>_id_rsa <linux_user>@<ip of template vm>
ssh-copy-id -i /home/<user>/.ssh/ansible_id_rsa ansible@<ip of template vm>
```

Once done, it's possible to check if SSH connection to the target virtual machine works with both users:

```
ssh -i /home/<user>/.ssh/<linux_user>_id_rsa <linux_user>@<ip of template vm>
ssh -i /home/<user>/.ssh/ansible_id_rsa ansible@<ip of template vm>
```

* Configure SSH server by enforcing security. An example of /etc/ssh/sshd_config is reported below:

```
# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
PermitRootLogin no
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
KbdInteractiveAuthentication no

# Kerberos options
KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the KbdInteractiveAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via KbdInteractiveAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and KbdInteractiveAuthentication to 'no'.
UsePAM no

#AllowAgentForwarding yes
AllowTcpForwarding no
#GatewayPorts no
X11Forwarding no
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /run/sshd.pid
#MaxStartups 10:30:100
PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
Banner none

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem sftp  /usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server
#PasswordAuthentication no

AllowUsers <linux_user> ansible
```

* Configure ufw firewall by enabling all outgoing traffic, denying all incoming traffic and allowing only SSH incoming traffic

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw enable
```

At this point the new virtual machine should be completed, so it can be prepared to be converted in a template for VMware.

## Prepare the virtual machine to become a template for VMware

* Remove all SSH configuration files related to the host itself

```
cd /etc/ssh
sudo rm -f ssh_hosts_*
``` 

**IMPORTANT** These files must be regenerated once a new virtual machine is created from the template, otherwise SSH will not run. See below for more details on that.

* Reset the machine id. This step is really important in order to avoid multiple virtual machines cloned from the same template to have the same ID. This infact can cause issues at the networking level with DHCP reservations, etc

```
sudo truncate -s 0 /etc/machine-id
```

Check if a symbolic link in /var/lib/dbus to /etc/machine-id exists, otherwise create it

```
sudo ln -s /etc/machine-id /var/lib/dbus/machine-id
```

* Clean apt cache

```
sudo apt-get clean
```

* Remove uneeded deb packages

```
sudo apt autoremove
```

* Poweroff the virtual machine, then remove any iso file eventyally connected

* Create the template from the virtual machine for VMware vSphere console

## Create and configure a new virtual machine from the template

* From VMware vSphere console create a new virtual machine using the Ubuntu template just created and start it
* When the virtual machine is up, login using VMware console as <linux_user>
* Change the hostname

```
sudo nano /etc/hostname
sudo nano /etc/hosts
```

* If the virtual machine requires a static IP address instead of one given by DHCP server, adjust the configuration accordingly by editing the file /etc/netplan/00-installer-config.yaml. The following is a configuration example:

```
sudo vi 00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  renderer: networkd
  ethernets:
    ens33:
      addresses:
        - 192.168.1.247/24
      nameservers:
        addresses: [4.2.2.2, 8.8.8.8]
      routes:
        - to: default
          via: 192.168.1.1
  version: 2
  ```

In the above file we have used following:

  - ens33 is the interface name
  - addresses are used to set the static ip
  - nameservers used to specify the DNS server ips
  - routes used to specify the default gateway

To make above changes into the effect the apply these changes using following netplan command:

```
sudo netplan apply
```

To view the IP address just set, run this command:

```
ip addr show ens33
```

To view the default route:

```
ip route show
```

* Reconfigure SSH server and restart it:

```
sudo dpkg-reconfigure openssh-server
sudo systemctl restart ssh
```

It's possible now to login to the new virtual machine via SSH from a remote host with the proper SSH private key installed for <linux_user> and even run Ansible playbooks from an Ansible control node that has the ansible linux user's SSH private key installed to automate the installation of additional software, install OS updates and so on.