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
Google Cloud environment, Team Griffin Infrastructure diagram

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
