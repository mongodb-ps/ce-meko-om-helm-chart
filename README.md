# k8s_om

**This documentation is a WIP**

## Table of Contents

## Description

A series of Helm Charts to deploy MongoDB Ops Manager within Kubernetes with the MongoDB Kubernetes Operator.

## Prerequisites

The [MongoDB Enterprise Kubernetes Operator](https://docs.mongodb.com/kubernetes-operator/master/) must be installed and operation. Instructions on installing the MongoDB Kubernetes Operator can be found in the MongoDB [documentation](https://docs.mongodb.com/kubernetes-operator/master/installation/). It is highly recommended that MongoDB Professional Services be engaged to deploy and configure Ops Manager to ensure the deployment is secure and reliable.

[Helm](https://helm.sh/docs/intro/install/) is required to be installed and [Helmfile](https://github.com/roboll/helmfile) is also highly recommended. If Helmfile is used you will also need [Helm-Diff](https://github.com/databus23/helm-diff).


# Notes
OM Admin user secret
OM TLS secret
APPDB TLS secret