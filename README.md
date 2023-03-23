# Easy OpenVPN

OpenVPN is a great free VPN service. It is very easy to set up for a one-to-one
client–server setup by using symmetric keys. However, if you need several
clients, the steps needed complicates the setup quite a bit.

easyrsa is a very helpful script that makes creating certificates and keys a
lot easier. However, for a beginner, even easyrsa may be too complicated. This
project aims to help you set up a server and several clients in a matter of
seconds, securel

Install the packages openvpn and easyrsa before starting.

## Overview

The OpenVPN server needs to be accessible from a public IP address. This IP
address (or its fully qualified domain name) is set in the variable
*SERVERNAME*, and it is used by clients to connect to the server. If the
computer running the server does not have a public IP address, you must ensure
to forward your chosen port (*PORT*) to the computer. You may need to whitelist
traffic to this port (UDP) as well.

If you intend to use VPN to route traffic to an internal network, and that
network cannot host the VPN server, you may use what this project refers to as
an "uplink client". This client is special in that the server routes traffic
from the other clients to the desired networks on that client. For instance,
you are running your server in a cloud, but you want to let all clients reach
devices on 172.16.17.0/24, behind the "uplink client". In order to achieve
this, the server needs "iroute" directives that sends this traffic to the
"uplink client". In addition, all clients need to have "route" directives, and
the "uplink" client needs a static IP address. The scripts in this project
makes this easy to set up. If you don't need this feature, simply don't run
*vpn-add-uplink-client*.

You can have several such "uplink clients" in OpenVPN, providing access to
several different networks, but this is too complicated for this relatively
simple project. Modify route, iroute, ifconfig-push and ifconfig-pool
accordingly.

In order to access networks through VPN, you may need to enable IP forwarding
on your computer and create NAT rules.

This project uses an IP address pool to assign clients addresses. The addresses
are therefore not static. You may make assignments persistent by adding the
configuration directive "ifconfig-pool-persist" and modify the contents of the
file. See the man page for more details. The "uplink client" has a static IP
address that is not included in the IP address pool.

easy-openvpn consists of four files:

### vpn-vars

This file consists of variables used by the three included scripts and it
**must** be modified. At the very least you have to replace *HOSTNAME* so that
your clients have a valid IP address or hostname to connect to. You also want
to change the port and ensure that it is whitelisted and/or forwarded in your
firewall. Lastly, you may want to edit the default subnet. If you want to route
certain traffic through your server, uncomment and modify *ROUTES* as well.

Here is a list of variables of interest:
| Variable | Description |
| --- | --- |
| SERVERNAME | Used in certificate, directory and configuration file names |
| HOSTNAME | Public IP address or FQDN (fully qualified domain name) that the clients will use to reach the OpenVPN server (**must be changed**) |
| PORT | UDP port that the clients will connect to. Default: 21021 |
| SUBNET | Internal VPN network. Default: 10.0.10.0 |
| NETMASK | Netmask determining the size of the internal VPN network (this decides how many clients that can be online simultaneously). Default: 255.255.255.0 |
| UPLINKCLIENTIP | Static IP address of the "uplink client". Default: 10.0.10.2. If no "uplink client" is used, this variable may be ignored. |
| IPPOOLSTART | First IP address to be used to assign clients. Default: 10.0.10.3. This can be set to 10.0.10.2 if no "uplink client is used". |
| ROUTES | Networks that should be routed to the server (which in turn may route the traffic to the "uplink client", if any. Each entry in the bash array must be a quoted "subnet netmask". |

### vpn-setup

After modifing vpn-vars, run this script to set up the server. Only run the
script once. If it fails, it's best to clear the files in /etc/openvpn/server/
before running it again.

If the script succeeds, the OpenVPN server will run and await connections. You
can check whether OpenVPN runs by running `systemctl status
openvpn-server@SERVERNAME` (replace SERVERNAME).

### vpn-add-uplink-client

Run this script to create an "uplink client", if this feature is needed.

### vpn-add-client

Create a VPN client configuration file, embedded with all necessary certificate
and key files. This file can easily be imported in Linux Network Manager by running

1. `nmcli connection import type openvpn file servername-clientname.conf`
1. `nmcli connection modify servername-clientname ipv4.never-default yes`

If you skip the last step, you'll route all traffic through the VPN connection.
This may not be what you want. You'll need the packages network-manager-openvpn
(and network-manager-openvpn-gnome for the GUI).

The same file can be used on Windows and MacOS, as long as you remove the
resolvconf lines starting with "up" and "down".

---
¹: Effort has been made to create a secure OpenVPN configuration, but you
shouldn't use this outside of a lab environment until you've studied the
generated configuration files and evaluated the settings used.
