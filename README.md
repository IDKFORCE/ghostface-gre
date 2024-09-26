## Read Carefully


## This setup creates a GRE (Generic Routing Encapsulation) tunnel between two servers, which allows for encapsulating one network protocol within another. In this case, it encapsulates the network traffic between two servers (Server 1 and Server 2) over the internet ## using their external IPs.

Overview of what it does:
- GRE Tunnel Creation:
## A GRE tunnel is a point-to-point connection that enables encapsulating a wide variety of network layer protocols inside virtual point-to-point links.
## It is used to connect two separate networks securely or to forward traffic that normally wouldn’t be routable across the internet.
- Method 1: Using nmcli (NetworkManager)
## nmcli is a NetworkManager CLI tool to manage networking configurations on systems like RHEL or Fedora.
## You create a GRE interface named gre1 on both servers and configure it to use:
## The external IPs of both servers (Server 1: 1.1.1.1, Server 2: 2.2.2.2) for the tunnel.
## The internal IPs for the GRE tunnel (Server 1: 10.10.1.1, Server 2: 10.10.2.1).
## A /24 network (255.255.255.0 subnet mask) to route traffic between the two subnets (10.10.1.0/24 and 10.10.2.0/24).
- Firewall configuration (iptables):
## Allows GRE traffic between the two external IPs by accepting any GRE packets from either server's IP.
## MSS clamping ensures that packet fragmentation issues are minimized when traffic crosses the GRE tunnel, preventing MTU (Maximum Transmission Unit) mismatches.
## Method 2: Manual GRE Setup (Using Shell Scripts)
## This method provides a manual way of configuring the same GRE tunnel using Linux ip commands, particularly useful for systems that don’t use NetworkManager (like Ubuntu, or in environments where manual control is preferred).
- The scripts use variables to configure:
## GRE interface (gre1).
- Local and remote IPs for each server.
- Internal IPs and routes to connect subnets across the tunnel.
- Functions:
## up(): Adds the GRE interface, brings it up, assigns an IP address, and sets up routing for the subnet.
## down(): Takes down the GRE interface and deletes it.
## The case statement at the end allows the script to handle commands like ./script.sh up to start the tunnel or ./script.sh down to bring it down.
- Key Points of Feedback:

- Tunnel Creation:

## This creates a point-to-point virtual connection between two servers over the internet. The GRE tunnel encapsulates IP traffic inside IP, making it look like the two servers are on the same local network.

- Routing and Networking:

## Traffic from Server 1’s internal GRE interface (10.10.1.1) destined for 10.10.2.0/24 is routed through the GRE tunnel. Similarly, traffic from Server 2 (10.10.2.1) destined for 10.10.1.0/24 is routed through the tunnel.
## This allows for transparent communication across these subnets without requiring changes to how applications or services communicate internally.

- Firewall:

## The use of iptables ensures that only GRE traffic between these two specific external IP addresses is accepted. This helps protect against unwanted GRE traffic.
## The TCP MSS clamping is an important step to prevent MTU mismatches, which can cause issues with packet fragmentation across the tunnel.

- Cross-Compatibility:

## Method 1 is ideal for RHEL, Fedora, or other systems with nmcli and NetworkManager, making it easier to configure networking with a single command.
## Method 2 works for systems where NetworkManager is not available or preferred, such as on older Ubuntu systems or other Linux distributions, allowing manual control over the GRE tunnel.
- In summary:
## You are setting up a virtual network between Server 1 and Server 2 using GRE tunnels, ensuring they can securely communicate with each other as if they are on the same local network.
## Method 1 simplifies the configuration with nmcli on RHEL/Fedora.
## Method 2 uses shell scripts to configure the GRE tunnel manually for more control or on other Linux distributions.

## Method 1: Using nmcli (RHEL, Fedora)

- Server 1 Configuration:
```js
nmcli conn add type ip-tunnel ifname gre1 mode gre remote 2.2.2.2 local 1.1.1.1 \
-- ip-tunnel.mtu 1500 ip-tunnel.ttl 255 ipv4.method manual ipv4.addresses 10.10.1.1 \
ipv4.routes "10.10.2.0/24"
```

- Server 2 Configuration:

```js
nmcli conn add type ip-tunnel ifname gre1 mode gre remote 1.1.1.1 local 2.2.2.2 \
-- ip-tunnel.mtu 1500 ip-tunnel.ttl 255 ipv4.method manual ipv4.addresses 10.10.2.1 \
ipv4.routes "10.10.1.0/24"
```

- Firewall Configuration:

```js
iptables -A INPUT -p gre -s 2.2.2.2 -j ACCEPT
iptables -A INPUT -p gre -s 1.1.1.1 -j ACCEPT
iptables -A FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
```

## Method 2: Manual GRE Setup (for Ubuntu and other systems)

- Server 1 Script:
```js
#!/bin/bash

DEV=gre1
LOCAL=1.1.1.1
REMOTE=2.2.2.2
IP=10.10.1.1
NET=10.10.2.0

up(){
 ip tunnel add $DEV mode gre remote $REMOTE local $LOCAL ttl 255
 ip link set $DEV up
 ip addr add $IP dev $DEV
 ip route add $NET/24 dev $DEV
}

down(){
 ip link set $DEV down
 ip tunnel del $DEV
}

case "$1" in
up)
  up
  ;;
down)
  down
  ;;
*)
  echo "Usage: $0 [up|down]"
  ;;
esac
```

- Server 2 Script:
```js
#!/bin/bash

DEV=gre1
LOCAL=2.2.2.2
REMOTE=1.1.1.1
IP=10.10.2.1
NET=10.10.1.0

up(){
 ip tunnel add $DEV mode gre remote $REMOTE local $LOCAL ttl 255
 ip link set $DEV up
 ip addr add $IP dev $DEV
 ip route add $NET/24 dev $DEV
}

down(){
 ip link set $DEV down
 ip tunnel del $DEV
}

case "$1" in
up)
  up
  ;;
down)
  down
  ;;
*)
  echo "Usage: $0 [up|down]"
  ;;
esac
```
