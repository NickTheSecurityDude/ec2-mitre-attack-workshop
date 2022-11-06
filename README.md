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

Enumerate the domain to find common DNS records.
```
gobuster dns -w /home/ec2-user/pentesting-tools/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-small.txt  -d pentestingdemo.com
````

Now with the domains it found, enumerate those for common files:
```
gobuster dir -w /home/ec2-user/pentesting-tools/SecLists/Discovery/Web-Content/Common-PHP-Filenames.txt  -u http://finance.pentestingdemo.com
```


## Step 2 - Credential Access

Now go to the pages that it found.

On report.php we see we're able able to bypass authentication to access it:
http://finance.pentestingdemo.com/reports.php

Now, click on the first link:
http://finance.pentestingdemo.com/view_report.php?url=https://security-ace-public-files.s3.us-west-2.amazonaws.com/sa-lab/financials/xyz_corp_2022_Q3.csv

This takes us to a new page, not found by the enumerator: view_report.php.  Also notice that it has a variable: url

Try replacing that URL with the meta data URL.

Its blocked:

Take a look at this page for other options:
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Request%20Forgery/README.md#ssrf-url-for-cloud-instances

We see if we use instance-data, its not blocked:

Now try to get the temporary credentials:


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

