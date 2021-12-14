# k8s_om

**This documentation is a WIP**

## Table of Contents
- [k8s_om](#k8s_om)
  - [Table of Contents](#table-of-contents)
  - [Description](#description)
  - [Steps to Deploy](#steps-to-deploy)
  - [Prerequisites](#prerequisites)
    - [CA Certificate for Ops Manager _REQUIRED_](#ca-certificate-for-ops-manager-required)
    - [TLS Secret for Ops Manager Application Servers _HIGHLY ENCOURAGED_](#tls-secret-for-ops-manager-application-servers-highly-encouraged)
    - [CA Certificate for MongoDB Deployments _HIGHLY ENCOURAGED_](#ca-certificate-for-mongodb-deployments-highly-encouraged)
    - [TLS Secrets for MongoDB Deployments _HIGHLY ENCOURAGED_](#tls-secrets-for-mongodb-deployments-highly-encouraged)
    - [Ops Manager First User _REQUIRED_](#ops-manager-first-user-required)
    - [MongoDB First User _REQUIRED_](#mongodb-first-user-required)
  - [Set Up](#set-up)
    - [deploymentName](#deploymentname)
    - [opsManager.omVersion](#opsmanageromversion)
    - [opsManager.replicas:](#opsmanagerreplicas)
    - [opsManager.adminUserSecret:](#opsmanageradminusersecret)
    - [opsManager.binarySource:](#opsmanagerbinarysource)
    - [opsManager.tlsSecretName](#opsmanagertlssecretname)
    - [opsManager.emailServerHostname](#opsmanageremailserverhostname)
    - [opsManager.adminEmailAddress](#opsmanageradminemailaddress)
    - [opsManager.fromEmailAddress](#opsmanagerfromemailaddress)
    - [opsManager.replyEmailAddress:](#opsmanagerreplyemailaddress)
    - [opsManager.emailPort: 465](#opsmanageremailport-465)
    - [opsManager.emailSecure: true](#opsmanageremailsecure-true)
    - [opsManager.emailProtocol: smtp](#opsmanageremailprotocol-smtp)
    - [opsManager.podLimitCPU: 4](#opsmanagerpodlimitcpu-4)
    - [opsManager.podRequestsCPU: 2](#opsmanagerpodrequestscpu-2)
    - [opsManager.podLimitMemory: 8Gi](#opsmanagerpodlimitmemory-8gi)
    - [opsManager.podRequestsMemory: 4Gi](#opsmanagerpodrequestsmemory-4gi)
    - [opsManager.initPodLimitCPU: 2](#opsmanagerinitpodlimitcpu-2)
    - [opsManager.initPodRequestsCPU: 1](#opsmanagerinitpodrequestscpu-1)
    - [opsManager.initPodLimitMemory: 2Gi](#opsmanagerinitpodlimitmemory-2gi)
    - [opsManager.initPodRequestsMemory: 1G](#opsmanagerinitpodrequestsmemory-1g)
    - [opsManager.localBinariesMountEnabled](#opsmanagerlocalbinariesmountenabled)
    - [opsManager.extServiceEnabled: true](#opsmanagerextserviceenabled-true)
    - [opsManager.extServiceType: NodePort](#opsmanagerextservicetype-nodeport)
    - [opsManager.extServicePort: 32111](#opsmanagerextserviceport-32111)
    - [opsManager.extCentralUrl: mongod3.mo](#opsmanagerextcentralurl-mongod3mo)
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

### TLS Secret for Ops Manager Application Servers _HIGHLY ENCOURAGED_

This requires one Kubernetes TLS secret.

The secrets contain the X.509 key and certificate. A Subject Alternate Name (SAN) entry must exist for the service.

The certificate must include the name of FQDN external to Kubernetes as a Subject Alternate Name (SAN) if external access is required. 

The secret can be created as follows:

```shell
kubectl --kubeconfig=<CONFIG_FILE> -n <NAMESPACE> create secret tls <deploymentName>-cert \
  --cert=<path-to-cert> \
  --key=<path-to-key>
```

**REQUIRED** if `tls.enabled` is `true`.

### CA Certificate for MongoDB Deployments _HIGHLY ENCOURAGED_

The certificate must include the whole certificate chain of the Certificate Authority that signed the X.509 certificate for pods. 

This is stored in a configMap is set in the relevant values.yaml as tls.caConfigMap. The name of the key in the configmap **MUST** be `ca-pem`, this can be created via:

```shell
kubectl --kubeconfig=<CONFIG_FILE> -n <NAMESPACE> create configmap <name-of-configmap> \
  --from-file=ca-pem
```

This is most likely common in all MongoDB deployments.

**REQUIRED** if `tls.enabled` is `true`.

### TLS Secrets for MongoDB Deployments _HIGHLY ENCOURAGED_

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

### Ops Manager First User _REQUIRED_

```shell
kubectl --kubeconfig=<CONFIG_FILE> -n <NAMESPACE> create secret generic <adminusercredentials> \
  --from-literal=Username="<username>" \
  --from-literal=Password="<password>" \
  --from-literal=FirstName="<firstname>" \
  --from-literal=LastName="<lastname>"
```

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
|opsManager.extServicePort|The Kubernetes port number to use for the NodePort, if `NodePort` is selected for `opsManager.extServiceType`|
|opsManager.extCentralUrl|The URL, including port, that will be used by clients external to Kubernetes for the Ops Manager service|
|backups.enabled|Boolean to determine if the backup system for Ops Manager is enabled|
|backups.members|Number of backup daemons to create. One is normally enough|
|backups.head.binariesMountEnabled|Boolean to determine if a PVC is created to store the MongoDB binaries. Required if `opsManager.binarySource` is `local`|
|backups.head.binariesStorageClass|The Kubernetes StorageClass to be used for the binaries mount|
|backups.head.binariesStorageSize|The size, including units, for the binaries mount|
|backups.head.storageClass|The Kubernetes StorageClass for the Head database. Required if `backups.enabled` is `true`|
|backups.head.storageSize|The size, including units, for the Head database storage|
|backups.head.podLimitCPU|The maximum CPUs that can be allocated to the Head database container|
|backups.head.podRequestsCPU|The initial CPUs allocated to the Head database container|
|backups.head.podLimitMemory|The maximum memory that can be allocated to the Head database container|
|backups.head.podRequestsMemory|The initial memory allocated to the Head database container|
|backups.head.initPodLimitCPU|The maximum CPUs that can be allocated to the Head init container|
|backups.head.initPodRequestsCPU|The initial CPUs allocated to the Head init container|
|backups.head.initPodLimitMemory|The maximum memory that can be allocated to the Head init container|
|backups.head.initPodRequestsMemory|The initial memory allocated to the Head init container|
|backups.oplogStores.stores[]|An array of objects containing details on required Oplog Stores|
|backups.oplogStores.stores[].name|The name to be used for the Oplog Store configuration within Ops Manager|
|backups.oplogStores.stores[].mdbResource|The Kubernetes resource name of the Opolog Store MongoDB replica set (must be created in advance)|
|backups.oplogStores.stores[].userResource|The Kubernetes resource name of the Opolog Store MongoDB User (must be created in advance)|
|backups.blockstores.stores[]|An array of objects containing details on required Block Stores|
|backups.blockstores.stores[].name|The name to be used for the Block Store configuration within Ops Manager|
|backups.blockstores.stores[].mdbResource|The Kubernetes resource name of the Block Store MongoDB replica set (must be created in advance)|
|backups.blockstores.stores[].userResource|The Kubernetes resource name of the Block Store MongoDB User (must be created in advance)|
|backups.s3stores[]|An array if objects containing details of S3 stores for backups|
|backups.s3stores[].name|The name given to the S3 store in Ops Manager|
|backups.s3stores[].mdbResource|The Kubernetes resource name of the MongoDB replica set used for metadata (must be created in advance)|
|backups.s3stores[].userResource|The Kubernetes resource name of the MongoDB User for the metadata database replica set (must be created in advance)|
|backups.s3stores[].s3secret|The Kubernetes Secert resource name that contains the access AWS access key and username|
|backups.s3stores[].pathStyleEnabled|Boolean to determine of path style is used (true) or virtual-host-style (false) URL endpoint are used|
|backups.s3stores[].bucketEndpoint|The URL of the S3 bucket endpoint|
|backups.s3stores[].bucketName|The name for the S3 bucket|
|appDB.replicas|Numer of members in the AppDB replica set. Should be 3 at the minimum|
|appDB.mdbVersion|The version of MongoDB to use for the AppDB. Must match a tag in the `quay.io` registry|
|appDB.adminSecretName|The name of the Kubernetes secret containing the admin user's password|
|appDB.tlsSecretName|The name of the Kubernetes TLS secret containing the X.509 key and certificate for the AppDB|
|appDB.CAConfigmapName|The Kubernetes configmap name containing the CA certificate for the TLS certificates|
|appDB.podLimitCPU|The maximum memory that can be allocated to the AppDB database container|
|appDB.podRequestsCPU|The initial CPUs allocated to the AppDB database container|
|appDB.podLimitMemory|The maximum memory that can be allocated to the AppDB database container|
|appDB.podRequestsMemory|The initial memory allocated to the AppDB database container|
|appDB.initPodLimitCPU|The maximum memory that can be allocated to the AppDB init database container|
|appDB.initPodRequestsCPU|The initial CPUs allocated to the AppDB init database container|
|appDB.initPodLimitMemory|The maximum memory that can be allocated to the AppDB init database container|
|appDB.initPodRequestsMemory|The initial memory allocated to the AppDB database init container|
|appDB.storageClass|The Kubernetes StorageClass name that will be used for VPCs for the AppDB instances|
|appDB.storageSize|The amount of storage to allocate for the AppDB mongod instances|

### deploymentName

The name that will be used for the Ops Manager instance. Must be unique in the namespace.

### opsManager.omVersion

The version of Ops Manager to use.

### opsManager.replicas:

The number of Ops Manager application servers to deploy. For high availability, this should be `2` or higher.

### opsManager.adminUserSecret:

The name of the Kubernetes secret that holds the credentials for the first user in Ops Manager.

The secret must consist of the following keys:

* Username
* Password
* FirstName
* LastName

See the prerequisites on [Ops Manager First User](#ops-manager-first-user-required) for further details.

### opsManager.binarySource:

Setting to determine if the MongoDB Automation Agents retrieve the mongod binaries from the Internet sources or from Ops Manager. The setting also determines if the Ops Manager retrieves the binaries from the Internet or is air-gapped.

Options:
* `remote` Automation Agent gets the binaries from MognoDB on the Internet
* `hybrid` Automation Agent have no Internet access and will retieve the binaries from Ops Manager, but Ops Manager retrieves the binaries via the Internet
* `remote` Automation Agent have no Internet access and will retieve the binaries from Ops Manager, but Ops Manager does *NOT* have Internet access and therefore the binaries are loaded onto the application servers (and backup daemons) manually. If this is selected also set `opsManager.localBinariesMountEnabled` to `true` so storage can be allocated for the binaries.

### opsManager.tlsSecretName

The name of the Kubernetes TLS secret that contains the TLS X.509 key and certificate for the Ops Manager application servers.

The certificates need to include a SAN of the LoadBalancer or NodePort service if access is required from external to Kubernetes. See the prerequisites on [TLS Secret for Ops Manager Application Servers](#tls-secret-for-ops-manager-application-servers-highly-encouraged).

### opsManager.emailServerHostname

The Fully Qualified Domain Name (FQDN) of the email server for alerting and invitation purposes.

### opsManager.adminEmailAddress

The email address for the admin user of Ops Manager.

### opsManager.fromEmailAddress

The email address that will be used as the "from address" for emails sent by Ops Manager.

### opsManager.replyEmailAddress:

The email address used as the "reply address" for emails sent by Ops Manager.

### opsManager.emailPort: 465
### opsManager.emailSecure: true
### opsManager.emailProtocol: smtp
### opsManager.podLimitCPU: 4
### opsManager.podRequestsCPU: 2
### opsManager.podLimitMemory: 8Gi
### opsManager.podRequestsMemory: 4Gi
### opsManager.initPodLimitCPU: 2
### opsManager.initPodRequestsCPU: 1
### opsManager.initPodLimitMemory: 2Gi
### opsManager.initPodRequestsMemory: 1G
### opsManager.localBinariesMountEnabled
### opsManager.extServiceEnabled: true
### opsManager.extServiceType: NodePort
### opsManager.extServicePort: 32111
### opsManager.extCentralUrl: mongod3.mo

backups:
  enabled: false
  members: 2
  head:
    binariesMountEnabled: f
    binariesStorageClass: o
    binariesStorageSize: 5G
    storageClass: ops-manag
    storageSize: 4Gi
    podLimitCPU: 2
    podRequestsCPU: 1
    podLimitMemory: 2Gi
    podRequestsMemory: 1Gi
    initPodLimitCPU: 2
    initPodRequestsCPU: 1
    initPodLimitMemory: 2Gi
    initPodRequestsMemory: 
  oplogStores:
    stores:
      - name: oplog0
        mdbResource: oplog0
        userResource: oplog
  blockstores:
    stores:
      - name: blockstore0
        mdbResource: blocks
        userResource: block
  s3stores:

appDB:
  replicas: 3
  mdbVersion: 5.0.1-ent
  adminSecretName: om-db-us
  tlsSecretName: ops-manage
  CAConfigmapName: custom-c
  podLimitCPU: 2
  podRequestsCPU: 2
  podLimitMemory: 2Gi
  podRequestsMemory: 2Gi
  storageClass: appsdb
  storageSize: 10Gi 

# Notes
OM Admin user secret
OM TLS secret
APPDB TLS secret