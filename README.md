# k8s_om

**This documentation is a WIP**

## Table of Contents
- [k8s_om](#k8s_om)
  - [Table of Contents](#table-of-contents)
  - [Description](#description)
  - [Steps to Deploy](#steps-to-deploy)
  - [Prerequisites](#prerequisites)
    - [CA Certificate for Ops Manager _REQUIRED_](#ca-certificate-for-ops-manager-required)
    - [CA Certificate for MongoDB Deployments _HIGHLY ENCOURAGED_](#ca-certificate-for-mongodb-deployments-highly-encouraged)
    - [TLS PEM Files for MongoDB Deployments _HIGHLY ENCOURAGED_](#tls-pem-files-for-mongodb-deployments-highly-encouraged)
    - [MongoDB First User _REQUIRED_](#mongodb-first-user-required)
  - [Set Up](#set-up)
- [Notes](#notes)

## Description

A series of Helm Charts to deploy MongoDB Ops Manager within Kubernetes with the MongoDB Kubernetes Operator.

## Steps to Deploy

1. Ensure [Prerequisites](#prerequisites) are met
2. Create [Ops Manager Access Token](#ops-manager-api-access-token-required) (Progammatic Access)
3. Create Kubernetes `configmap` for [Ops Manager X.509 Certificate Authority (CA) certificate chain](#ca-certificate-for-ops-manager-required)
4. Create Kubernetes `configmap` for [MongoDB deployments CA certificate chain](#ca-certificate-for-mongodb-deployments-highly-encouraged) - if requires - and seriously, this should just be a normal thing
5. Create Kubernets secrets for the [MonogDB instances TLS and cluster authentication](#tls-pem-files-for-mongodb-deployments-highly-encouraged) - once again this is "if requires", but should be just a normal thing.....look at your life choices if you are not doing this!
6. Create a Kubernetes secret for the [`root`](mongodb-first-user-required) user of the MongoDB deployment
7. Create the `values.yaml` file for the deployment.


## Prerequisites

The [MongoDB Enterprise Kubernetes Operator](https://docs.mongodb.com/kubernetes-operator/master/) must be installed and operation. Instructions on installing the MongoDB Kubernetes Operator can be found in the MongoDB [documentation](https://docs.mongodb.com/kubernetes-operator/master/installation/). It is highly recommended that MongoDB Professional Services be engaged to deploy and configure Ops Manager to ensure the deployment is secure and reliable.

[Helm](https://helm.sh/docs/intro/install/) is required to be installed and [Helmfile](https://github.com/roboll/helmfile) is also highly recommended. If Helmfile is used you will also need [Helm-Diff](https://github.com/databus23/helm-diff).

### CA Certificate for Ops Manager _REQUIRED_

This is **REQUIRED** because your Ops Manager should be using TLS!

The certificate must include the whole certificate chain of the Certificate Authority that signed the X.509 certificate for Ops Manager.

This can be a common configmap if more than one deployment is in a Kubernetes namespace and Ops Manager Organisation.

This is stored in a configMap is set in the relevant `values.yaml` as `opsManager.caConfigMap`. The name of the key in the configmap **MUST** be `mms-ca.crt`, this can be created via:

```shell
kubectl --kubeconfig=<CONFIG_FILE> -n <NAMESPACE> create configmap <name-of-configmap> \
  --from-file=mms-ca.crt
```

This is most likely common in all MongoDB deployments.

### CA Certificate for MongoDB Deployments _HIGHLY ENCOURAGED_

The certificate must include the whole certificate chain of the Certificate Authority that signed the X.509 certificate for pods. 

This is stored in a configMap is set in the relevant values.yaml as tls.caConfigMap. The name of the key in the configmap **MUST** be `ca-pem`, this can be created via:

```shell
kubectl --kubeconfig=<CONFIG_FILE> -n <NAMESPACE> create configmap <name-of-configmap> \
  --from-file=ca-pem
```

This is most likely common in all MongoDB deployments.

**REQUIRED** if `tls.enabled` is `true`.

### TLS PEM Files for MongoDB Deployments _HIGHLY ENCOURAGED_

This requires two secrets: one for the client communications and one for cluster communications.

The secrets contain the X.509 key and certificate. One key/certificate pair is used for all members of the replica set, therefore a Subject Alternate Name (SAN) entry must exist for each member of the replica set. The SANs will be in the form of:

**\<replicaSetName\>-\<X\>.\<replicaSetName\>.\<namespace\>**

Where `<replicaSetName>` is the `replicaSetName` in the `values.yaml` for your deployment and `<X>` is the 0-based number of the pod.

The certificates must include the name of FQDN external to Kubernetes as a Subject Alternate Name (SAN) if external access is required. 

The secrets must be named as follows:

**\<replicaSetName\>-\<cert\>**

**\<replicaSetName\>-\<clusterfile\>**

The two secrets can be created as follows:

```shell
kubectl --kubeconfig=<CONFIG_FILE> -n <NAMESPACE> create secret tls <replicaSetName>-cert \
  --cert=<path-to-cert> \
  --key=<path-to-key>

kubectl --kubeconfig=<CONFIG_FILE> -n <NAMESPACE> create secret tls <replicaSetName>-clusterfile \
  --cert=<path-to-cert> \
  --key=<path-to-key>
```

**REQUIRED** if `tls.enabled` is `true`.

### MongoDB First User _REQUIRED_

A secret must exist for the first user in MongoDB. This will be a user with the `root` role. The name of the secret must be set in the releveant `values.yaml` as `rootSecret` value. The secret must contain a key called `password` that contains the password for the user. The username is set to `root`.

The secret can be create via `kubectl` as follows:

```shell
kubectl --kubeconfig=<CONFIG_FILE> -n <NAMESPACE> create secret generic <name-of-secret> \
  --from-literal=password=<password>
```

The name of the user that is created has the pattern of **ap-\<replicaSetName\>-root**, where `<replicaSetName>` is the `replicaSetName` in the `values.yaml` for your deployment.

## Set Up

Two environment variables are required, called ENV and NS (both case senstive). The first describes the selected Git environment for deployment and the second describes the Kubernetes namespace.

The variables for each deployment are contained in the `values.yaml`. The `values.yaml` file for the selected environment must reside in a directory under `charts/values/<ENV>` such as `charts/values/dev/values.yaml` or `charts/values/production/values.yaml`. Each **\<ENV\>** directory will be a different deployment. The `examples` directory contains an examples `values.yaml` file, plus there are examples under the `charts/values` directory so the reader can see the structure.

The following table describes the values required in the relevant `values.yaml`:

|Key|Purpose|
|--------------------------|------------------------------------|
|deploymentName|The name for the Ops Manager deployment (becomes the Kubernetes resource name)|
|opsManager.omVersion|Version of Ops Manager to use|
|opsManager.replicas|Number of Ops Manager application servers to deploy|
|opsManager.adminUserSecret|The name of the Kubernetes secret containing the first user's password for Ops Manager|
|opsManager.binarySource|The source for the MongoDB binaries. Can be `local`, `hybrid`, or `remote`|
|opsManager.tlsSecretName|The Kubernetes TLS secret containing the X.509 certificate for Ops Manager application servers|
|opsManager.emailServerHostname|The email server FQDN|
|opsManager.adminEmailAddress|The email address of the Admin user for Ops Manager|
|opsManager.fromEmailAddress|The fram email address used by Ops Manager when sending emails|
|opsManager.replyEmailAddress|The email address used as the `reply-to` address for emails sent by Ops Manager|
|opsManager.emailPort|The port used by the email server|
|opsManager.emailSecure|Boolean to determine if TLS is used for email communications|
|opsManager.emailProtocol|The protocol used by the email server. `smtp` or `smtps`|
|opsManager.podLimitCPU|The maximum CPUs that can be allocated to the Ops Manager application container|
|opsManager.podRequestsCPU|The initial CPUs allocated to the Ops Manager application application container|
|opsManager.podLimitMemory|The maximum memory that can be allocated to the Ops Manager application container|
|opsManager.podRequestsMemory|The initial memory allocated to the Ops Manager application container|
|opsManager.initPodLimitCPU|The maximum CPUs that can be allocated to the Ops Manager init container|
|opsManager.initPodRequestsCPU|The initial CPUs allocated to the Ops Manager application init container|
|opsManager.initPodLimitMemory|The maximum memory that can be allocated to the Ops Manager init container|
|opsManager.initPodRequestsMemory|The initial memory allocated to the Ops Manager init container|
|opsManager.localBinariesMountEnabled|Boolean to determine if a PVC is created to store the MongoDB binaries. Required if `opsManager.binarySource` is `local`|
|opsManager.extServiceEnabled|Boolean to determine if access to Ops Manager from clients/users external to Kubernetes is required|
|opsManager.extServiceType|The service type created for external access. Selection is `NodePort` or `LoadBalancer`|
|opsManager.extServicePort|
|opsManager.extCentralUrl|
|backups.enabled|Boolean to determine if the backup system for Ops Manager is enabled|
|backups.members|Number of backup daemons to create. One is normally enough|
|backups.head.binariesMountEnabled|
|backups.head.binariesStorageClass|
|backups.head.binariesStorageSize||
|backups.head.storageClass||
|backups.head.storageSize||
|backups.head.podLimitCPU||
|backups.head.podRequestsCPU||
|backups.head.podLimitMemory||
|backups.head.podRequestsMemory||
|backups.head.initPodLimitCPU||
|backups.head.initPodRequestsCPU||
|backups.head.initPodLimitMemory||
|backups.head.initPodRequestsMemory||
|backups.oplogStores.stores[]
|backups.oplogStores.stores[].name||
|backups.oplogStores.stores[].mdbResource||
|backups.oplogStores.stores[].userResource||
|backups.blockstores.stores[]
|backups.blockstores.stores[]name||
|backups.blockstores.stores[]mdbResource||
|backups.blockstores.stores[]userResource||
|backups.s3stores[]||
|backups.s3stores[].name||
|backups.s3stores[].mdbResource||
|backups.s3stores[].userResource||
|backups.s3stores[].s3secret||
|backups.s3stores[].pathStyleEnabled||
|backups.s3stores[].bucketEndpoint||
|backups.s3stores[].bucketName||
|backups.appDB.replicas||
|backups.appDB.mdbVersion||
|backups.appDB.adminSecretName||
|backups.appDB.tlsSecretName||
|backups.appDB.CAConfigmapName||
|backups.appDB.podLimitCPU||
|backups.appDB.podLimitMemory||
|backups.appDB.storageClass||
|backups.appDB.storageSize||

# Notes
OM Admin user secret
OM TLS secret
APPDB TLS secret