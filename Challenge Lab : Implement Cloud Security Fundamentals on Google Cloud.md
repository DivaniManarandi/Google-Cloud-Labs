# Implement Cloud Security Fundamentals on Google Cloud : Challenge Lab

## Challenge scenario
You have started a new role as a junior member of the security team for the Orca team in Jooli Inc. 
Your team is responsible for ensuring the security of the Cloud infrastructure and services that the company's applications depend on.
You are expected to have the skills and knowledge for these tasks, so don't expect step-by-step guides to be provided.

## Your challenge

You have been asked to deploy, configure, and test a new Kubernetes Engine cluster that will be used for application development and pipeline testing by the Orca development team.
As per the organization's security standards you must ensure that the new Kubernetes Engine cluster is built according to the organization's most recent security standards and thereby must comply with the following:

The cluster must be deployed using a dedicated service account configured with the least privileges required.
The cluster must be deployed as a Kubernetes Engine private cluster, with the public endpoint disabled, and the master authorized network set to include only the ip-address of the Orca group's management jumphost.
The Kubernetes Engine private cluster must be deployed to the orca-build-subnet in the Orca Build VPC.
From a previous project you know that the minimum permissions required by the service account that is specified for a Kubernetes Engine cluster is covered by these three built in roles:
- _roles/monitoring.viewer_
- _roles/monitoring.metricWriter_
- _roles/logging.logWriter_

These roles are specified in the Google Kubernetes Engine (GKE)'s Harden your cluster's security guide in the Use least privilege Google service accounts section.
You must bind the above roles to the service account used by the cluster as well as a custom role that you must create in order to provide access to any other services specified by the development team. 
Initially you have been told that the development team requires that the service account used by the cluster should have the permissions necessary to add and update objects in Google Cloud Storage buckets. 
To do this you will have to create a new custom IAM role that will provide the following permissions:

- _storage.buckets.get_
- _storage.objects.get_
- _storage.objects.list_
- _storage.objects.update_
- _storage.objects.create_
  
Once you have created the new private cluster you must test that it is correctly configured by connecting to it from the jumphost, orca-jumphost, in the management subnet orca-mgmt-subnet. 
As this compute instance is not in the same subnet as the private cluster you must make sure that the master authorized networks for the cluster includes the internal ip-address for the instance, and you must specify the --internal-ip flag when retrieving cluster credentials using the gcloud container clusters get-credentials command.
All new cloud objects and services that you create should include the "orca-" prefix.
Your final task is to validate that the cluster is working correctly by deploying a simple application to the cluster to test that management access to the cluster using the kubectl tool is working from the orca-jumphost compute instance.
For all tasks in this lab, use the Region region and the Zone zone.

## Solving tasks
Export the following variables
Make sure to replace [content] with the values of your lab!
```yml
export CUSTOM_SECURIY_ROLE=[your_custom_security_role]
export SERVICE_ACCOUNT=[your_service_account]
export CLUSTER_NAME=[your_cluster_name]
export ZONE=[your_zone]
```
## Task 1. Create a custom security role
Your first task is to create a new custom IAM security role called Custom Securiy Role that will provide the Google Cloud storage bucket and object permissions required to be able to create and update storage objects.

Set the Compute Zone:
```yml
gcloud config set compute/zone $ZONE
```
Create a Role Definition File:
```yml
cat > role-definition.yaml <<EOF_END
title: "$CUSTOM_SECURIY_ROLE"
description: "Permissions"
stage: "ALPHA"
includedPermissions:
- storage.buckets.get
- storage.objects.get
- storage.objects.list
- storage.objects.update
- storage.objects.create
EOF_END
```
Create the Custom Security Role:
```yml
gcloud iam service-accounts create orca-private-cluster-sa --display-name "Orca Private Cluster Service Account"
gcloud iam roles create $CUSTOM_SECURIY_ROLE --project $DEVSHELL_PROJECT_ID --file role-definition.yaml
```

## Task 2. Create a service account
Your second task is to create the dedicated service account that will be used as the service account for your new private cluster. You must name this account Service Account.
```yml
gcloud iam service-accounts create $SERVICE_ACCOUNT --display-name "Orca Private Cluster Service Account"
```

## Task 3. Bind a custom security role to a service account
You must now bind the Cloud Operations logging and monitoring roles that are required for Kubernetes Engine Cluster service accounts as well as the custom IAM role you created for storage permissions to the Service Account you created earlier.

```yml
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member serviceAccount:$SERVICE_ACCOUNT@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/monitoring.viewer
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member serviceAccount:$SERVICE_ACCOUNT@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/monitoring.metricWriter
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member serviceAccount:$SERVICE_ACCOUNT@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/logging.logWriter
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member serviceAccount:$SERVICE_ACCOUNT@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role projects/$DEVSHELL_PROJECT_ID/roles/$CUSTOM_SECURIY_ROLE
```
## Task 4. Create and configure a new Kubernetes Engine private cluster
Create the GKE Cluster:
```yml
gcloud container clusters create $CLUSTER_NAME --num-nodes 1 --master-ipv4-cidr=172.16.0.64/28 --network orca-build-vpc --subnetwork orca-build-subnet --enable-master-authorized-networks --master-authorized-networks 192.168.10.2/32 --enable-ip-alias --enable-private-nodes --enable-private-endpoint --service-account $SERVICE_ACCOUNT@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --zone $ZONE
```
You must now use the service account you have configured when creating a new Kubernetes Engine private cluster. The new cluster configuration must include the following:

- The cluster must be called Cluster Name
- The cluster must be deployed to the subnet orca-build-subnet
- The cluster must be configured to use the Service Account service account.
- The private cluster options enable-master-authorized-networks, enable-ip-alias, enable-private-nodes, and enable-private-endpoint must be enabled.

Once the cluster is configured you must add the internal ip-address of the orca-jumphost compute instance to the master authorized network list.

## Task 5. Deploy an application to a private Kubernetes Engine cluster
You have a simple test application that can be deployed to any cluster to quickly test that basic container deployment functionality is working and that basic services can be created and accessed. You must configure the environment so that you can deploy this simple demo to the new cluster using the jumphost orca-jumphost.

Connect to the Cluster and Deploy:
```yml
gcloud compute ssh --zone "$ZONE" "orca-jumphost" --project "$DEVSHELL_PROJECT_ID" --quiet --command "gcloud config set compute/zone $ZONE && gcloud container clusters get-credentials $CLUSTER_NAME --internal-ip && sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin && kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0 && kubectl expose deployment hello-server --name orca-hello-service --type LoadBalancer --port 80 --target-port 8080"
```

Note: Make sure to properly install the gke-gcloud-auth-plugin before running any kubectl commands. This is detailed below in Tip 1.
```yml
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```
This deploys an application that listens on port 8080 that can be exposed using a basic load balancer service for testing.

### Tips and Tricks
Tip 1. Make sure to use the gke-gcloud-auth-plugin, which is needed for continued use of kubectl. You can install it by running the following commands. Make sure to replace your GKE cluster name and zone, as well as your Project ID.
```yml
sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
echo "export USE_GKE_GCLOUD_AUTH_PLUGIN=True" >> ~/.bashrc
source ~/.bashrc
gcloud container clusters get-credentials <your cluster name> --internal-ip --project=<project ID> --zone <cluster zone>
```
Tip 2. When adding the internal ip-address of the orca-jumphost machine to the list of authorized addresses for the private Kubernetes Engine cluster you should use a /32 netmask to ensure that only the specific compute instance is authorized.

Tip 3. You cannot connect directly to a Kubernetes Engine private cluster from a VPC or other network outside of the VPC the private cluster has been deployed to if the enable-private-endpoint option has been specified. This represents the highest security option for a private cluster and you must use a jumphost, or a proxy within the same VPC as the cluster, and you must use that jumphost or proxy to connect to the internal managment ip-address for the cluster.
