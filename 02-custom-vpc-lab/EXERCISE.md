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

TODO: Document below:

all: baseline networking accounts compute-create security

baseline: create-project set-project link-billing enable-compute 

networking: baseline create-vpc

accounts: create-role create-fe-sa bind-fe-sa create-be-sa bind-be-sa

compute-create: create-fe-it create-fe-ig set-fe-ig create-be-it create-be-ig set-be-ig

compute-stop: stop-fe-ig stop-be-ig

security: create-fw-rules

delete: unset-project delete-project

### Pre-requisites

Edit the `Makefile` with the appropriate variables for your environment. Most notably, you will need to retrieve your billing id by running the following command from inside the gcloud shell:

```gcloud
gcloud alpha billing accounts list
```

The Makefile has been commented as much as possible, so that you can interpret what is being executed and why.
