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

# modify Flux kustomization and add
# - cluster-sync.yaml
# - notification-receiver.yaml
# - receiver-service.yaml
# - webhook-token.yaml
# - image-update-automation.yaml

# you also need to create the webhook for the Git Repository
# Payload URL: http://<LoadBalancerAddress>/<ReceiverURL>
# Secret: the webhook-token value
$ kubectl -n flux-system get svc/receiver
$ kubectl -n flux-system get receiver/webapp

$ make destroy-clusters
```

## Local Installation

For local installation simply follow the instructions found on the official [Crossplane documentation](https://crossplane.io/docs/v1.7/getting-started/install-configure.html).

```bash
# install latest Crossplane release using Helm in a dedicated namespace
kubectl create namespace crossplane-system

helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

helm install crossplane --namespace crossplane-system crossplane-stable/crossplane --set provider.packages={crossplane/provider-aws:v0.25.0}

## check everything came up OK
helm list -n crossplane-system
kubectl get all -n crossplane-system
```

## AWS Provider

For AWS the configuration needs to reference the required credentials in the form of a secret.
These are basically the `aws_access_key_id` and `aws_secret_access_key` from the default profile found in the `${HOME}/.aws/credentials` file. With this information we can create a secret and reference it from a provider config resource.

```bash
kubectl create secret generic aws-credentials -n crossplane-system --from-file=credentials=${HOME}/.aws/credentials

# we could manually installe the AWS provider
# kubectl crossplane install provider crossplane/provider-aws:v0.24.1

cd crossplane
kubectl apply -n crossplane-system -f aws/provider.yaml
kubectl apply -n crossplane-system -f aws/providerconfig.yaml

kubectl get events
kubectl get crds

# create an S3 bucket in eu-central-1
kubectl apply -f aws/s3/bucket.yaml
aws s3 ls

# create an ECR in eu-central-1
kubectl apply -f aws/ecr/repository.yaml
aws ecr describe-repositories

# create SNS topic and subscription
kubectl apply -f aws/sns/topic.yaml
aws sns list-topics
kubectl apply -f aws/sns/subscription.yaml
aws sns list-subscriptions
aws sns publish --subject Test --message Crossplane --topic-arn arn:aws:sns:eu-central-1:<ACCOUNT_NUMBER>:email-topic

# create a SQS queue
kubectl apply -f aws/sqs/queue.yaml
aws sqs list-queues

# create EFS file system and mount point
k apply -f aws/efs/filesystem.yaml
aws efs describe-file-systems
k apply -f aws/efs/mounttarget.yaml
# the mount target takes some time to be available
aws efs describe-mount-targets --file-system-id <FileSystemId>

# create MQ brokers
kubectl create secret generic test-activemq-admin -n crossplane-system --from-literal=password=testPassword123
kubectl apply -f aws/mq/activemq-broker.yaml
kubectl get broker
kubectl describe secret test-activemq

kubectl create secret generic test-rabbitmq-admin -n crossplane-system --from-literal=password=testPassword123
kubectl apply -f aws/mq/rabbitmq-broker.yaml
kubectl get broker
kubectl describe secret test-rabbitmq

# use XRD to create an ECR
kubectl apply -f xrd/repository/definition.yaml
kubectl apply -f xrd/repository/composition.yaml
kubectl apply -f xrd/repository/examples/example-repository.yaml

cd xrd/repository/
kubectl crossplane build configuration --ignore=examples/example-repository.yaml

# use XRD to create an S3 bucket
kubectl apply -f xrd/bucket/definition.yaml
kubectl apply -f xrd/bucket/composition.yaml
kubectl apply -f xrd/bucket/examples/example-bucket.yaml

cd xrd/bucket/
kubectl crossplane build configuration --ignore=examples/example-bucket.yaml

# use XRD to create PostgreSQL instance
kubectl apply -f xrd/postgresql/definition.yaml
kubectl apply -f xrd/postgresql/composition.yaml
kubectl apply -f xrd/postgresql/examples/example-db.yaml

kubectl get postgresqlinstances.db.aws.qaware.de example-db
kubectl get claim

kubectl get secrets
kubectl describe secret example-db-conn

kubectl apply -f xrd/postgresql/examples/example-db-client.yaml
kubectl get pods
kubectl logs example-db-client-sjdh7

cd xrd/postgresql/
kubectl crossplane build configuration --ignore=examples/example-db.yaml,examples/example-db-client.yaml
```

## GCP Provider

For examples of the GCP provider have a look the [Github repository](https://github.com/crossplane/provider-gcp/tree/master/examples)

```bash
# we need to create a GCP service account and secret
gcloud iam service-accounts create crossplane-system --display-name=Crossplane
gcloud iam service-accounts keys create gcp-credentials.json --iam-account crossplane-system@cloud-native-night.iam.gserviceaccount.com
kubectl create secret generic gcp-credentials -n crossplane-system --from-file=credentials=./gcp-credentials.json
```

## Maintainer

M.-Leander Reimer (@lreimer), <mario-leander.reimer@qaware.de>

## License

This software is provided under the MIT open source license, read the `LICENSE`
file for details.
