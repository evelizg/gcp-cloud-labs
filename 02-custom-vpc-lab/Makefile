# Populate with your variables
PROJECT_ID := gcp-challenge-net-lab-22
# To retrieve your billing account id, run the command `gcloud alpha billing accounts list` from cloud shell
ACCOUNT_ID := 0175C8-6FE4A3-D0CC88
DEFAULT_REGION := us-west1
DEFAULT_ZONE := us-west1-b
CUSTOM_VPC_NAME := custom-vpc-lab-cc1
FRONTEND_NAME:= frontend-subnet-1
FRONTEND_SUBNET := 192.168.16.0/24
BACKEND_NAME:= backend-subnet-1
BACKEND_SUBNET := 192.168.32.0/24
LAB_LABELS = gcp-custom-vpc
IAM_ROLE_NAME = dt_custom_role_lab2
IAM_ROLE_TITLE = dtCustomRoleLab2
FRONTEND_SA = frontend-sa-dt-1
BACKEND_SA = backend-sa-dt-1
FRONTEND_IT = frontend-lab-it
FRONTEND_IG = frontend-lab-ig
BACKEND_IT = backend-lab-it
BACKEND_IG = backend-lab-ig
NETWORK_TAG = open-ssh-tag

.PHONY: all

all: baseline networking accounts compute-create security

baseline: create-project set-project link-billing enable-compute 

networking: baseline create-vpc

accounts: create-role create-fe-sa bind-fe-sa create-be-sa bind-be-sa

compute-create: create-fe-it create-fe-ig set-fe-ig create-be-it create-be-ig set-be-ig

compute-stop: stop-fe-ig stop-be-ig

security: create-fw-rules

delete: unset-project delete-project

create-project:
	# Create new project
	gcloud projects create $(PROJECT_ID) --labels lab_id=$(LAB_LABELS)
	# Check project is created
	gcloud projects describe $(PROJECT_ID)

set-project:
	#Set the project for the current session
	gcloud config set project $(PROJECT_ID)

link-billing:
	# Get list of billing accounts
	gcloud alpha billing accounts list
	#Link billing account to project
	gcloud alpha billing accounts projects link $(PROJECT_ID) --account-id=$(ACCOUNT_ID)
	# Confirm billing account has been linked
	gcloud beta billing accounts --project=$(PROJECT_ID) list

enable-compute:
	#Enable compute service
	gcloud services enable compute.googleapis.com
	#Add gcloud services compute default region and zone using variables above
	gcloud compute project-info add-metadata \
	 --metadata google-compute-default-region=$(DEFAULT_REGION),google-compute-default-zone=$(DEFAULT_ZONE)
	# Check that default region and zone is configured
	gcloud compute project-info describe
	# List compute services and ensure that they are enabled
	gcloud services list --enabled --filter=compute

create-vpc:
	# Create custom VPC
	gcloud compute --project=$(PROJECT_ID) networks create $(CUSTOM_VPC_NAME) \
	 --description="$(CUSTOM_VPC_NAME) for custom VPC lab" --subnet-mode=custom
	# Create frontend subnet and associate to custom VPC
	gcloud compute --project=$(PROJECT_ID) networks subnets create $(FRONTEND_NAME) \
	 --network=$(CUSTOM_VPC_NAME) --region=$(DEFAULT_REGION) --range=$(FRONTEND_SUBNET)
	# Create backend subnet and associate to custom VPC
	gcloud compute --project=$(PROJECT_ID) networks subnets create $(BACKEND_NAME) \
	 --network=$(CUSTOM_VPC_NAME) --region=$(DEFAULT_REGION) --range=$(BACKEND_SUBNET)

create-role:
	# Create IAM role
	gcloud iam roles create $(IAM_ROLE_NAME) --project $(PROJECT_ID) \
	--description "Custom IAM role for GCP Lab 2" --title $(IAM_ROLE_TITLE) \
	--permissions logging.logEntries.create,monitoring.metricDescriptors.create,monitoring.metricDescriptors.get,monitoring.metricDescriptors.list,monitoring.monitoredResourceDescriptors.get,monitoring.monitoredResourceDescriptors.list,monitoring.timeSeries.create \
	--stage GA
	# List IAM role
	gcloud iam roles describe $(IAM_ROLE_NAME) --project $(PROJECT_ID)

create-fe-sa:
	# Create service account for frontend
	gcloud iam service-accounts create $(FRONTEND_SA) --display-name "display-$(FRONTEND_SA)" --description "Frontend Service account for GCP Lab 2"
	# Show service account info
	gcloud iam service-accounts describe $(FRONTEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com

bind-fe-sa:
	# Bind service account for frontend
	gcloud iam service-accounts add-iam-policy-binding \
	  $(FRONTEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com \
	  --member='serviceAccount:$(FRONTEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com' \
	  --role='projects/$(PROJECT_ID)/roles/$(IAM_ROLE_NAME)'
	# List service account
	gcloud iam service-accounts get-iam-policy $(FRONTEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com

create-be-sa:
	# Create service account for backend
	gcloud iam service-accounts create $(BACKEND_SA) --display-name "display-$(BACKEND_SA)" --description "Backend Service account for GCP Lab 2"
	# Show service account info
	gcloud iam service-accounts describe $(BACKEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com

bind-be-sa:
	# Bind service account for backend
	gcloud iam service-accounts add-iam-policy-binding \
	  $(BACKEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com \
	  --member='serviceAccount:$(BACKEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com' \
	  --role='projects/$(PROJECT_ID)/roles/$(IAM_ROLE_NAME)'
	# List service account
	gcloud iam service-accounts get-iam-policy $(BACKEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com

create-fe-it:
	# Create frontend instance template
	gcloud beta compute --project=$(PROJECT_ID) instance-templates \
	create $(FRONTEND_IT) --machine-type=f1-micro --subnet=projects/$(PROJECT_ID)/regions/$(DEFAULT_REGION)/subnetworks/$(FRONTEND_NAME) \
	--network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=$(FRONTEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com \
	--scopes=https://www.googleapis.com/auth/cloud-platform --region=$(DEFAULT_REGION) --image=debian-9-stretch-v20191210 \
	--image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=$(FRONTEND_IT) --reservation-affinity=any \
	-- tags=$(NETWORK_TAG)

create-fe-ig:
	# Create frontend instance group
	gcloud beta compute --project=$(PROJECT_ID) instance-groups managed create $(FRONTEND_IG) \
	--base-instance-name=$(FRONTEND_IG) --template=$(FRONTEND_IT) --size=1 --description="Instance group for Frontend Lab 2" \
	--zones=$(DEFAULT_REGION)-a,$(DEFAULT_REGION)-b,$(DEFAULT_REGION)-c --instance-redistribution-type=PROACTIVE

set-fe-ig:
	# Setup autoscaling for frontend
	gcloud beta compute --project "$(PROJECT_ID)" instance-groups managed \
	set-autoscaling "$(FRONTEND_IG)" --region "$(DEFAULT_REGION)" --cool-down-period "60" --max-num-replicas "2" --min-num-replicas "1" --target-cpu-utilization "0.6"

stop-fe-ig:
	# Stop frontend instance group
	gcloud compute instance-groups managed stop-autoscaling $(FRONTEND_IG) --region=$(DEFAULT_REGION)

create-be-it:
	# Create backend instance template
	gcloud beta compute --project=$(PROJECT_ID) instance-templates \
	create $(BACKEND_IT) --machine-type=f1-micro --subnet=projects/$(PROJECT_ID)/regions/$(DEFAULT_REGION)/subnetworks/$(BACKEND_NAME) \
	--network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=$(BACKEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com \
	--scopes=https://www.googleapis.com/auth/cloud-platform --region=$(DEFAULT_REGION) --image=debian-9-stretch-v20191210 \
	--image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=$(BACKEND_IT) --reservation-affinity=any

create-be-ig:
	# Create backend instance group
	gcloud beta compute --project=$(PROJECT_ID) instance-groups managed create $(BACKEND_IG) \
	--base-instance-name=$(BACKEND_IG) --template=$(BACKEND_IT) --size=1 --description="Instance group for Backend Lab 2" \
	--zones=$(DEFAULT_REGION)-a,$(DEFAULT_REGION)-b,$(DEFAULT_REGION)-c --instance-redistribution-type=PROACTIVE
	
set-be-ig:
	# Setup autoscaling for backend
	gcloud beta compute --project "$(PROJECT_ID)" instance-groups managed \
	set-autoscaling "$(BACKEND_IG)" --region "$(DEFAULT_REGION)" --cool-down-period "60" --max-num-replicas "2" --min-num-replicas "1" --target-cpu-utilization "0.6"

stop-be-ig:
	# Stop frontend instance group
	gcloud compute instance-groups managed stop-autoscaling $(BACKEND_IG) --region=$(DEFAULT_REGION)

create-fw-rules:
	# Firewall rules for inbound SSH on network tags
	gcloud compute --project=$(PROJECT_ID) firewall-rules create open-ssh-by-tag-fwr \
	--direction=INGRESS --priority=400 --network=$(CUSTOM_VPC_NAME) --action=ALLOW --rules=tcp:22 \
	--source-ranges=0.0.0.0/0 --target-tags=$(NETWORK_TAG)
	# Create firewall rules for incoming frontend to backend
	gcloud compute --project=$(PROJECT_ID) firewall-rules create allow-frontend-to-backend-fwr --description="Frontend service accounts to backend service accounts" \
	--direction=INGRESS --priority=700 --network=$(CUSTOM_VPC_NAME) --action=ALLOW --rules=icmp --source-service-accounts=$(FRONTEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com \
	--target-service-accounts=$(BACKEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com
	 # Create firewall rules for incoming backend to backend
	gcloud compute --project=$(PROJECT_ID) firewall-rules create allow-backend-to-backend-fwr --description="Backend service accounts to backend service accounts" \
	--direction=INGRESS --priority=800 --network=$(CUSTOM_VPC_NAME) --action=ALLOW --rules=icmp --source-service-accounts=$(BACKEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com \
	--target-service-accounts=$(BACKEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com
	# Create firewall rule to block all connections from backend
	gcloud compute --project=$(PROJECT_ID) firewall-rules create block-all-connections-from-backend-fwr --description="Block all connections from backend" \
	--direction=INGRESS --priority=1000 --network=$(CUSTOM_VPC_NAME) --action=DENY --rules=all \
	--source-service-accounts=$(BACKEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com
	# Create firewall rules for incoming service account from Cloud Shell
	gcloud compute --project=$(PROJECT_ID) firewall-rules create allow-icmp-incoming-to-frontend-fwr --description="Incoming ICMP to frontend"\
	--direction=INGRESS --priority=1200 --network=$(CUSTOM_VPC_NAME) --action=ALLOW --rules=icmp --source-ranges=0.0.0.0/0 \
	--target-service-accounts=$(FRONTEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com
	# Egress rule to block all outbound traffic
	gcloud compute --project=$(PROJECT_ID) firewall-rules create no-backend-egress-fwr --description="Block all backend egress traffic." \
	--direction=EGRESS --priority=2000 --network=$(CUSTOM_VPC_NAME) --action=DENY --rules=all --destination-ranges=0.0.0.0/0 \
	--target-service-accounts=$(BACKEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com
	# Egress rule to allow backend to backend
	gcloud compute --project=$(PROJECT_ID) firewall-rules create allow-backend-egress-to-vpc-fwr --description="Allow all backend to backend egress traffic." \
	--direction=EGRESS --priority=1000 --network=$(CUSTOM_VPC_NAME) --action=ALLOW --rules=icmp --destination-ranges=192.168.0.0/18 \
	--target-service-accounts=$(BACKEND_SA)@$(PROJECT_ID).iam.gserviceaccount.com

unset-project:
	# Unset project for current session
	gcloud config unset project

delete-project:
	# Delete project and WITHOUT prompting the user to confirm
	gcloud projects delete $(PROJECT_ID) --quiet
