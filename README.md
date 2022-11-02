# EC2 Mitre Attack Workshop

In this workshop you will simulate an AWS account breach from start to finish, starting with reconnaissance, ending with impact, and a number of steps in between such as credential access, privilege escalation and exfiltration.

To get started you will need:
- an AWS Account (please use a sandbox account)
- your IPv4 source IP
- a domain (or subdomain provided by the workshop)

Click the button to launch the resources in the us-west-2:

[![CloudFormation Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png "Launch Workshop Stack")](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=ec2-mitre-workshop&templateURL=https://security-ace-public-files.s3.us-west-2.amazonaws.com/templates/sa-lab-ROOT.yaml) 

You will only need to enter the two required parameters.
Only change the VPC subnet cidr, if you already have a VPC running with that range.

**Disclaimer this stack creates vulnerable resources as well as resources which incur costs from AWS, terminate when not using**

Cleaning up:
- Terminate the stack called: ec2-mitre-workshop.
- Manaually delete any resources causing a rollback and repeat the first step.
- You will be charged for any resources left behind.
