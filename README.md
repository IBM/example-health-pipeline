# example-health-pipeline

## Background

Example Health is a demo app the folks in my group created along with some other folks from the z/OS group.  Initally, it was meant to demonstrate app moderization, but evolved over time to also include analytic capibilities.  Currently, five different images constitute the Example Health app, and we have them all deployed into a single Red Hat OpenShift on IBM Cloud cluster, but anytime the app needs to be deployed in a new cluster the install can take a bit of time.  Thus, the impitus for this project - a tekton pipeline to deploy them all!

## Prerequisties

OpenShift cluster
kubectl
oc

## Steps

1. Install Teckton, Dashboard, and extensions
2. apply base pipeline and run
3. optional analytics

### 1. Target your cluster

Login to cloud.ibm.com and go to the overview page for your openshift cluster. Click on the **OpenShift web console** button in the upper right.  Once on your web console, select the dropdown from the upper right corner (the label contains your email address), and select **Copy Login Command**.  Paste this into your local console window.  It should resemble:

```
$ oc login https://c100-e.us-east.containers.cloud.ibm.com:XXXXX --token=XXXXXXXXXXXXXXXXXXXXXXXXXX
```

### 2. Tekton install

```
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/previous/v0.5.2/release.yaml
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/previous/v0.1.1/release.yaml
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/webhooks-extension/previous/v0.1.1/release.yaml
```

### 3. Create service account
```
$ oc create serviceaccount pipeline
$ oc adm policy add-scc-to-user privileged -z pipeline
$ oc adm policy add-role-to-user edit -z pipeline
```

### 4. Apply resources, pipeline and run
```
$ git clone https://github.com/loafyloaf/example-health-pipeline.git
$ cd example-health-pipeline
$ kubectl apply -f example-health-resources
$ kubectl apply -f example-health-pipeline
```

You can then use the tekton dashboard to run your pipeline by selecting the **Example Health Pipeline** from the list of pipelines and then clicking the **Create Pipeline Run** button in the upper right.
