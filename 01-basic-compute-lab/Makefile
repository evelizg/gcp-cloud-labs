# Populate with your variables
PROJECT_ID := gcp-lab-challenge-3255
# To retrieve your billing account id, run the command `gcloud alpha billing accounts list` from cloud shell
ACCOUNT_ID := 12345-654321-ABCDEF
DEFAULT_REGION := us-west
DEFAULT_ZONE := us-west1-b
BUCKET = gs://gcp-challenge-lab-1-logging-2233/
INSTANCE_NAME = lab-vm-2

.PHONY: all

all: release

release:
	# Create new project
	gcloud projects create $(PROJECT_ID) --labels labid=gcp-lab-11-11
	# Check project is created
	gcloud projects describe $(PROJECT_ID)
	#Set the project for the current session
	gcloud config set project $(PROJECT_ID)
	# Get list of billing accounts
	gcloud alpha billing accounts list
	#Link billing account to project
	gcloud alpha billing accounts projects link $(PROJECT_ID) --account-id=$(ACCOUNT_ID)
	# Confirm billing account has been linked
	gcloud beta billing accounts --project=$(PROJECT_ID) list
	#nable compute service
	gcloud services enable compute.googleapis.com
	#Add gcloud services compute default region and zone using variables above
	gcloud compute project-info add-metadata \
	    --metadata google-compute-default-region=$(DEFAULT_REGION),google-compute-default-zone=$(DEFAULT_ZONE)
	# Check that default region and zone is configured
	gcloud compute project-info describe
	# List compute services and ensure that they are enabled
	gcloud services list --enabled --filter=compute
	# List current buckets
	gsutil ls
	# Create storage bucket
	gsutil mb -l us $(BUCKET)
	# List created storage bucket
	gsutil ls $(BUCKET)
	# chmod Upload worker script file so that it is executable
	chmod +x worker-startup-script.sh
	# Create instance, using the `worker-startup-script.sh` as a startup script.
	gcloud compute --project=$(PROJECT_ID) \
	instances create $(INSTANCE_NAME) --description="GCE Lab Challenge 1" --zone=$(DEFAULT_ZONE) \
	--machine-type=f1-micro --image=debian-9-stretch-v20191121 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard \
	--boot-disk-device-name=instance-1 --reservation-affinity=any --metadata-from-file startup-script=./worker-startup-script.sh \
	--metadata lab-logs-bucket=$(BUCKET) --scopes=https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/trace.append,https://www.googleapis.com/auth/devstorage.write_only
	# Check that the instance is up
	gcloud compute instances list
	# Wait 300 seconds so that stress test can run and copy the file to the storage bucket
	sleep 300
	# Check that stress test results are in bucket
	gsutil cat $(BUCKET)machine-$(INSTANCE_NAME)-finished.txt

