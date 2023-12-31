# Hacks Notes

## Notes

## Argo Workflows

Setup
```shell
argo auth token
kubectl -n argo-workflows port-forward deployment.apps/argo-workflows-server 2746:2746


```

Create AWS HCP Cluster using Argo Workflows

```shell
kubectl create configmap pcluster-config \
  --from-file=config=cluster-config.yaml

kubectl delete configmap pcluster-config

kubectl get configmap pcluster-config -o yaml

kubectl create secret generic aws-creds \
  --from-literal AWS_ACCESS_KEY_ID=REPLACEME \
  --from-literal AWS_SECRET_ACCESS_KEY=REPLACEME \
  --from-literal AWS_DEFAULT_REGION=us-east-2

kubectl apply -f create-hpc-template.yaml
kubectl apply -f describe-hpc-template.yaml
kubectl apply -f update-hpc-template.yaml
kubectl apply -f update-hpc-compute-fleet-template.yaml

k apply -f argo-workflows/
argo template list
argo submit --watch --from workflowtemplate/pcluster-create-cluster \
  -p cluster="test-cluster" \
  -p cluster_version="3.7.1" \
  -p config="$(cat cluster-config.yaml)"

argo submit --watch --from workflowtemplate/pcluster-list-clusters \
  -p cluster_version="3.7.1"

argo submit --watch --from workflowtemplate/pcluster-delete-cluster \
  -p cluster="test-cluster" \
  -p cluster_version="3.7.1"

argo submit --watch --from workflowtemplate/pcluster-describe-cluster \
  -p cluster="test-cluster" \
  -p cluster_version="3.7.1"

argo submit --watch --from workflowtemplate/pcluster-update-compute-fleet \
  -p cluster="test-cluster" \
  -p cluster_version="3.7.1" \
  -p status="STOP_REQUESTED"

argo submit --watch --from workflowtemplate/pcluster-update-cluster \
  -p cluster="test-cluster" \
  -p cluster_version="3.7.1" \
  -p config="$(cat cluster-config.yaml)"

argo submit --watch --from workflowtemplate/pcluster-describe-compute-fleet \
  -p cluster="test-cluster" \
  -p cluster_version="3.7.1"

argo logs @latest

kubectl apply -f describe-hpc-template.yaml
argo template list
argo submit --watch --from workflowtemplate/describe-hpc
argo logs @latest
```


## Argo Events

```shell

k create ns hpc
k create cm -n hpc hpc-my-cluster-3 --from-literal=cluster=my-cluster-3 --from-file=config=../cluster-config.yaml
k delete cm -n hpc hpc-my-cluster
```


## Debug

```shell
python3 -m pip install --upgrade pip
python3 -m pip install --user --upgrade virtualenv
python3 -m virtualenv hpc-ve
source hpc-ve/bin/activate
```

```shell
pip3 install awscli
pip3 install aws-parallelcluster@3.7.0
```

Create HPC cluster
```shell
export AWS_DEFAULT_REGION="us-east-2"
export HPC_CLUSTER_NAME="hcp-carlos"
export HPC_CONFIG_FILE="cluster-config.yaml"

# create vpc and cluster-config.yaml
pcluster configure --config $HPC_CONFIG_FILE



pcluster create-cluster --cluster-configuration $HPC_CONFIG_FILE --cluster-name $HPC_CLUSTER_NAME --region $AWS_DEFAULT_REGION


pcluster update-compute-fleet --cluster-name $HPC_CLUSTER_NAME --status STOPPED


pcluster delete-cluster -n $HPC_CLUSTER_NAME --debug
```

```json
{
  "cluster": {
    "clusterName": "hcp-carlos",
    "cloudformationStackStatus": "CREATE_IN_PROGRESS",
    "cloudformationStackArn": "arn:aws:cloudformation:us-east-2:$ACCOUNT:stack/hcp-carlos/e9b93440-5962-11ee-b8b2-0af6f2ba9b3d",
    "region": "us-east-2",
    "version": "3.7.0",
    "clusterStatus": "CREATE_IN_PROGRESS",
    "scheduler": {
      "type": "slurm"
    }
  }
}
```
pcluster describe-cluster --cluster-name $HPC_CLUSTER_NAME
```json
{
  "creationTime": "2023-09-22T16:13:06.037Z",
  "version": "3.7.0",
  "clusterConfiguration": {
    "url": "REDACTED"
  },
  "tags": [
    {
      "value": "3.7.0",
      "key": "parallelcluster:version"
    },
    {
      "value": "hcp-carlos",
      "key": "parallelcluster:cluster-name"
    }
  ],
  "cloudFormationStackStatus": "CREATE_IN_PROGRESS",
  "clusterName": "hcp-carlos",
  "computeFleetStatus": "UNKNOWN",
  "cloudformationStackArn": "arn:aws:cloudformation:us-east-2:$ACCOUNT:stack/hcp-carlos/e9b93440-5962-11ee-b8b2-0af6f2ba9b3d",
  "lastUpdatedTime": "2023-09-22T16:13:06.037Z",
  "region": "us-east-2",
  "clusterStatus": "CREATE_IN_PROGRESS",
  "scheduler": {
    "type": "slurm"
  }
}
```

```shell
pcluster describe-cluster --cluster-name $HPC_CLUSTER_NAME --region $AWS_DEFAULT_REGION | jq .clusterStatus
"CREATE_IN_PROGRESS"

pcluster describe-cluster --cluster-name $HPC_CLUSTER_NAME --region $AWS_DEFAULT_REGION | jq .clusterStatus
"CREATE_COMPLETE"
```

```json
{
  "creationTime": "2023-09-22T16:13:06.037Z",
  "headNode": {
    "launchTime": "2023-09-22T16:17:40.000Z",
    "instanceId": "i-0a9d0ede395579f2c",
    "publicIpAddress": "18.118.10.7",
    "instanceType": "t2.micro",
    "state": "running",
    "privateIpAddress": "10.0.0.228"
  },
  "version": "3.7.0",
  "clusterConfiguration": {
    "url": "REDACTED"
  },
  "tags": [
    {
      "value": "3.7.0",
      "key": "parallelcluster:version"
    },
    {
      "value": "hcp-carlos",
      "key": "parallelcluster:cluster-name"
    }
  ],
  "cloudFormationStackStatus": "CREATE_COMPLETE",
  "clusterName": "hcp-carlos",
  "computeFleetStatus": "RUNNING",
  "cloudformationStackArn": "arn:aws:cloudformation:us-east-2:$ACCOUNT:stack/hcp-carlos/e9b93440-5962-11ee-b8b2-0af6f2ba9b3d",
  "lastUpdatedTime": "2023-09-22T16:13:06.037Z",
  "region": "us-east-2",
  "clusterStatus": "CREATE_COMPLETE",
  "scheduler": {
    "type": "slurm"
  }
}
```


```shell
aws --region $AWS_DEFAULT_REGION cloudformation list-stacks \
   --stack-status-filter "CREATE_COMPLETE" \
   --query "StackSummaries[].StackName" | \
   grep -e "parallelclusternetworking-"
```


### Tutorial

```shell
export HPC_SSH_KEY="$HOME/.ssh/hpc-carlos.pem"
pcluster ssh -i $HPC_SSH_KEY --cluster-name $HPC_CLUSTER_NAME
```


```shell
squeue
sinfo
```

>>PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
  queue1*      up   infinite     10  idle~ queue1-dy-t2micro-[1-10]

```shell
#!/bin/bash
#SBATCH --job-name=test_job
#SBATCH --nodes=3
#SBATCH --ntasks-per-node=1

echo "Hello from SLURM! Running on node: $(hostname)"

```

```shell
sbatch test_job.sh
squeue
scontrol show job <job_id>
cat slurm-<job_id>.out
cat slurm-<job_id>.err
```

```shell
srun --nodes=3 --ntasks-per-node=1 hostname
```


get logs
sudo cat /var/log/slurmctld.log

sbatch -N 2 --wrap "srun test_job.sh"

sbatch -N 3 -p spot -C "[c5.large*1&t2.micro*2]" --wrap "srun test_job.sh"

argo submit --watch create-hpc.yaml

argo logs @latest

