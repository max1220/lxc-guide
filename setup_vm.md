$VMIP = Desired IP of VM
$ROOTFS = Inside the rootfs of the VM
  To access files in your rootfs from your unpriviledged user, you need a uidmap'ed shell.
  Read: https://s3hh.wordpress.com/2013/03/07/experimenting-with-user-namespaces/
$VMNAME = Name of VM
$GATEWAY = Gateway IP(192.168.100.1)
$NAMESERVERS = Nameservers, seperated by " "
  You can get this using: "echo $(cat /etc/resolv.conf | grep "nameserver" | tr "\n" " ")"
$NAMESERVER = Single nameserver
  Just take one of the command from above; this is overwritten automaticly later

New VM
 * "lxc-create -t download -n $VMNAME"
 * Don't start container yet!
 * config:
   (...)
     "lxc.network.flags = up"
     "lxc.network.ipv4 = $VMIP/24"
     "lxc.network.ipv4.gateway = $GATEWAY"
     "lxc.start.auto = 1"
 * $ROOTFS/etc/network/interfaces:
     "auto lo"
     "iface lo inet loopback"
     ""
     "auto eth0"
     "iface eth0 inet static"
     "  address $VMIP"
     "  netmask 255.255.255.0"
     "  gateway $GATEWAY"
     "  dns-nameservers $NAMESERVERS"
 * $ROOTFS/etc/apt/sources.list:
     "deb http://ftp.de.debian.org/debian jessie main contrib non-free"
     "deb http://security.debian.org/ jessie/updates main contrib non-free"
 * Start the container:
     "lxc-start -n $VMNAME -d"
 * Access using:
     "lxc-attach -n $VMNAME"
 * Temporarily setup nameserver:
     "echo nameserver 8.8.8.8 > /etc/resolv.conf"
 * Install basic packages:
     "apt-get update"
     "apt-get install -y nano iputils-ping bash-completion screen openssh-server resolvconf"
     "dpkg-reconfigure locales"
     (Select whatever locales you want)
 * Configure sshd: "nano /etc/ssh/sshd_config"
     (At the verry bottom:)
     "UsePAM no"
     (PAM seems to be non-working in LXC-Containers)
 * Import SSH key(Root login only allowed using key by default):
     "mkdir -p /root/.ssh/"
     "chmod 700 /root/.ssh"
     "nano /root/.ssh/authorized_keys"
     (Paste your public key(s) in here)
     "chmod 600 /root/.ssh/authorized_keys"
     (Using ~ instead of /root won't work, since lxc-attach doesn't set the home path correctly)
 * Unlock root account or set password:
     "usermod --unlock root"
     or
     "passwd"
 * Exit container:
     "exit"
 * Restart VM:
     "lxc-stop -n $VMNAME && lxc-start -n $VMNAME -d"
     (Might take a while for some reason)
 * Add port forward(As root):
     "iptables -t nat -A PREROUTING -p tcp -i eth0 --dport $EXT_PORT -j DNAT --to-destination $VMIP:$INT_PORT"
     Where $EXT_PORT is the external port and $INT_PORT is the port on the VM to forward it to
     Make permanent:
     "nano /etc/network/if-pre-up.d/lxc-$VMNAME"
       "#!/bin/bash"
       (Copy the line(s) from above here)
       "exit 0"
     "chmod +x /etc/network/if-pre-up.d/lxc-$VMNAME"


(Tested on a Ubuntu 14.04 as host and an Debian Jessie guest on amd64 8.11.2015)
