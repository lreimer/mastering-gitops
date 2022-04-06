# Mastering GitOps

Demo repository for my Crossplane talk at the Mastering GitOps conference.

## Prerequisites

You need to have the following tools installed locally to be able to complete all steps:
- [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- [eksctl](https://eksctl.io/)
- [gcloud CLI](https://cloud.google.com/sdk/gcloud)
- [kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
- [flux](https://fluxcd.io/docs/get-started/)
- [Helm](https://helm.sh/docs/intro/install/)

## Bootstrapping

```bash
# define required ENV variables for the next steps to work
$ export AWS_ACCOUNT_ID=export AWS_ACCOUNT_ID=`aws sts get-caller-identity --query Account --output text`
$ export GITHUB_USER=lreimer
$ export GITHUB_TOKEN=<your-token>

# setup an EKS cluster with Flux2
$ make create-eks-cluster
$ make bootstrap-eks-flux2

# setup a GKE cluster with Flux2
$ make create-gke-cluster
$ make bootstrap-gke-flux2

$ make destroy-clusters
```

## Maintainer

M.-Leander Reimer (@lreimer), <mario-leander.reimer@qaware.de>

## License

This software is provided under the MIT open source license, read the `LICENSE`
file for details.
