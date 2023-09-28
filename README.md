# Deploy AWS HPC Parallel Clusters usign Amazon EKS and GitOps

Deploy [AWS ParallelCluster](https://aws.amazon.com/hpc/parallelcluster/) using GitOps on [Amazon EKS](https://aws.amazon.com/eks/)

The main components to deploy HPC Clusters using Amazon EKS:
1. [Argo Events](https://argoproj.github.io/argo-events/) using a EventSource, this watches for confimap events (ADD, UPDATE, DELETE), any configmap with the label `app=pcluster` in any namespace
2. [Argo Workflows](https://argoproj.github.io/argo-workflows/) get triggered by Argo Sensor for each event
3. [Argo CD](https://argo-cd.readthedocs.io/en/stable/) deploy ConfigMap with HPC Parallel Cluster specification

Deploy manually or automatically using GitOps(ArgoCD) using the AWS EKS Blueprints GitOps Bridge [infra/eks](./infra/eks):
```shell
kubectl apply gitops/argo-workflows/
kubectl apply gitops/argo-events/
kubectl apply gitops/hpc-clusters/
```

Try the Argo Workflows
```shell
# Terminal logs
stern -n A pcluster

argo submit --watch --from \
clusterworkflowtemplate/pcluster-create-cluster \
-p cluster_name="my-cluster" \
-p cluster_config="$(cat examples/cluster-config.yaml)"

argo submit --watch --from \
clusterworkflowtemplate/pcluster-list-clusters

argo submit --watch --from \
clusterworkflowtemplate/pcluster-describe-cluster \
-p cluster_name=my-cluster

argo submit --watch --from \
clusterworkflowtemplate/pcluster-describe-compute-fleet \
-p cluster_name=my-cluster

argo submit --watch --from \
clusterworkflowtemplate/pcluster-update-compute-fleet \
-p cluster_name=my-cluster \
-p status=STOP_REQUESTED

argo submit --watch --from \
clusterworkflowtemplate/pcluster-update-cluster \
-p cluster_name="my-cluster" \
-p cluster_config="$(cat examples/cluster-config-updated.yaml)"

argo submit --watch --from \
clusterworkflowtemplate/pcluster-update-compute-fleet \
-p cluster_name=test-cluster \
-p status=START_REQUESTED

argo submit --watch --from \
clusterworkflowtemplate/pcluster-delete-cluster \
-p cluster_name=my-cluster-1

```

Create HPC Parallel Cluster by creating configmap
```shell
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: hpc-my-cluster-test
  namespace: hpc
  labels:
    app: pcluster
data:
  cluster_name: hpc-my-cluster-test
  cluster_version: "3.7.1"
  region: us-east-2
  cluster_config: |
    Region: us-east-2
    Image:
      Os: ubuntu2004
    HeadNode:
      InstanceType: t2.micro
      Networking:
        SubnetId: subnet-0b439e3159831d2dd
      Ssh:
        KeyName: hpc-carlos
    Scheduling:
      Scheduler: slurm
      SlurmQueues:
      - Name: queue1
        ComputeResources:
        - Name: t2micro
          Instances:
          - InstanceType: t3.small
          MinCount: 0
          MaxCount: 10
        Networking:
          SubnetIds:
          - subnet-05f258192d3e81132
EOF
```

Configure AWS Credentials
```shell
kubectl create secret generic aws-creds -n default --from-literal=AWS_ACCESS_KEY_ID=xyz --from-literal=AWS_SECRET_ACCESS_KEY=xyz
```
> Workflows run in `default` namespace, you can update the argo event sensors




### Resources
- https://github.com/aws/aws-parallelcluster
- https://gallery.ecr.aws/parallelcluster/pcluster-api
- https://docs.aws.amazon.com/parallelcluster/v2/ug/what-is-aws-parallelcluster.html
- https://docs.aws.amazon.com/parallelcluster/v2/ug/getting-started-configuring-parallelcluster.html
- https://slurm.schedmd.com/rosetta.pdf
- https://slurm.schedmd.com/quickstart.html
- https://www.nccs.nasa.gov/nccs-users/instructional/using-slurm


