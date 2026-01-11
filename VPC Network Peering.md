## VPC Network Peering

Google Cloud Virtual Private Cloud (VPC) Network Peering allows private connectivity across two VPC networks regardless of whether or not they belong to the same project or the same organization.
VPC Network Peering allows you to build SaaS (Software-as-a-Service) ecosystems in Google Cloud, making services available privately across different VPC networks within and across organizations, allowing workloads to communicate in private space.
VPC Network Peering is useful for:
- Organizations with several network administrative domains.
- Organizations that want to peer with other organizations.

 ### Create a custom network in two projects

 #### 1. Create a custom network in both projects
Within the same organization node, a network could be hosting services that need to be accessible from other VPC networks in the same or different projects.

n this lab, you have been provisioned 2 projects, the first project is project-A and second is project-B.
1. In each project, start a new Cloud Shell by click + icon.
2. In the Cloud Shell for project-A, set the project ID for the project-A
```yaml
gcloud config set project qwiklabs-gcp-04-bc82d4f78061
``` 
3. In the Cloud Shell for project-B, set the project ID for the project-B:
```yaml
gcloud config set project qwiklabs-gcp-00-5816deb5613d
```

**project-A:**

1. Go back to first Cloud Shell and run the following to create a custom network:
```yaml
gcloud compute networks create network-a --subnet-mode custom
```
2. Create a subnet within this VPC and specify a region and IP range by running:
```yaml
gcloud compute networks subnets create network-a-subnet --network network-a \
    --range 10.0.0.0/16 --region "REGION 1"
```
3. Create a VM instance:
```yaml
gcloud compute instances create vm-a --zone "ZONE 1" --network network-a --subnet network-a-subnet --machine-type e2-small
```
4. Run the following to enable SSH and icmp, because you'll need a secure shell to communicate with VMs during connectivity testing:
```yaml
gcloud compute firewall-rules create network-a-fw --network network-a --allow tcp:22,icmp
```

**project-B:**
1. Switch to the second Cloud Shell and create a custom network:
```yaml
gcloud compute networks create network-b --subnet-mode custom
```
2.Create a subnet within this VPC and specify a region and IP range by running:
```yaml
gcloud compute networks subnets create network-b-subnet --network network-b \
    --range 10.8.0.0/16 --region "REGION 2"
```
3. Create a VM instance:
```yaml
gcloud compute instances create vm-b --zone "ZONE 2" --network network-b --subnet network-b-subnet --machine-type e2-small
```
4. Run the following to enable SSH and icmp, because you'll need a secure shell to communicate with VMs during connectivity testing:
```yaml
gcloud compute firewall-rules create network-b-fw --network network-b --allow tcp:22,icmp
```

#### 2. Set up a VPC network peering session

**Peer network-A with network-B:**

<img width="600" height="292" alt="1" src="https://github.com/user-attachments/assets/a10064b8-6d72-4730-ac70-9096969c273b" />

project-A

Go to the VPC Network Peering in the Cloud Console by navigating to the Networking section and clicking VPC Network > VPC network peering in the left menu. Once you're there:

1. Click Create connection.
2. Click Continue.
3. Type "peer-ab" as the Name for this side of the connection.
4. Under Your VPC network, select the network you want to peer (network-a).
5. Set the Peered VPC network radio buttons to In another project.
6. Paste in the Project ID of the second project.
```yaml
"PROJECT ID 2"
```
7. Type in the VPC network name of the other network (network-b).
8. Click Create.
At this point, the peering state remains INACTIVE because there is no matching configuration in network-b in project-B. You should see the Status message, Waiting for peer network to connect.

**Peer network-b with network-a**

<img width="600" height="289" alt="2" src="https://github.com/user-attachments/assets/96d463a2-78c5-4610-ba57-8f5150ad238f" />

project-B

1. Click Create connection.
2. Click Continue.
3. Type "peer-ba" as the Name for this side of the connection.
4. Under Your VPC network, select the network you want to peer (network-b).
5. Set the Peering VPC network radio buttons to In another project, unless you wish to peer within the same project.
6. Specify the Project ID of the first project.
```yaml
"PROJECT ID 1"
```
7. Specify VPC network name of the other network (network-a).
8. Click Create.
In the VPC network peering, you should now see peer-ba listed in the property list.
VPC Network Peering becomes ACTIVE and routes are exchanged As soon as the peering moves to an ACTIVE state, traffic flows are set up:
- Between VM instances in the peered networks: Full mesh connectivity.
- From VM instances in one network to Internal Load Balancing endpoints in the peered network.

<img width="600" height="291" alt="3" src="https://github.com/user-attachments/assets/320df1be-165a-4d47-a851-44be3441dde7" />

The routes to peered network CIDR prefixes are now visible across the VPC network peers. These routes are implicit routes generated for active peerings. They don't have corresponding route resources. The following command lists routes for all VPC networks for project-A.
```yaml
gcloud compute routes list --project
```
