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
IAM_ROLE_NAME = dt_custom_role_121
IAM_ROLE_TITLE = dtCustomRole121
FRONTEND_SA = frontend-sa-dt-121
BACKEND_SA = backend-sa-dt-121

.PHONY: all

all: create-vpc

accounts: create-role create-fe-sa bind-fe-sa create-be-sa bind-be-sa

create-vpc:
	# Create custom VPC
	gcloud compute --project=$(PROJECT_ID) networks create $(CUSTOM_VPC_NAME) --description="$(CUSTOM_VPC_NAME) for custom VPC lab" --subnet-mode=custom
	# Create backend subnet and associate to custom VPC
	gcloud compute --project=$(PROJECT_ID) networks subnets create $(FRONTEND_NAME)-subnet --network=$(CUSTOM_VPC_NAME) --region=$(DEFAULT_REGION) --range=$(FRONTEND_SUBNET)
	# Create backend subnet and associate to custom VPC
	gcloud compute --project=$(PROJECT_ID) networks subnets create $(BACKEND_NAME)-subnet --network=$(CUSTOM_VPC_NAME) --region=$(DEFAULT_REGION) --range=$(BACKEND_SUBNET)

create-role:
	# Create IAM role
	gcloud iam roles create $(IAM_ROLE_NAME) --project $(PROJECT_ID) \
	--description "Custom role description" --title $(IAM_ROLE_TITLE) \
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