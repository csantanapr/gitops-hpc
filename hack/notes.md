# Hacks Notes

## Notes

Create AWS HCP Cluster using Argo Workflows

```shell
kubectl create configmap pcluster-config \
  --from-file=config=cluster-config.yaml

kubectl create secret generic aws-creds \
  --from-literal AWS_ACCESS_KEY_ID=REPLACEME \
  --from-literal AWS_SECRET_ACCESS_KEY=REPLACEME \
  --from-literal AWS_DEFAULT_REGION=us-east-2

kubectl apply -f create-hpc-template.yaml
argo template list
argo submit --watch --from workflowtemplate/create-hpc

argo logs @latest
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
```

Create HPC cluster
```shell
export AWS_DEFAULT_REGION="us-east-2"
export HPC_CLUSTER_NAME="hcp-carlos"
export HPC_CONFIG_FILE="cluster-config.yaml"

pcluster configure --config $HPC_CONFIG_FILE



pcluster create-cluster --cluster-configuration $HPC_CONFIG_FILE --cluster-name $HPC_CLUSTER_NAME --region $AWS_DEFAULT_REGION
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
pcluster describe-cluster --cluster-name $HPC_CLUSTER_NAME --region $AWS_DEFAULT_REGION
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
pcluster ssh -i $HPC_SSH_KEY --cluster-name $HPC_CLUSTER_NAME --region $AWS_DEFAULT_REGION
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

