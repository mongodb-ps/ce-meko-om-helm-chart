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
    - [CA Certificate for AppDB _HIGHLY ENCOURAGED_](#ca-certificate-for-appdb-highly-encouraged)
    - [TLS Secrets for AppDB _HIGHLY ENCOURAGED_](#tls-secrets-for-appdb-highly-encouraged)
    - [Ops Manager First User _REQUIRED_](#ops-manager-first-user-required)
    - [MongoDB Admin User _REQUIRED_](#mongodb-admin-user-required)
    - [Ops Manager Backup System](#ops-manager-backup-system)
    - [S3 snapshot store](#s3-snapshot-store)
  - [Set Up](#set-up)
    - [deploymentName](#deploymentname)
    - [opsManager.omVersion](#opsmanageromversion)
    - [opsManager.replicas:](#opsmanagerreplicas)
    - [opsManager.adminUserSecret:](#opsmanageradminusersecret)
    - [opsManager.binarySource](#opsmanagerbinarysource)
    - [opsManager.tlsSecretName](#opsmanagertlssecretname)
    - [opsManager.emailServerHostname](#opsmanageremailserverhostname)
    - [opsManager.adminEmailAddress](#opsmanageradminemailaddress)
    - [opsManager.fromEmailAddress](#opsmanagerfromemailaddress)
    - [opsManager.replyEmailAddress](#opsmanagerreplyemailaddress)
    - [opsManager.emailPort](#opsmanageremailport)
    - [opsManager.emailSecure](#opsmanageremailsecure)
    - [opsManager.emailProtocol](#opsmanageremailprotocol)
    - [opsManager.podLimitCPU](#opsmanagerpodlimitcpu)
    - [opsManager.podRequestsCPU](#opsmanagerpodrequestscpu)
    - [opsManager.podLimitMemory](#opsmanagerpodlimitmemory)
    - [opsManager.podRequestsMemory](#opsmanagerpodrequestsmemory)
    - [opsManager.initPodLimitCPU](#opsmanagerinitpodlimitcpu)
    - [opsManager.initPodRequestsCPU](#opsmanagerinitpodrequestscpu)
    - [opsManager.initPodLimitMemory](#opsmanagerinitpodlimitmemory)
    - [opsManager.initPodRequestsMemory](#opsmanagerinitpodrequestsmemory)
    - [opsManager.localBinariesMountEnabled](#opsmanagerlocalbinariesmountenabled)
    - [opsManager.localBinariesMountStorageClass](#opsmanagerlocalbinariesmountstorageclass)
    - [opsManager.localBinariesMountStorageSize](#opsmanagerlocalbinariesmountstoragesize)
    - [opsManager.extServiceEnabled](#opsmanagerextserviceenabled)
    - [opsManager.extServiceType](#opsmanagerextservicetype)
    - [opsManager.centralUrl](#opsmanagercentralurl)
    - [opsManager.extServicePort](#opsmanagerextserviceport)
    - [appDB.replicas](#appdbreplicas)
    - [appDB.mdbVersion](#appdbmdbversion)
    - [appDB.mongoDBAdminPasswdSecretName](#appdbmongoDBAdminPasswdSecretname)
    - [appDB.tlsSecretName](#appdbtlssecretname)
    - [appDB.CAConfigmapName](#appdbcaconfigmapname)
    - [appDB.podLimitCPU](#appdbpodlimitcpu)
    - [appDB.podRequestsCPU](#appdbpodrequestscpu)
    - [appDB.podLimitMemory](#appdbpodlimitmemory)
    - [appDB.podRequestsMemory](#appdbpodrequestsmemory)
    - [appDB.initPodLimitCPU](#appdbinitpodlimitcpu)
    - [appDB.initPodRequestsCPU](#appdbinitpodrequestscpu)
    - [appDB.initPodLimitMemory](#appdbinitpodlimitmemory)
    - [appDB.initPodRequestsMemory](#appdbinitpodrequestsmemory)
    - [appDB.storageClass](#appdbstorageclass)
    - [appDB.storageSize](#appdbstoragesize)
    - [backups.enabled](#backupsenabled)
    - [backups.members](#backupsmembers)
    - [backups.head.binariesMountEnabled](#backupsheadbinariesmountenabled)
    - [backups.head.binariesStorageClass](#backupsheadbinariesstorageclass)
    - [backups.head.binariesStorageSize](#backupsheadbinariesstoragesize)
    - [backups.head.storageClass](#backupsheadstorageclass)
    - [backups.head.storageSize](#backupsheadstoragesize)
    - [backups.head.podLimitCPU](#backupsheadpodlimitcpu)
    - [backups.head.podRequestsCPU](#backupsheadpodrequestscpu)
    - [backups.head.podLimitMemory](#backupsheadpodlimitmemory)
    - [backups.head.podRequestsMemory](#backupsheadpodrequestsmemory)
    - [backups.head.initPodLimitCPU](#backupsheadinitpodlimitcpu)
    - [backups.head.initPodRequestsCPU](#backupsheadinitpodrequestscpu)
    - [backups.head.initPodLimitMemory](#backupsheadinitpodlimitmemory)
    - [backups.head.initPodRequestsMemory](#backupsheadinitpodrequestsmemory)
    - [backups.oplogStores[]](#backupsoplogstores)
    - [backups.oplogStores.name](#backupsoplogstoresname)
    - [backups.oplogStores.mdbResource](#backupsoplogstoresmdbresource)
    - [backups.oplogStores.userResource](#backupsoplogstoresuserresource)
    - [backups.blockstores[]](#backupsblockstores)
    - [backups.blockstores[].name](#backupsblockstoresname)
    - [backups.blockstores[].mdbResource](#backupsblockstoresmdbresource)
    - [backups.blockstores[].userResource](#backupsblockstoresuserresource)
    - [backups.s3stores[]](#backupss3stores)
    - [backups.s3stores[].name](#backupss3storesname)
    - [backups.s3stores[].mdbResource](#backupss3storesmdbresource)
    - [backups.s3stores[].userResource](#backupss3storesuserresource)
    - [backups.s3stores[].s3secret](#backupss3storess3secret)
    - [backups.s3stores[].pathStyleEnabled](#backupss3storespathstyleenabled)
    - [backups.s3stores[].bucketEndpoint](#backupss3storesbucketendpoint)
    - [backups.s3stores[].bucketName](#backupss3storesbucketname)
  - [Run](#run)

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

### CA Certificate for AppDB _HIGHLY ENCOURAGED_

The certificate must include the whole certificate chain of the Certificate Authority that signed the X.509 certificate for pods. 

This is stored in a configMap is set in the relevant values.yaml as tls.caConfigMap. The name of the key in the configmap **MUST** be `ca-pem`, this can be created via:

```shell
kubectl --kubeconfig=<CONFIG_FILE> -n <NAMESPACE> create configmap <name-of-configmap> \
  --from-file=ca-pem
```

This is most likely common in all MongoDB deployments.

**REQUIRED** if `tls.enabled` is `true`.

### TLS Secrets for AppDB _HIGHLY ENCOURAGED_

This requires a Kubernetes TLS secert in a certain format.

The secrets contain the X.509 key and certificate. One key/certificate pair is used for all members of the replica set, therefore a Subject Alternate Name (SAN) entry must exist for each member of the replica set. The SANs will be in the form of:

**\<deploymentName\>-db-\<X\>.\<replicaSetName\>.\<namespace\>**

Where `<deploymentName>` is the `deploymentName` in the `values.yaml` for your deployment and `<X>` is the 0-based number of the pod.

The secret must be named as follows:

**\<deploymentName\>-appdb-\<cert\>**

The secret can be created as follows:

```shell
kubectl --kubeconfig=<CONFIG_FILE> -n <NAMESPACE> create secret tls <deploymentName>-appdb-cert \
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

### MongoDB Admin User _REQUIRED_

A secret must exist for the admin user of the AppDB. The name of the secret must be set in the releveant `values.yaml` as `mongoDBAdminPasswdSecretName` value. The secret must contain a key called `password` that contains the password for the user. The username is set to `mongodb-ops-manager` by the Kubernetes Operator.

The secret can be create via `kubectl` as follows:

```shell
kubectl --kubeconfig=<CONFIG_FILE> -n <NAMESPACE> create secret generic <name-of-secret> \
  --from-literal=password=<password>
```

### Ops Manager Backup System

To enable the backup system within Ops Manager set the `backups.enabled` field to `true`. This will create a backup daemon, as well as a head database (which is no longer used in more modern MongoDB deployments). At least one of either a Blockstore or S3store must be selected as the snapshot store, and an oplog store is also required. The Blockstore and Oplog store are both MongoDB replica sets and need to be configured and deployed with the `k8s_mdb` Helm charts. This presents a chicken and egg situation, so Ops Manager should initially be deployed without backups enabled. The Blockstore(s) (if using Blockstores) and the Oplog store should be deployed, then the backup system should be enabled for Ops Manager. All options, including the Oplog store, require a MongoDB user that can be configured when deploying the actual MognoDB instance via the `k8s_mdb` Helm charts.

### S3 snapshot store

If the backup system is enabled, there is the choice to use S3-compatible devices as a snapshot store. This option requires another secret for the S3 credentials (accessKey and secretKey for the S3 endpoint).

The secret can be created as follows:

```shell
kubectl --kubeconfig=<CONFIG_FILE> -n <NAMESPACE> create secret generic <name-of-secret> \
  --from-literal=accessKey="<AKIAIOSFODNN7EXAMPLE>" \
  --from-literal=secretKey="<wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY>"
```

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
|opsManager.localBinariesMountStorageClass|The name of the Kubernetes StorageClass that will be used as the local storage for MongoDB binaries, if required|
|opsManager.localBinariesMountStorageSize|The size of the storage to be allocated for the binaries if required|
|opsManager.extServiceEnabled|Boolean to determine if access to Ops Manager from clients/users external to Kubernetes is required|
|opsManager.extServiceType|The service type created for external access. Selection is `NodePort` or `LoadBalancer`|
|opsManager.extServicePort|The Kubernetes port number to use for the NodePort, if `NodePort` is selected for `opsManager.extServiceType`|
|opsManager.extCentralUrl|The URL, including port, that will be used by clients external to Kubernetes for the Ops Manager service|
|appDB.replicas|Numer of members in the AppDB replica set. Should be 3 at the minimum|
|appDB.mdbVersion|The version of MongoDB to use for the AppDB. Must match a tag in the `quay.io` registry|
|appDB.mongoDBAdminPasswdSecretName|The name of the Kubernetes secret containing the admin user's password|
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
|backups.oplogStores[]|An array of objects containing details on required Oplog Stores|
|backups.oplogStores[].name|The name to be used for the Oplog Store configuration within Ops Manager|
|backups.oplogStores[].mdbResource|The Kubernetes resource name of the Opolog Store MongoDB replica set (must be created in advance)|
|backups.oplogStores[].userResource|The Kubernetes resource name of the Opolog Store MongoDB User (must be created in advance)|
|backups.blockstores[]|An array of objects containing details on required Block Stores|
|backups.blockstores[].name|The name to be used for the Block Store configuration within Ops Manager|
|backups.blockstores[].mdbResource|The Kubernetes resource name of the Block Store MongoDB replica set (must be created in advance)|
|backups.blockstores[].userResource|The Kubernetes resource name of the Block Store MongoDB User (must be created in advance)|
|backups.s3stores[]|An array if objects containing details of S3 stores for backups|
|backups.s3stores[].name|The name given to the S3 store in Ops Manager|
|backups.s3stores[].mdbResource|The Kubernetes resource name of the MongoDB replica set used for metadata (must be created in advance)|
|backups.s3stores[].userResource|The Kubernetes resource name of the MongoDB User for the metadata database replica set (must be created in advance)|
|backups.s3stores[].s3secret|The Kubernetes Secert resource name that contains the access AWS access key and username|
|backups.s3stores[].pathStyleEnabled|Boolean to determine of path style is used (true) or virtual-host-style (false) URL endpoint are used|
|backups.s3stores[].bucketEndpoint|The URL of the S3 bucket endpoint|
|backups.s3stores[].bucketName|The name for the S3 bucket|

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

### opsManager.binarySource

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

### opsManager.replyEmailAddress

The email address used as the "reply address" for emails sent by Ops Manager.

### opsManager.emailPort

The port, as an integer, that the email server uses for communications.

### opsManager.emailSecure

A boolean value to determine if TLS is used with email communications.

### opsManager.emailProtocol

The protocol to use with the email server. Choices are `smtp` or `smtps`.

### opsManager.podLimitCPU

The CPU limit that can be assigned to each Ops Manager Application Server pod.

### opsManager.podRequestsCPU

The initial CPUs assigned to each Ops Manager Application Server pod.

### opsManager.podLimitMemory

The maximum memory that can be assigned to each Ops Manager Application Server pod. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### opsManager.podRequestsMemory

The initial memory assigned to each Ops Manager Application Server pod. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### opsManager.initPodLimitCPU

The CPU limit that can be assigned to each Ops Manager Application Server init pod.

### opsManager.initPodRequestsCPU

The initial CPUs assigned to each Ops Manager Application Server init pod.

### opsManager.initPodLimitMemory

The maximum memory that can be assigned to each Ops Manager Application Server init pod. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### opsManager.initPodRequestsMemory

The initial memory assigned to each Ops Manager Application Server init pod. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### opsManager.localBinariesMountEnabled

A bollean to determine if a VPC is created for the MongoDB binaries. This is requried when `opsManager.binarySource` is `local` mode.

### opsManager.localBinariesMountStorageClass

The name of the Kubernetes StorageClass that will be used to create the PVC to storage the MongoDB binaries, ir required.

### opsManager.localBinariesMountStorageSize

The size of the storage required for the MongoDB binaries, if needed. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### opsManager.extServiceEnabled

A boolean value to determine if access to Ops Manager is required from external to Kubernetes (this includes other Kubernetes clusters).

### opsManager.extServiceType

The type of Kuberentes Service to create for the external access. Can be `NodePort` or `LoadBlaancer`.

Kubernetes [documentation](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types) should be consulted on the best method for the environment.

Only required if `opsManager.extServiceEnabled` is set to `true`.

### opsManager.centralUrl

The protocol, FQDN and port number that will be used by clients to gain access to Ops Manager. This can be a CNAME that resolves to a work node or the load balancer (depends on the selection for `opsManager.extServiceType`)

If not provided the default is `https://<deploymentName>-svc.<namespace>.svc.cluster.local`

### opsManager.extServicePort

The worker node port to use for external access if `NodePort` is selected for `opsManager.extServiceType`. This port must be unique wthin the whole Kubernetes cluster and not in use. Ports normally start from 30000.

Only required if `opsManager.extServiceType` is set to `NodePort`

### appDB.replicas

Number of replica members to create in the AppDB replica set, this should be at least `3` and an odd number.

### appDB.mdbVersion

The MongoDB version that will be deployed as the AppDB. This must aligned with a tag in the quay.io repository for [mongodb-enterprise-appdb-database](https://quay.io/repository/mongodb/mongodb-enterprise-appdb-database?tab=tags).

### appDB.mongoDBAdminPasswdSecretName

The name of the Kubernetes Secret for the MongoDB admin user (mongodb-ops-manager) for the AppDB. The key within th esecret must be called `password`. 

See further information in the [MongoDB Admin User](#mongodb-admin-user-required) section.

### appDB.tlsSecretName

The name of the Kubernets TLS Secret that contains the X.509 key and certificate for the AppDB.

Details on this secret can be found in the [TLS Secrets for AppDB](#tls-secrets-for-appdb-highly-encouraged) section.

### appDB.CAConfigmapName

The name of the Kubernetes Configmap containing the CA X.509 certificate associated with the TLS certificates.

Further information can be found on this in the [CA Certificate for AppDB](#ca-certificate-for-appdb-highly-encouraged) section.

### appDB.podLimitCPU

The CPU limit that can be assigned to each AppDB pod.

### appDB.podRequestsCPU

The initial CPUs assigned to each AppDB pod.

### appDB.podLimitMemory

The maximum memory that can be assigned to each AppDB pod. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### appDB.podRequestsMemory

The initial memory assigned to each AppDB pod. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### appDB.initPodLimitCPU

The CPU limit that can be assigned to each AppDB init pod.

### appDB.initPodRequestsCPU

The initial CPUs assigned to each AppDB init pod.

### appDB.initPodLimitMemory

The maximum memory that can be assigned to each AppDB init pod. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### appDB.initPodRequestsMemory

The initial memory assigned to each AppDB init pod. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### appDB.storageClass

The name of the Kubernetes StorageClass that will be used for persistent storage for the AppDB.

### appDB.storageSize

The size of the storage that will be allocated to the pods from the `appDB.storageClass`. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### backups.enabled

A boolean value to determine if the Ops Manager backup system is enable and deployed.

### backups.members

The number of backup daemons to deploy. As of Ops Manager 4.2 the Backup Daemon really only does cleaning/pruning of backups and is responsible for queryable backups.

Required if `backups.enabled` is `true`. 

### backups.head.binariesMountEnabled

If `opsManager.binarySource` is set to `local` then this also should be true so the MongoDB binaries for the head database have a storage location.

### backups.head.binariesStorageClass

The name of the Kubernetes StorageClass that will be used to create the PVC for the local binaries store.

Required only if `backups.head.binariesMountEnabled` is true.

### backups.head.binariesStorageSize

The size of the storage to allocate for the MongoDB binaries.

Required only if `opsManager.binarySource` is set to `local`.

### backups.head.storageClass

The name of the Kubernetes StorageClass that will be used to create the PVC for the head database. As of Ops Manager 4.2 backing up MongoDB 4.2 with FCV of 4.2 this is of limited value.

Is this is absent the default StorageClass will be used if configured in Kubernetes.

### backups.head.storageSize

The size of the storage to allocate for the head database.

Required if `backups.enabled` is `true`. 

### backups.head.podLimitCPU

The CPU limit that can be assigned to each head pod.

### backups.head.podRequestsCPU

The initial CPUs assigned to each head pod.

### backups.head.podLimitMemory

The maximum memory that can be assigned to each head pod. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### backups.head.podRequestsMemory

The initial memory assigned to each head pod. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### backups.head.initPodLimitCPU

The CPU limit that can be assigned to each head init pod.

### backups.head.initPodRequestsCPU

The initial CPUs assigned to each head init pod.

### backups.head.initPodLimitMemory

The maximum memory that can be assigned to each head init pod. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### backups.head.initPodRequestsMemory

The initial memory assigned to each head init pod. The units suffix can be one of the following: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.

### backups.oplogStores[]

An array of Oplog store resources.

### backups.oplogStores.name

The name that will be configured for the Oplog resource.

### backups.oplogStores.mdbResource

The name of the Kubernetes MongoDB resource that is selected as this particular Oplog Store.

### backups.oplogStores.userResource

The name of the Kubernetes MongoDB User resource that has the required privileges to be the backup user for the MongoDB resource.

### backups.blockstores[]

An array of Blockstore resources for snapshots.

This or s3stores is required if `backups.enabled` is `true`.

### backups.blockstores[].name

The name that will be configured for the Blockstore resource.

### backups.blockstores[].mdbResource

The name of the Kubernetes MongoDB resource that is selected as this particular Blockstore.

### backups.blockstores[].userResource

The name of the Kubernetes MongoDB User resource that has the required privileges to be the backup user for the MongoDB resource.

### backups.s3stores[]

An array of S3 resources for snapshots.

### backups.s3stores[].name

The name that will be configured for the S3 snapshot store resource.

### backups.s3stores[].mdbResource

The name of the Kubernetes MongoDB resource that is selected as the MongoDB replica set for metadata.

### backups.s3stores[].userResource

The name of the Kubernetes MongoDB User resource that has the required privileges to be the backup user for the MongoDB resource used as the metadata store.

### backups.s3stores[].s3secret

The name of the Kubernetes secret that contains the various AWS keys and usernames for the S3 device.

### backups.s3stores[].pathStyleEnabled

A boolean determining if path style URL are used (if `true`) or virtual-host-style URL are used (if `false`).

### backups.s3stores[].bucketEndpoint

The URL of the S3 endpoint.

### backups.s3stores[].bucketName

Name of the S3 bucket that will host the snapshots.

## Run

To use the Helm charts via helmfile perform the following:

```shell
ENV=dev NS=mongodb KUBECONFIG=$PWD/kubeconfig helmfile apply
```

The `kubeconfig` is the config file to gain access to the Kubernetes cluster. The `ENV=dev` is the environment to use for the `values.yaml`, in this case an environment called `dev`.

To see what the actual YAML files will look like without applying them to Kubernetes use:

```shell
ENV=dev helmfile template
```

To destroy the environment (the PersistentVolumes will remain) use the following command:

```shell
ENV=dev NS=mongodb KUBECONFIG=$PWD/kubeconfig helmfile destroy
```