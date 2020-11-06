


[![Join Discord](https://img.shields.io/badge/Join%20us%20on-Discord-e01563.svg)](https://discord.gg/72JDKy4)

# Devtron Installation

Devtron is an open source software delivery workflow for kubernetes written in go. It is designed as a self-serve platform for operationalizing and maintaining applications (AppOps) on kubernetes in a developer friendly way.

## Introduction

This chart bootstraps deployment of all required components for installation of [Devtron Platform](https://github.com/devtron-labs) on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

It packages third party components like 

 - [Grafana](https://github.com/grafana/grafana) for displaying application metrics 
 - [Argocd](https://github.com/argoproj/argo-cd/) for gitops 
 - [Argo workflows](https://github.com/argoproj/argo) for CI
 - [Clair](https://github.com/quay/clair) & [Guard](https://github.com/guard/guard) for image scanning
 - [Kubernetes External Secrets](https://github.com/godaddy/kubernetes-external-secrets) for ingegrating with external secret management stores like [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) or [HashiCorp Vault](https://www.vaultproject.io/)
 - [Nats](https://github.com/nats-io) for event streaming
 - [Postgres](https://github.com/postgres/postgres) as datastore
 - Fork of [Argo Rollout](https://github.com/argoproj/argo-rollouts) 

## How to use it

### Install with Helm

This chart is currently not available on the official helm repository therefore you need to download it to install it.

```bash
$ git clone [https://github.com/devtron-labs/devtron-installation-script.git](https://github.com/devtron-labs/devtron-installation-script.git)
$ cd devtron-installation-script/charts
$ #modify values in values.yaml
$ kubectl create ns devtroncd
$ helm install devtron . -f values.yaml
```
For more details about configuration see the [helm chart configuration](#configuration)

### Install with kubectl

If you don't want to install helm on your cluster and just want to use `kubectl` to install `devtron platform`, then please follow following steps:

```bash
$ git clone [https://github.com/devtron-labs/devtron-installation-script.git](https://github.com/devtron-labs/devtron-installation-script.git)
$ cd devtron-installation-script/
$ kubectl apply create ns devtroncd
$ kubectl -n devtroncd apply -f devtron/crds
$ # wait for crd to install
$ kubectl apply -n devtroncd -f charts/devtron/templates/install.yaml
$ #edit install/devtron-operator-configs.yaml
$ kubectl apply -n devtroncd -f install/devtron-operator-configs.yaml
$ kubectl apply -n devtroncd -f charts/devtron/templates/devtron-installer.yaml
```
### Access devtron dashboard

devtron dashboard in now available at the `BASE_URL/dashboard`, where `BASE_URL` is same as provided in `values.yaml` in case of installation via helm chart OR provided in `charts/template/configmap-secret.yaml` in case of installation via kubectl.

For login use username:`admin` and for password run command mentioned below.
```bash
$ kubectl -n devtroncd get secret devtron-secret -o jsonpath='{.data.ACD_PASSWORD}' | base64 -d
```
To log into grafana use username: `admin` and for password run command mentioned below.
```bash
$ kubectl -n devtroncd get secret devtron-grafana-cred-secret -o jsonpath='{.data.admin-password}' | base64 -d
```
### Configuration

All parameters mentioned in the `values.yaml` are mandatory.

First section is ***secrets.env*** and it has following properties
| Parameter | Description | Default |
|----------:|:------------|:--------|
| **POSTGRESQL_PASSWORD*** | password for postgres database (required) | change-me |
| **GIT_TOKEN** | git token for the gitops work flow, please note this is not for source code of repo and this token should have full access to create, delete, update repository (required) |  |
| **WEBHOOK_TOKEN** | If you want to continue using jenkins for CI then please provide this for authentication of requests  |  |

Second section is ***configs*** and has following properties
| Parameter | Description | Default |
|----------:|:------------|:--------|
| **BASE_URL_SCHEME** | either of http or https (required) | http |
| **BASE_URL** | url without scheme and trailing slash `eg. devtron.ai` (required) | `change-me` |
| **DEX_CONFIG** | dex config if you want to integrate login with SSO (optional) for more information check [Argocd documentation](https://argoproj.github.io/argo-cd/operator-manual/user-management/) | 
| **GIT_PROVIDER** | git provider for storing config files for gitops, currently only GITHUB and GITLAB are supported (required) | `GITHUB` | |
| **GITLAB_NAMESPACE_ID** | if GIT_PROVIDER is GITLAB | | 
| **GITLAB_NAMESPACE_NAME** | if GIT_PROVIDER is GITLAB | |
| **GIT_USERNAME** | git username for the GIT_PROVIDER  (required) | |
| **GITHUB_ORGANIZATION** | if GIT_PROVIDER is GITHUB | |
| **DEFAULT_CD_LOGS_BUCKET_REGION** | AWS region of bucket to store CD logs (required) | |
| **DEFAULT_CACHE_BUCKET** | AWS bucket to store docker cache (required) |  |
| **DEFAULT_CACHE_BUCKET_REGION** | AWS region of cache bucket defined in previous step (required) | |
| **DEFAULT_BUILD_LOGS_BUCKET** | AWS bucket to store build logs (required) | |
| **CHARTMUSEUM_STORAGE_AMAZON_BUCKET** | AWS bucket to store charts (required) |  |
| **CHARTMUSEUM_STORAGE_AMAZON_REGION** | AWS region for bucket defined in previous step to store charts (required) | |
| **EXTERNAL_SECRET_AMAZON_REGION** | AWS region for secret manager to pick (required) |  |
| **PROMETHEUS_URL** | url of prometheus where all cluster data is stored, if this is wrong, you will not be able to see application metrics like cpu, ram, http status code, latency and throughput (required) |  |

example of DEX_CONFIG is

    DEX_CONFIG: |-
      connectors:
        - type: oidc
          id: google
          name: Google
          config:
            issuer: https://accounts.google.com
            clientID: xxxxxxxx-qwwdsdsqsxxxxxxxxx.apps.googleusercontent.com
            clientSecret: fssdsdw121wwxssd
            redirectURI: <BASE_URL_SCHEME>://<BASE_URL>/api/dex/callback
            hostedDomains:
            - abc.com

**Please Note:**
Ensure that the cluster has access to the DEFAULT_CACHE_BUCKET, DEFAULT_BUILD_LOGS_BUCKET, CHARTMUSEUM_STORAGE_AMAZON_BUCKET and AWS secrets backends (SSM & secrets manager)

### Cleanup
Run following commands to delete all the components installed by devtron
```bash
$ cd devtron-installation-script/
$ kubectl delete -n devtroncd -f yamls/
$ kubectl delete -n devtrocd -f charts/devtron/templates/devtron-installer.yaml 
$ kubectl delete -n devtrocd -f charts/devtron/templates/install.yaml
$ kubectl delete ns devtroncd
```
### Trouble shooting steps

 1. How do I know when installation is complete?
     Run following command
   ```bash
$ kubectl -n devtroncd get installers installer-devtron -o jsonpath='{.status.sync.status}'
```  
Once installation process is complete, status will becomes `Applied` 
It may take around 30 mins for installation to complete
 2. How do I track progress of installation?
     Run following command to check logs of the pod
 ```bash
$ pod=$(kubectl -n devtroncd get po -l app=inception -o jsonpath='{.items[0].metadata.name}')&& kubectl -n devtroncd logs -f $pod
```
 3. devtron installer logs have error and I want to restart installation.
     Run following command to clean up components installed by devtron installer
 ```bash
$ cd devtron-installation-script/
$ kubectl delete -n devtroncd -f yamls/
$ kubectl -n devtroncd patch installer installer-devtron --type json -p '[{"op": "remove", "path": "/status"}]'
```
 
 In case you are still facing issues please feel free to reach out to us on [discord](https://discord.gg/72JDKy4)