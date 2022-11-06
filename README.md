# EC2 Mitre Attack Workshop

In this workshop you will simulate an AWS account breach from start to finish, starting with reconnaissance, ending with impact, and 8 steps in between such as credential access, privilege escalation and exfiltration.

To get started you will need:
- an AWS Account (please use a sandbox account)
- your IPv4 source IP (you can find that here: https://www.whatismyip.com/)
- a domain (or subdomain provided by the workshop)

Click the button to launch the resources in the us-west-2:

[![CloudFormation Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png "Launch Workshop Stack")](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=ec2-mitre-workshop&templateURL=https://security-ace-public-files.s3.us-west-2.amazonaws.com/templates/sa-lab-ROOT.yaml) 

You will only need to enter the two required parameters.
Only change the VPC subnet cidr, if you already have a VPC running with that range.

**Disclaimer this stack creates vulnerable resources as well as resources which incur costs from AWS, terminate when not using**

## Step 0 - Point the Domain

View the hosted zone in Route 53 and look for the ns record.

View the domain in Registered Domains and Edit the Name Server to match what was in the hosted zone.

Wait at least 15 minutes for the nameservers to propagate (this may take up to 24 hours).


## Step 1 - Reconnaissance



## Step 2 - Credential Access



## Step 3 - Discovery



## Step 4 - Lateral Movement



## Step 5 - Privilege Escalation



## Step 6 - Defense Evation



## Step 7 - Persistance



## Step 8 - Collection



## Step 9 - Exfiltration 



## Step 10 - Impact



Cleaning up:
- Terminate the stack called: ec2-mitre-workshop.
- Manaually delete any resources causing a rollback and repeat the first step.
- You will be charged for any resources left behind.

