# GitOps for HPC

Deploy [AWS ParallelCluster](https://aws.amazon.com/hpc/parallelcluster/) using GitOps on [Amazon EKS](https://aws.amazon.com/eks/)





Run Argo Workflows

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
-p cluster_name=my-cluster

```





### Resources
- https://github.com/aws/aws-parallelcluster
- https://gallery.ecr.aws/parallelcluster/pcluster-api
- https://docs.aws.amazon.com/parallelcluster/v2/ug/what-is-aws-parallelcluster.html
- https://docs.aws.amazon.com/parallelcluster/v2/ug/getting-started-configuring-parallelcluster.html
- https://slurm.schedmd.com/rosetta.pdf
- https://slurm.schedmd.com/quickstart.html
- https://www.nccs.nasa.gov/nccs-users/instructional/using-slurm


