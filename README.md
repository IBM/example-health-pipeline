# Using Tekton Pipelines with Red Hat OpenShift on IBM Cloud

## Background

Example Health is a demo app the folks in my group created along with some other folks from the z/OS group.  Initally, it was meant to demonstrate app moderization, but evolved over time to also include analytic capibilities.  Currently, five different images constitute the Example Health app, and we have them all deployed into a single Red Hat OpenShift on IBM Cloud cluster, but anytime the app needs to be deployed in a new cluster the install can take a bit of time.  Thus, the impitus for this project - a tekton pipeline to deploy them all!  Using Tekton has reduced deployment time from 45 minutes of clicking around and configuring down to execution of a few commands and waiting for about 15 minutes.

![example-health](./images/example-health.png)

This pipeline uses several methods for deploying images in OpenShift.  The Patent UI image is constructed from node.js code via OpenShift's Source-to-image functionality; the Admin UI likewise from a php repository.  The two analytics images are also build, but rather via Dockerfiles.  The JEE business logic image is deployed directly from Docker Hub.

## Prerequisties

This tutorial assumes you have a Red Hat OpenShift on IBM Cloud cluster already provisioned.  If not, you can create one via the [IBM Cloud web console](https://cloud.ibm.com/kubernetes/catalog/cluster/create?platformType=openshift) or using the `ibmcloud` CLI.  When using the latter, this [tutorial](https://cloud.ibm.com/docs/openshift?topic=openshift-openshift_tutorial#openshift_create_cluster) may come in handy.

You will also need a few CLIs, including `kubectl` and `oc`.  You can get them [here](https://www.okd.io/download.html)

## Steps

1. Target your cluster
2. Install Tekton
3. Create service account
4. Install Tasks
5. Apply resources, pipeline and run

### 1. Target your cluster

Login to cloud.ibm.com and go to the overview page for your openshift cluster. Click on the **OpenShift web console** button in the upper right.  Once on your web console, select the dropdown from the upper right corner (the label contains your email address), and select **Copy Login Command**.  Paste this into your local console window.  It should resemble:

```
$ oc login https://c100-e.us-east.containers.cloud.ibm.com:XXXXX --token=XXXXXXXXXXXXXXXXXXXXXXXXXX
```

### 2. Tekton install

Back in your OpenShift web console, select **Operators** -> **Operators Hub** from the navigation menu on the left of your OpenShift web console and then search for the OpenShift Pipelines Operator.

![operatorhub](./images/operatorhub.png)

Click on the tile and then the subsequent Install button.

![pipelineoperator](./images/pipelineoperator.png)

Keep the default settings on the Create Operator Subscription page and click Subscribe.

### 3. Create service account

To make sure the pipeline has the appropriate permissions to store images in the local OpenShift registry, we need to create a service account.  We'll call it 'pipeline':

```bash
$ oc new-project example-health
$ oc create serviceaccount pipeline
$ oc adm policy add-scc-to-user privileged -z pipeline
$ oc adm policy add-role-to-user edit -z pipeline
```

### 4. Install Tasks

Tekton Pipelines generally are constructed of individual tasks.  We will be using a couple of tasks maintained by the both the Tekton and OpenShift communities: `openshift-client` allows you to execute CLI commands against your OpenShift cluster, and the `s2i-node` and `s2i-php` tasks are responsible for building images via OpenShift's source-to-image functionality.  To install:

```bash
$ git clone https://github.com/loafyloaf/example-health-pipeline.git
$ cd example-health-pipeline
$ oc create -f openshift-client.yaml
$ oc create -f s2i-nodejs.yaml
$ oc create -f s2i-php.yaml

```

### 5. Apply resources, pipeline and run

Now we just need to apply a couple of files to the cluster.  The first, 'example-health-resources' defines the location of the github repositories and the names we will use for the images we create as they are stored in the registry.  As you can probably guess, the `example-health-pipeline` file defines all the steps of our pipeline: building, deploying and exposing our images.

**Note**: While the Patient and Admin UI parts of the Example Health application work out-of-the-box, the Analytics section needs futher information to fully function. You need to edit `example-health-pipeline.yaml` and provide a Mapbox [access token](https://www.mapbox.com/account/access-tokens), the name of your cluster, and your hash (found in the URL of your dashboard) and Mongo datalake credentials. See the Analytics [repo](https://github.com/IBM/example-health-analytics) for more details. You also need to expose the ports in the **analytics** services to their routes once the cluster is set up.

```bash
$ oc apply -f health-pvc.yaml
$ oc apply -f example-health-resources.yaml
$ oc apply -f example-health-pipeline.yaml
```

You can then run your pipeline by executing the command:
```bash
$ oc apply -f health-pipeline-run.yaml
```

Once created, you can follow along with the progress of your pipeline run from the list of  **Pipelines --> Pipeline Runs** in your cluster.  Success looks similar to:

![success](./images/example-health-success.png)

# Sample output

In your web console, navigate to **Networking --> Routes** to see the list of URLs for your applications:

![routes](./images/example-health-routes.png)

Click on the URL listed for the `patientui` to confirm installation:

![patientui](./images/example-health-patientui.png)


## License

This code pattern is licensed under the Apache License, Version 2. Separate third-party code objects invoked within this code pattern are licensed by their respective providers pursuant to their own separate licenses. Contributions are subject to the [Developer Certificate of Origin, Version 1.1](https://developercertificate.org/) and the [Apache License, Version 2](https://www.apache.org/licenses/LICENSE-2.0.txt).

[Apache License FAQ](https://www.apache.org/foundation/license-faq.html#WhatDoesItMEAN)
