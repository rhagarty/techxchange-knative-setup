# Instructions for setting up Knative on the OCP cluster

These instructions should only be completed if:

1. You are the workshop instructor and you need to set up Knative on OCP so that all students will able to run InstantOn applications on that shared platform.

2. You are attempting to complete the lab on your own outside of a workshop environment.
   
It can also serve as a good reference for workshop attendees wishing to get details on what is involved in the Knative setup process.

# Steps:

1. [Initial lab setup](#1-initial-lab-setup)
1. [Enhance the OpenShift Cloud Platform (OCP) environment](#2-enhance-the-openshift-cloud-platform-ocp-environment)
2. [Configure the Knative service](#3-configure-the-knative-service)

## 1. Initial lab setup

### Login as root

From the terminal, login as root:

```bash
su --login root
```

Use the password provided in the TechZone VM reservation.

### Clone the application from GitHub

```bash
cd /home/techzone
git clone https://github.com/rhagarty/techxchange-knative-setup.git
cd techxchange-knative-setup
```

### Login to the OpenShift console

Click on the TechZone OCP reservation to find the link to the OpenShift console UI, along with the username and password to access the console.

![ocp-access](images/ocp-access.png)

### Login to the OpenShift CLI

From the OpenShift console UI, click the username in the top right corner, and select `Copy login command`.

![ocp-cli-login](images/ocp-cli-login.png)

Press `Display Token` and copy the `Log in with this token` command.

![ocp-cli-token](images/ocp-cli-token.png)

Paste the command into your terminal window. You should receive a confirmation message that you are logged in.

## 2. Enhance the OpenShift Cloud Platform (OCP) environment

Perform the following steps to enhance OCP to better manage OCP services, such as Knative, which provides serverless or scale-to-zero functionality. 

### Install the Liberty Operator

The Liberty Operator provides resources and configurations that make it easier to run Open Liberty applications on OCP.

```bash
kubectl apply --server-side -f https://raw.githubusercontent.com/OpenLiberty/open-liberty-operator/main/deploy/releases/1.2.1/kubectl/openliberty-app-crd.yaml
```

### Install the OpenShift serverless operator

Switch to the "default" namespace:

```bash
oc project default
```

Type the following commands to install the serverless operator and Knative service.

```bash
oc apply -f serverless-subscription.yaml
```

```bash
oc apply -f serving.yaml
```

### Verify the OpenShift serverless operator is installed and ready

```bash
oc get csv
```

You should see the following output:

![ocp-serverless](images/ocp-serverless.png)

### Install the Cert Manager 

The Cert Manager adds certifications and certification issuers as resource types to Kubernetes

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.3/cert-manager.yaml
```

### Set parameter to enable InstantOn capabilities

In order to build InstantOn images, enable sandbox containers to use netlink system calls. 

```bash
setsebool virt_sandbox_use_netlink 1
```

## 3. Configure the Knative service

Knative provides the serverless, or scale-to-zero feature of Kubernetes.

### Verify the Knative service is ready

> **NOTE**: The service may take a minute to load. after issuing the preceding `apply` command.

```bash
oc get knativeserving.operator.knative.dev/knative-serving -n knative-serving --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'
```

Your output should match the following:

![ocp-knative](images/ocp-knative.png)

### Edit the Knative permissions to allow to the ability to add Capabilities

```bash
kubectl -n knative-serving edit cm config-features -oyaml
```

Add in the following line just bellow the “data” tag at the top:
```yaml
kubernetes.containerspec-addcapabilities: enabled
```

> **IMPORTANT**: to save your change and exit the file, hit the escape key, then type `:x`. 

### Verify Knative time-out periods 

Knative will stop the pod if it does not receive a request in the specified time frame, which is set in a configuration yaml file. For this lab, the settings are in the `serving.yaml` file, and currently set to 30 seconds (as shown below).

```bash
apiVersion: operator.knative.dev/v1beta1
kind: KnativeServing
metadata:
    name: knative-serving
    namespace: knative-serving
spec:
  config:
    autoscaler:
      scale-to-zero-grace-period: "30s"
      scale-to-zero-pod-retention-period: "0s"
```