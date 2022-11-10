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

<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step-0-1.png" width="800">
  
View the domain in Registered Domains and Edit the Name Server to match what was in the hosted zone.

<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step-0-2.png" width="800">

Wait at least 15 minutes for the nameservers to propagate (this may take up to 24 hours).


## Step 1 - Reconnaissance

Enumerate the domain to find common DNS records.
```
gobuster dns -w /home/ec2-user/pentesting-tools/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-small.txt  -d pentestingdemo.com
````

![Search DNS](https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step1-1.png" width="600">


Now with the domains it found, enumerate those for common files:
```
gobuster dir -w /home/ec2-user/pentesting-tools/SecLists/Discovery/Web-Content/Common-PHP-Filenames.txt  -u http://finance.pentestingdemo.com
```

<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step1-2.png" width="800">

## Step 2 - Credential Access

Now go to the pages that it found.

<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step2-1.png" width="600">

On report.php we see we're able able to bypass authentication to access it:
http://finance.pentestingdemo.com/reports.php

Now, click on the first link:
http://finance.pentestingdemo.com/view_report.php?url=https://security-ace-public-files.s3.us-west-2.amazonaws.com/sa-lab/financials/xyz_corp_2022_Q3.csv

This takes us to a new page, not found by the enumerator: view_report.php.  Also notice that it has a variable: 

<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step2-2.png" width="800">

Try replacing that URL with the meta data URL.

Its blocked:
<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step2-3.png" width="800">

Take a look at this page for other options:
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Request%20Forgery/README.md#ssrf-url-for-cloud-instances

We see if we use instance-data, its not blocked:

<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step2-4.png" width="800">

Now try to get the temporary credentials:

<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step2-5.png" width="800">

## Step 3 - Discovery

Now that we have credentials, lets see what else we can find in the account.

First enumerate the account with ScoutSuite:
```
virtualenv -p python3 venv
source venv/bin/activate
scout aws --max-workers 3
```

Now view the report:
python3 -m http.server 8001

One more tool we can use for enumerating is CloudMapper.

Edit the config file:
```
cd pentesting-tools/cloudmapper/
nano config.json
```

<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step3-cloudmapper-1.png" width="600">

Then run cloudmapper:
```
python3 -m venv ./venv && source venv/bin/activate
python cloudmapper.py collect --account securityace 
python3 cloudmapper.py report --account securityace
python3 cloudmapper.py prepare --account securityace
python3 cloudmapper.py webserver --public
```

The last command starts a webserver on port 8000 so you can view the Cloud Mapper results:
http://your-tools-ip:8000

<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step3-cloudmapper-2.png" width="800">

## Step 4 - Lateral Movement

Now with the data from Scout Suite and Cloud Mapper try to move laterally within the AWS account.

Pay particular attention to the resources ScoutSuite identitifed as vulnerable.


## Step 5 - Privilege Escalation

Using a tool called Pacu can be good way to escalation your privileges.

Pacu can be installed as so:
```
mkdir pacu && cd pacu
python3 -m venv venv && source venv/bin/activate
python3 -m pip install --upgrade pip
Pip3 install pacu
```

To run it, first set the keys:

<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step5-1.png" width="800">

Then you can enumerate IAM resources:
<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step5-2.png" width="800">

And then try to have it automatically escalat your privilege:
<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step5-3.png" width="800">

In addition to Pacu, you should also try to manually escalate your privilege.

Starting as the web server user:
<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step5-5.png" width="800">

You can list all roles that have allow "root" in the local account to assume it, and view the details of them:
<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step5-5b.png" width="800">

Then you can check the attached and inline policies attached to that account:
<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step5-6.png" width="800">

Since any local account (with sts:AssumeRole permissions) can assume it, switch to that user:
<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step5-7.png" width="800">

Try a priviledged command or two, notice create-user was blocked by create-role, as well as attach-role works:
<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step5-8.png" width="800">

Now, try create-user again, this time it works:
<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step5-9.png" width="800">

## Step 6 - Defense Evation

The following techniques can be used for Defense Evasion:
- Delete CloudWatch Log Group
- Delete CloudTrail
- Stop CloudTrail Logging
- Tamper with CloudTrail settings
- Tamper with Security Services, ex. AWS Config, SecurityHub, etc.
- Delete Logs from S3
- PutBucketLifeCycle with short expiration on logs

## Step 7 - Persistance

There are multiple ways to keep persistance access to the account, in the event the SSRF vulnerability from Step 2 is fixed.

For example you may want to create a new user or add an additional set of access keys for an existing user.

## Step 8 - Collection

Now that you have a foothold you can collect data, such as any crown jewel assets or look for sensitive info like access keys.

Common services to target here are S3 and CodeCommit.

A good tool to use for this is TruffleHog, this is particually useful on git/codecommit repos because it can quickly check all branches.

```
trufflehog git https://github.com/NickTheSecurityDude/code-commit-test-repo.git |grep -E "Detector|File"
```

<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step8-1.png" width="800">

## Step 9 - Exfiltration 

Now you can exfiltrate data out of this account.  

This may include:
- Sharing EC2 SnapShots with your account
- Sharing RDS SnapShots with your account
- Copying an S3 bucket to a bucket you own

## Step 10 - Impact

A common way to leave your mark when doing a pen test is by updating the tags of resources.  (Ensure the client is OK with this before preceeding).

For example you can modify the CloudFormation stack to add a name tag, indicating the account was compromised.

<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step10-1.png" width="800">

And when you view resources in the console you will see your mark.

<img src="https://security-ace-public-files.s3.us-west-2.amazonaws.com/workshop-images/step10-2.png" width="600">

## Step 99 - Cleaning up

- Terminate the stack called: ec2-mitre-workshop.
- Manaually delete any resources causing a rollback and repeat the first step.
- You will be charged for any resources left behind.

