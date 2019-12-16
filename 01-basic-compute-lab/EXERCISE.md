# Requirements #

Below are the requirements for Exercise 1:
 
1) Create brand new project
2) Create GCE instance that runs provided script `worker-startup-script.md`
3) System logs should available in Stackdriver Logs
4) New GCS bucket for resulting log files
5) Log file appears in new bucket after instance finishes starting up
6) No need to SSH to instance

## Solution 

I've used a Makefile to deploy this solution. To execute, perform the following:

```
make
```