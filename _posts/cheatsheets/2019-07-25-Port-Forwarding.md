---
published: false
layout: post
categories: cheatsheets
---
# Summary
{:.no_toc}

* toc
{:toc}

# SSH PortForwarding

## Configure Simple SSH Port-Forwarding

Run these commands on the server that will be performing the port forwarding.

### 1. Enable IP Forwarding

	sudo sysctl net.ipv4.ip_forward=1
    
### 2. Forward traffic on port 1111 to IP 1.1.1.1 on port 22

	sudo iptables -t nat -A PREROUTING -p tcp --dport 1111 -j DNAT --to-destination 1.1.1.1:22
    
dport = incoming port that will forward the traffic

to-destination = server IP address and port that you are forwarding to

### 3. Ask iptables to Masquerade

	 sudo iptables -t nat -A POSTROUTING -j MASQUERADE
     
### 4. Test

From the client PC, SSH to the server that is doing the port forwarding.  If the server doing the port forwarding is 2.2.2.2, then ssh to 2.2.2.2:1111.  You should be connected to 1.1.1.1 via SSH (port 22).

### 5. Save iptables rules

	sudo sh -c "iptables-save > /etc/iptables.rules"
    
### 6. Automatically apply iptables rules at startup

a. Edit the interface the rules apply to by editing /etc/network/interfaces

At the end of the network related lines for that interface, add the line:

	pre-up iptables-restore < /etc/iptables.rules
    
b. Edit /etc/sysctl.conf by adding the line net.ipv4.ip_forward = 1.

If you want to keep information from byte and packet counters, use the command:
	
    sudo sh -c "iptables-save -c > /etc/iptables.rules"
    
## List PREROUTING Rules

	sudo iptables -t nat {--line-numbers} -L
    
## Delete NAT Rule

	sudo iptables -t nat -D PREROUTING [line #]
    
Note: The option "-t nat" are not needed when you want to delete POSTROUTING, INPUT, or OUTPUT rules.