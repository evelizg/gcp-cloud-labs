# Requirements #

Below are the requirements for Exercise 1:
 
1) Create brand new project
2) Create GCE instance that runs provided script `worker-startup-script.md`
3) System logs should available in Stackdriver Logs
4) New GCS bucket for resulting log files
5) Log file appears in new bucket after instance finishes starting up
6) No need to SSH to instance

## Solution 

I've used a Makefile to deploy this solution. 

This Makefile has four operations:

1) `make project-baseline` - Create project baseline
2) `make compute` - Deploy a compute instance inside an existing baseline project
3) `make` - Create project baseline and deploy compute
4) `make remove` - Delete the entire project without prompting the user to confirm.

### Pre-requisites

Edit the `Makefile` with the appropriate variables for your environment. Most notably, you will need to retrieve your billing id by running the following command from inside the gcloud shell:

```gcloud
gcloud alpha billing accounts list
```

The Makefile has been commented as much as possible, you can interpret what is being executed and why.


### 1) `make project-baseline` - Create project baseline

This command will perform all the baseline tasks required prior to creating and validating the compute instance.
At a high level, these tasks are:

- Create the project - `create-project`
- Set the project so that the ensuing commands run from the project context - `set-project`
- Link the billing account to the project - `link-billing`
- Enable the compute service, so that compute instances can be created - `enable-compute`
- Create a bucket to store the diagnostic logs in - `create-bucket`
- Change the permissions on the startup script, so that it is executable - `prepare-script`

To execute, perform the following:

```console
make project-baseline
```

### 2) `make compute` - Deploy a compute instance inside an existing baseline project

This command will deploy the compute instance inside an existing project and copy the stress test log to an existing bucket:
At a high level, these tasks are:

- Deploy the compute instance with the startup script `worker-startup-script.sh` and pass the bucket as metadata to the instance - `deploy-compute`
- Wait 300 seconds for the `worker-startup-script.sh` to execute, then read the output file from the bucket which should have been copied as a result of the startup script successfully executing - `validate-compute`

To execute, perform the following:

```console
make compute
```

### 3) `make` - Create project baseline and deploy compute

This command is the combination of the targets `project-baseline` and `compute` in that order so that the deployment is successful.

To execute, perform the following:

```console
make 
```

### 4) `make remove` - Delete project, without user prompt

This command will delete the project, without prompting the user to confirm the deletion. **Take care in using the command.**
At a high level, these tasks are:

- Unset the project so that the ensuing commands run successfully - `unset-project`
- Delete the project and disable the user prompt - `delete-project`

To execute, perform the following:

```console
make remove
```
