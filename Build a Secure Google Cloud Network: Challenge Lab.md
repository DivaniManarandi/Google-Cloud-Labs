## Challenge scenario
You are a security consultant brought in by Jeff, who owns a small local company, to help him with his very successful website (juice-shop). Jeff is new to Google Cloud and had his neighbour's son set up the initial site. The neighbour's son has since had to leave for college, but before leaving, he made sure the site was running.
Below is the current set up:

<img width="630" height="408" alt="2" src="https://github.com/user-attachments/assets/7b94490c-1aff-4fb8-9612-2cf5bd7bf7e1" />


## Your challenge
You need to create the appropriate security configuration for Jeff's site. 

Your first challenge is to set up firewall rules and virtual machine tags. You also need to ensure that SSH is only available to the bastion via IAP.
For the firewall rules, make sure that:
- The bastion host does not have a public IP address.
- You can only SSH to the bastion and only via IAP.
- You can only SSH to juice-shop via the bastion.
- Only HTTP is open to the world for juice-shop.

Tips and tricks:
- Pay close attention to the network tags and the associated VPC firewall rules.
- Be specific and limit the size of the VPC firewall rule source ranges.
- Overly permissive permissions will not be marked correct.

<img width="640" height="415" alt="1" src="https://github.com/user-attachments/assets/5dd23a98-ff4e-4cc1-9255-cdd9ca9eb6ad" />

## Solving tasks

### Task 1: Check the firewall rules. Remove the overly permissive rules.

* Go to **VPC network** > **Firewall** > will see **open-access**
* Use the following command from the cloud console:

```yaml
gcloud compute firewall-rules delete open-access
```

### Task 2: Navigate to Compute Engine in the Cloud Console and identify the bastion host. The instance should be stopped. Start the instance

* Go to **Compute Engine**  > **VM Instances** > Select **bastion** > click on **Start**

![3](https://github.com/user-attachments/assets/8a006225-e545-4341-a52d-0e9ad432ea5b)

### Task 3: The bastion host is the one machine authorized to receive external SSH traffic. Create a firewall rule that allows SSH (tcp/22) from the IAP service. The firewall rule should be enabled on bastion via a network tag.

* Run the following:
* Make sure you replace <SSH IAP network tag> with the tag provided on the Left Pane.

```yaml
gcloud compute firewall-rules create <SSH IAP network tag> --allow=tcp:22 --source-ranges 35.235.240.0/20 --target-tags <SSH IAP network tag> --network acme-vpc

gcloud compute instances add-tags juice-shop --tags=permit-http-ingress-ql-622 --zone=us-west1-a
```

### Task 4: The `juice-shop` server serves HTTP traffic. Create a firewall rule that allows traffic on HTTP (tcp/80) to any address. The firewall rule should be enabled on juice-shop via a network tag

- Run the following:
* Make sure you replace <HTTP network tag> with the tag provided on the Left Pane.

```yaml
gcloud compute firewall-rules create <HTTP network tag> --allow=tcp:80 --source-ranges 0.0.0.0/0 --target-tags <HTTP network tag> --network acme-vpc

gcloud compute instances add-tags juice-shop --tags=<HTTP network tag> --zone=us-central1-b
```

### Task 5: You need to connect to `juice-shop` from the bastion using SSH. Create a firewall rule that allows traffic on SSH (tcp/22) from `acme-mgmt-subnet` network address. The firewall rule should be enabled on `juice-shop` via a network tag

* Run the following:
* Make sure you replace <SSH internal network tag> with the tag provided on the Left Pane.

```yaml
gcloud compute firewall-rules create <SSH internal network tag> --allow=tcp:22 --source-ranges 192.168.10.0/24 --target-tags <SSH internal network tag> --network acme-vpc
gcloud compute instances add-tags juice-shop --tags=<SSH internal network tag> --zone=us-central1-b
```

![4](https://github.com/user-attachments/assets/b906768b-bab2-4922-b92f-baa7b6d5a967)

![5](https://github.com/user-attachments/assets/651ba73d-869c-4b3b-ad90-1630dbd9f53c)

### Task 6: In the Compute Engine instances page, click the SSH button for the bastion host. Once connected, SSH to `juice-shop`

* Go to **Compute Engine** > **VM instances** > **SSH** to **bastion** host
* Run the following:

```yaml
ssh <internal IP of the juice-shop>
```
![6](https://github.com/user-attachments/assets/1bdd1578-2998-459d-b7df-c3a22eb06b7f)

![7](https://github.com/user-attachments/assets/9286825b-9c03-461f-a38b-62ea1857d669)


