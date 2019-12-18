# Desired Result - Setup

Below is the desired result setup for Exercise 2:

1) Two-tier setup: Frontend and Backend, each autoscaled across 2+ zones
2) Use ICMP (ping) to represent allowed traffic
	- Real-world usage would use other protocols/ports -  like TCP/80
3) Frontend:
	- Accepts incoming from Internet
	- Can connect outbound to backend and Internet
4) Backend:
	- Only accepts incoming from Frontend and other Backend
	- No outbound anywhere (except other Backend)
5) All firewall rules based on service accounts (except _open-ssh-tag_)


# Desired Result - Validation

1) From Cloud Shell or your computer:
	- Can ping Frontend instances
	- _Cannot_ ping Backend instances
	
2) When SSHed to a Frontend instance:
	- Can ping Backend instances (including cross-zone)
	- Can ping google.com
	
3) When SSHed to a Backend instance:
	- _Cannot_ ping Frontend instances
	- _Cannot_ ping google.com
	- _Can_ ping other Backend instances
	

# Tips

1) Clean up after yourself
	- Disable autoscaling and set "Number of instances" to zero.
