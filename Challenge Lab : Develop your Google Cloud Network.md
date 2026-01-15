# Develop your Google Cloud Network: Challenge Lab

## Challenge scenario
As a cloud engineer at Jooli Inc. and recently trained with Google Cloud and Kubernetes, you have been asked to help a new team (Griffin) set up their environment. The team has asked for your help and has done some work, but needs you to complete the work.
You are expected to have the skills and knowledge for these tasks so don’t expect step-by-step guides.

You need to complete the following tasks:
- Create a development VPC with three subnets manually
- Create a production VPC with three subnets manually
- Create a bastion that is connected to both VPCs
- Create a development Cloud SQL Instance and connect and prepare the WordPress environment
- Create a Kubernetes cluster in the development VPC for WordPress
- Prepare the Kubernetes cluster for the WordPress environment
- Create a WordPress deployment using the supplied configuration
- Enable monitoring of the cluster
- Provide access for an additional engineer
- Some Jooli Inc. standards you should follow:

Create all resources in the us-east1 region and us-east1-c zone, unless otherwise directed.
Use the project VPCs.
Naming is normally team-resource, e.g. an instance could be named kraken-webserver1.
Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share: unless directed, use e2-medium.
Your challenge
You need to help the team with some of their initial work on a new project. They plan to use WordPress and need you to set up a development environment. Some of the work was already done for you, but other parts require your expert skills.

As soon as you sit down at your desk and open your new laptop you receive the following request to complete these tasks. Good luck!

Environment
<img width="813" height="821" alt="1" src="https://github.com/user-attachments/assets/a38ced15-5f97-4e7b-b583-667d24c8ded3" />


## Task 1. Create development VPC manually
Create a VPC called griffin-dev-vpc with the following subnets only:

griffin-dev-wp
IP address block: 192.168.16.0/20
griffin-dev-mgmt
IP address block: 192.168.32.0/20

Run this command to authenticate the Cloud Shell: A popup is likely to be displayed, click on the “Activate“ button and move ahead.
```yml
gcloud auth list
```

Confirm that you’re in the right project:
```yml
gcloud config list project
```

Save the Region and Zone provided in the lab instructions as variables
```yml
export REGION=us-central1
export ZONE=us-central1-a
```

Run the echo command to confirm that the right values have been saved.
```yml
echo $REGION
echo $ZONE
```
![1](https://github.com/user-attachments/assets/a7b560b0-9ff7-49c4-a232-0b73fec694c2)

Create the griffin-dev-vpc network with the command below. We will also set the subnet mode of the network to custom, thus allowing us to manually create our own subnetworks from the network.
```yml
gcloud compute networks create griffin-dev-vpc --subnet-mode custom
```

Create griffin-dev-wp subnetwork using the IP address block: 192.168.16.0/20 as the range and specifying the provided region as our preferred resource location.
```yml
gcloud compute networks subnets create griffin-dev-wp --network griffin-dev-vpc --range 192.168.16.0/20 --region $REGION
```

Create griffin-dev-mgmt subnetwork using the IP address block: 192.168.32.0/20 as the range and specifying the provided region as our preferred resource location.
```yml
gcloud compute networks subnets create griffin-dev-mgmt --network griffin-dev-vpc --range 192.168.32.0/20 --region $REGION
```
![2](https://github.com/user-attachments/assets/dde4385d-5388-4447-a04c-3bb860f0717a)

![3](https://github.com/user-attachments/assets/fd1ca353-f098-42db-bc20-0712d3f620cf)

## Task 2: Create Production VPC Manually
Create a VPC called griffin-prod-vpc with the following subnets only:

griffin-prod-wp
- IP address block: 192.168.48.0/20
griffin-prod-mgmt
- IP address block: 192.168.64.0/20

Create griffin-prod-vpc.
```yml
gcloud compute networks create griffin-prod-vpc --subnet-mode custom
```


Create griffin-prod-wp subnetwork.

![4](https://github.com/user-attachments/assets/df826727-fcda-490e-8cd1-75df887abb5f)

```yml
gcloud compute networks subnets create griffin-prod-wp --network griffin-prod-vpc --range 192.168.48.0/20 --region $REGION
```

Create griffin-prod-mgmt subnetwork.
```yml
gcloud compute networks subnets create griffin-prod-mgmt --network griffin-prod-vpc --range 192.168.64.0/20 --region $REGION
```

![5](https://github.com/user-attachments/assets/ba363e5f-7581-4bdb-89ba-2b0b0c05b137)

## Task 3: Create Bastion Host
Create a bastion host (instance) with two network interfaces, one connected to griffin-dev-mgmt and the other connected to griffin-prod-mgmt. Make sure you can SSH to the host.

A **_bastion host_** in **_Google Cloud_** is a **secure virtual machine** used to **access other private instances** that do not have **public IP addresses**.

Instead of exposing **all servers** to the **internet**, you **SSH into the _bastion host_ first**, and from there you connect to **internal resources**. This improves **_security_** by controlling and monitoring access through **one entry point**.

Here, to achive this;
We need to create a bastion instance with two network interfaces. One interface connects to the griffin-dev-vpc so it can access the griffin-dev-mgmt subnetwork. The other interface connects to the griffin-prod-vpc to access the griffin-prod-mgmt subnetwork.

To allow SSH access, we must add the SSH tag and choose the zone when creating the instance.

After that, we need to create firewall rules for both networks. These rules will control incoming and outgoing traffic to keep the bastion host secure and monitored.

Run the command below to create the bastion instance:
```yml
gcloud compute instances create bastion --network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt --network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt --tags=ssh --zone=$ZONE
```

![6](https://github.com/user-attachments/assets/f5fc0edb-32b7-48a0-952d-f696110155ee)

Create the firewall rule for the griffin-dev-vpc network that only allows SSH traffic on TCP port 22.
We can use 0.0.0.0/0 as the source range to allow connections from any IP address. This is not recommended for real environments, but it is acceptable for this temporary challenge lab.

We will set the target tag to ssh so it matches the tag used when creating the bastion instance. This makes sure the firewall rule applies only to instances with the ssh tag.
```yml
gcloud compute firewall-rules create fw-ssh-dev --network=griffin-dev-vpc --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22
```

Create the firewall rule for the griffin-prod-vpc network.
```yml
gcloud compute firewall-rules create fw-ssh-prod --network=griffin-prod-vpc --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22
```

![7](https://github.com/user-attachments/assets/4a4a8601-30fd-4aab-bd48-e7050391c02e)

## Task 4: Create and Configure Cloud SQL Instance.
1. Create a MySQL Cloud SQL Instance called griffin-dev-db in $REGION.
```yml
gcloud sql instances create griffin-dev-db --root-password password --region $REGION
```
![8](https://github.com/user-attachments/assets/48516213-8614-44e7-816e-1c3b2fea8703)

Run the command below to connect to the SQL instance you just created. You will be prompted to enter a password, enter the password you used in the previous instance creation command. Here it is “password“.

```yml
gcloud sql connect griffin-dev-db
```
Once in the MySQL environment, copy and paste in the SQL queries provided below and press enter. Type exit to come out of the SQL environment once done. Take note of the username: wp_user and password: stormwind_rules they will be used in later steps

```yml
CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%";
FLUSH PRIVILEGES;
```

![9](https://github.com/user-attachments/assets/e6a6746f-657f-4f7d-82af-1199f6df9810)

## Task 5: Create Kubernetes Cluster
Create a 2 node cluster (e2-standard-4) called griffin-dev, in the griffin-dev-wp subnet, and in zone $ZONE.

Run the command below to achieve this. Specifying the machine-type as e2-standard-4, number of nodes as 2, network as griffin-dev-vpc, subnetwork as griffin-dev-wp, and zone as the provided lab zone.

```yml
gcloud container clusters create griffin-dev --machine-type e2-standard-4 --num-nodes 2 --network griffin-dev-vpc --subnetwork griffin-dev-wp --zone $ZONE
```

## Task 6: Prepare the Kubernetes Cluster.

From Cloud Shell copy all files from gs://cloud-training/gsp321/wp-k8s.
- Do this with the command below
```yml
gsutil cp -r gs://cloud-training/gsp321/wp-k8s .
```
Now change directory to the wp-k8s folder.
```yml
cd wp-k8s
```
Now inside the folder, edit the wp-env.yaml by running this command:
```yml
nano wp-env.yaml
```

![10](https://github.com/user-attachments/assets/32b27cb1-5b1b-44ab-b171-72ff337191b6)

Once the file content is open, edit it to look like below. Changing the username field to wp_user and the password field to stormwind_rules respectively.

Save the file and exit nano by pressing CTRL+O, then Enter, then CTRL+X.

Now that we’ve appropriately edited the file, we need to create the configuration based on the resources specified in the wp-env.yaml file. We do this by running the following command:
```yml
kubectl create -f wp-env.yaml
```

Use the command below to create and add the key to the Kubernetes environment.
```yml
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.jsonc
```

![11](https://github.com/user-attachments/assets/8a702292-b7fd-4356-910a-a5d94fb6a31a)

## Task 7: Create a WordPress Deployment.
Now that you have provisioned the MySQL database, and set up the secrets and volume, you can create the deployment using wp-deployment.yaml.

1. Before you create the deployment you need to edit wp-deployment.yaml.
2. Replace YOUR_SQL_INSTANCE with griffin-dev-db's Instance connection name.
3. Get the Instance connection name from your Cloud SQL instance.
4. After you create your WordPress deployment, create the service with wp-service.yaml.

Edit the wp-deployment.yaml file by running the command below to open it up with the nano editor in the Cloud Shell.
```yml
nano wp-deployment.yaml
```

Edit the file by changing YOUR_SQL_INSTANCE to griffin-dev-db. You can confirm the instance connection name from Cloud Console by navigating to SQL > Instances like so:

Change WORDPRESS_DB_USER to wp_user, and WORDPRESS_DB_PASSWORD to stormwind_rules

Save the file and exit nano by pressing CTRL+O, then Enter, then CTRL+X.

![12](https://github.com/user-attachments/assets/caf8cb88-175b-4d53-8949-9b38d7b503ce)

Create the WordPress Deployment by running the following command
```yml
kubectl create -f wp-deployment.yaml
```

Create the Service by running the following command
```yml
kubectl create -f wp-service.yaml
``` 
You can check that the deployment and service have been created by running the following commands

```yml
kubectl get deployments
kubectl get services
``` 

![13](https://github.com/user-attachments/assets/34233605-be98-4895-b2b6-0226ddb1616a)

## Task 8: Enable Monitoring.
- Create an uptime check for your WordPress development site. This periodically verifies the availability of the WordPress development site.
You can do this by running the following command:

```yml
gcloud monitoring uptime create wp-uptime-check \
    --resource-type=uptime-url \
    --resource-labels=host=EXTERNAL_IP_ADDRESS
```

![14-1](https://github.com/user-attachments/assets/115de3c8-4776-4ad0-be9e-25959d762907)

## Task 9: Provide Access for an Additional Engineer
You have an additional engineer starting and you want to ensure they have access to the project. Grant them the editor role to the project.
The second user account for the lab represents the additional engineer.

We do this by navigating to the IAM & Admin section in Cloud Console to edit the role of the second user provided in the lab from Viewer to Editor as the case may be. This is illustrated in the images below:

- the second user detail:
- Locate the second user on the IAM page in Cloud Console and Click on the Edit icon:
- Click on the Role field and Select Editor under the Roles section:
- Click Save once done:
  
![14](https://github.com/user-attachments/assets/047e06b4-8a6d-4c27-b245-d05874c23a39)

![15](https://github.com/user-attachments/assets/b7b35e7f-73bb-4209-b1d2-e39881e52629)

✨🎉 Wow! You’ve completed the challenge successfully! 🚀👏

<img width="1080" height="1080" alt="develop-your-google-cloud-network-skill-badge" src="https://github.com/user-attachments/assets/d4be20a3-9490-4c22-8246-60806782ff68" />


