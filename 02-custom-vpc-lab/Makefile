# Populate with your variables
PROJECT_ID := gcp-challenge-1-net
# To retrieve your billing account id, run the command `gcloud alpha billing accounts list` from cloud shell
ACCOUNT_ID := XXXXXX-XXXXXX-XXXXXX
DEFAULT_REGION := us-west1
DEFAULT_ZONE := us-west1-b
CUSTOM_VPC_NAME := custom-vpc-lab-cc1
FRONTEND_NAME:= frontend-d1
FRONTEND_SUBNET := 192.168.202.0/24
BACKEND_NAME:= backend-d1
BACKEND_SUBNET := 192.168.203.0/24
LAB_LABELS = gcp-custom-vpc

.PHONY: all

all: create-vpc

create-vpc:
    # Create custom VPC
	gcloud compute --project=$(PROJECT_ID) networks create $(CUSTOM_VPC_NAME) --description="$(CUSTOM_VPC_NAME) for custom VPC lab" --subnet-mode=custom
	# Create backend subnet and associate to custom VPC
	gcloud compute --project=$(PROJECT_ID) networks subnets create $(FRONTEND_NAME)-subnet --network=$(CUSTOM_VPC_NAME) --region=$(DEFAULT_REGION) --range=$(FRONTEND_SUBNET)
	# Create backend subnet and associate to custom VPC
	gcloud compute --project=$(PROJECT_ID) networks subnets create $(BACKEND_NAME)-subnet --network=$(CUSTOM_VPC_NAME) --region=$(DEFAULT_REGION) --range=$(BACKEND_SUBNET)
create-role:
	# Create IAM role
	gcloud iam roles create YZZCustomGCERole --project gcp-challenge-1-net \
	--description "Custom role description" --title YZZmyCustomGCE \
	--permissions logging.logEntries.create,monitoring.metricDescriptors.create,monitoring.metricDescriptors.get,monitoring.metricDescriptors.list,monitoring.monitoredResourceDescriptors.get,monitoring.monitoredResourceDescriptors.list,monitoring.timeSeries.create \
	--stage GA
    # List IAM role
	gcloud iam roles describe YZZCustomGCERole --project gcp-challenge-1-net
	# Create service account
	gcloud iam service-accounts create yzz-custom-gce-sa --display-name "display-yzz-custom-gce-sa"
	# Show service account info
	gcloud iam service-accounts describe yzz-custom-gce-sa@gcp-challenge-1-net.iam.gserviceaccount.com
	# Bind service account
	gcloud iam service-accounts add-iam-policy-binding \
      yzz-custom-gce-sa@gcp-challenge-1-net.iam.gserviceaccount.com \
      --member='serviceAccount:yzz-custom-gce-sa@gcp-challenge-1-net.iam.gserviceaccount.com' \
      --role='roles/YZZCustomGCERole'