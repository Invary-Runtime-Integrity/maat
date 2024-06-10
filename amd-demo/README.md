# Objectives

This software is meant to show how an attestation framework might combine the results of attestations of Invary OS 
integrity and AMD SEV-SNP memory integrity to drive operational changes to the servicing of workloads by a Kubernetes
(K8S) cluster.

# Prerequisites

## AMD Hardware and Software

The AMD [snpguest](https://github.com/virtee/snpguest) tool will only complete full attestations on AMD hardware 
with SEV-SNP capabilities. While a binary version is included at 
[./amd-demo/invary-maat-docker/bin/snpguest](./amd-demo/invary-maat-docker/bin/snpguest),
you may need to build it from source for the latest features and hardware support.

You can test it in absence of the rest of the framework by running `snpguest ok`.

## Invary Software

You will need to obtain a licensed or evaluation copy of Invary software to run this demo. The versions included at

    ./amd-demo/invary-maat-docker/bin/invary
    ./amd-demo/invary-maat-docker/bin/invary-appraiser
    
are stubs which print an advisory message and exit with a 1 status code. If you replace those with binaries obtained
from Invary, the later **Build** instructions will allow you to deploy them to an environment.

## Docker Registry

You will need a local or infrastructure Docker registry in order to deploy the K8S pods.

## Helm

[https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/)

## Kubernetes Cluster

You will want a Kubernetes (K8S) cluster to install the components. The primary components used are:

* Kubernetes Dashboard
* Invary Maat as DaemonSet (runs on every node)
* Invary Maat Controller as Deployment
* Demo Workloads (don't do anything of substance besides respond to taints and tolerations)
* kubectl - configured to point to a *DEMO* cluster. The tool modifies workloads, so should not be run on critical infrastructure. 
    * sanity check: `kubectl get nodes`

# Building

## Point the Docker-dependent files to your Docker registry
 
    ./amd-demo/invary-maat-controller-docker/Dockerfile
    ./amd-demo/invary-maat-k8s/invary-maat/daemonset.yaml 
    ./amd-demo/invary-maat-k8s/invary-maat-controller/deployment.yaml

Replace the value `MY_REGISTRY` with your Docker registry URL.

## Let Maat and Maat controller know about your network topology

For Maat, update the known-clients collection here:

    ./amd-demo/invary-maat-docker/maat/share/maat/selector-configurations/invary-selector.xml

For the Maat controller, update the node information here:

    ./amd-demo/invary-maat-controller-docker/appraise

## Build Docker images

Replace the value `MY_REGISTRY` with your Docker registry URL.

    cd ./amd-demo/invary-maat-docker/
    docker build -t MY_REGISTRY/invary-demo/invary-maat:latest . && docker push MY_REGISTRY/invary-demo/invary-maat:latest
    cd ../invary-maat-controller-docker/
    docker build -t MY_REGISTRY/invary-demo/invary-maat-controller:latest . && docker push MY_REGISTRY/invary-demo/invary-maat-controller:latest

# Deployment

## Kubernetes Dashboard  

    helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard --version 7.3.2

    kubectl edit deployment -n kubernetes-dashboard

For the kubernetes-dashboard deployment, modify the tolerations to run regardless of confidentiality or integrity.
This will prevent the dashboard from terminating while demoing confidentiality gaps.

    - operator: Exists

You will need to manage routing or forwarding to the Kubernetes Dashboard tool to make it available at a location you can access.
One example that works for a local Kubernetes dashboard:

    kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard-kong-proxy 8443:443

## Invary Maat DaemonSet

    kubectl apply -k ./amd-demo/invary-maat-k8s/invary-maat

## Invary Maat Controller Deployment

    kubectl apply -k ./amd-demo/invary-maat-k8s/invary-maat-controller

## Kubernetes Dashboard

This change will make sure you have a user that can access the dashboard. Tokens can be retrieved by:

    kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d

# Flow

## Enter Invary Maat Controller

    kubectl exec -it -n invary invary-maat-controller-{SUFFIX} -- bash

## Appraisal

    ./appraise

## Appraisal with Enforcement

    ./appraise --enforce

## Demo SEV-SNP Attestation Failure

Exec a terminal on a Kubernetes node and run:

    rmmod sev-guest

While this does not remove the underlying hardware capability of SEV-SNP, it disrupts the attestation flow and triggers
a failed attestation. This can be seen using the above actions of `Appraisal` or `Appraisal with Enforcement`.

## Review the Kubernetes Dashboard

Open `https://localhost:8443/`


