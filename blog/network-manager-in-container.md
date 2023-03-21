---
title: Using network manager in an Ubuntu lxd container.
---
For one reason or another you may want to use NetworkManager inside a
container.  An lxd container of recent vintage Ubuntu release (18.04)
will have cloud-init render networking through systemd-networkd.

I needed to play around with a vpn that expected NetworkManager to be
running.  Here is a simple recipe for setting up network manager to
manage network in an LXD container.

 * create the container and then wait a bit for it to boot.

       $ lxc launch ubuntu-daily:18.04 nm-test
       $ sleep 10

 * disable cloud-init, remove cloud-init authored netplan config and install network manager.

       $ lxc exec nm-test -- touch /etc/cloud/cloud-init.disabled
       $ lxc exec nm-test -- rm -f /etc/netplan/50-cloud-init.yaml
       $ lxc exec nm-test -- apt-get --quiet update
       $ lxc exec nm-test -- apt-get install --no-install-recommends --assume-yes network-manager

 * Enable network manager to manage virtual ethernet devices \[[bug 1676547](https://bugs.launchpad.net/bugs/1676547)\] by creating an empty file to mask the distro-included version.

       $ lxc exec $name -- sh -c ': > /etc/NetworkManager/conf.d/10-globally-managed-devices.conf'

 * Restart network manager

       $ lxc exec nm-test -- systemctl restart network-manager


Thats it.  You should have functional `nmcli` now.

    root@nm-test:~# nmcli
    eth0: connected to Wired connection 1
            "eth0"
            ethernet (veth), 00:16:3E:63:BC:31, sw, mtu 1500
            ip4 default, ip6 default
            inet4 10.157.246.65/24
            route4 0.0.0.0/0
            route4 10.157.246.0/24
            inet6 fd42:54b0:2d39:df1:5394:5920:e551:94d9/64
            inet6 fe80::d670:4eee:34ec:2dd9/64
            route6 fd42:54b0:2d39:df1::/64
            route6 ::/0
            route6 ff00::/8
            route6 fe80::/64
            route6 fe80::/64

    lo: unmanaged
            "lo"
            loopback (unknown), 00:00:00:00:00:00, sw, mtu 65536

    DNS configuration:
            servers: 10.157.246.1
            domains: lxd
            interface: eth0

            servers: fe80::fced:6bff:fe63:9e55 fd42:54b0:2d39:df1::1
            interface: eth0


It may be useful for you to rename the automatically generated 'Wired Connection 1' to something more friendly.

    $ lxc exec nm-test -- nmcli conn modify 'Wired connection 1' connection.id eth0

Lastly, you may not get the a dhcp request by default.  I'm not sure why, but I have seen that happen.  To tell network-manager that you want one:

    $ lxc exec nm-test -- nmcli conn modify eth0 ipv4.method auto
