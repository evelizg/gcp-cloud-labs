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

## Solution 

I've used a Makefile to deploy this solution. 

This Makefile has eight operations:

1) `make baseline` - Create project baseline
2) `make networking` - Create the VPC for the project
3) `make accounts` - Create the IAM role and service account(s) and bind them.
4) `make compute-create` - Create the instance template(s), instance group(s) and define the autoscaling.
5) `make compute-stop` - Turn off the autoscaling on existing instance group(s)
6) `make security` - Create the firewalls needed for the solution.
7) `make all` - Create the entire solution, incorporating items 1 to 6 above.
8) `make delete` - Delete the entire project without prompting the user to confirm.

### Pre-requisites

Edit the `Makefile` with the appropriate variables for your environment. Most notably, you will need to retrieve your billing id by running the following command from inside the gcloud shell:

```gcloud
gcloud alpha billing accounts list
```

The Makefile has been commented as much as possible, so that you can interpret what is being executed and why.
