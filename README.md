# AWS VPC management

## Objective
Manage all aspect of AWS VPC ( virtual private cloud ) programatically via ansible

## Requirements
- linux environment such as centOS
- python 2.7 minimum
- pip boto and ansible
- AWS key pair ( EC2 > Network & security > Key Pairs )
- IAM user with privileges to manage vpc
  - AWS_ACCESS_KEY_ID
  - AWS_SECRET_ACCESS_KEY
- ami id available in current region
  - nat instance
  - regular instance such as Centos7

If you do not have the ami, search aws console (IMAGEs > AMIs > Public images )
Select the ami, click Actions > Copy AMI

## Overview
role vpc
- Configure vpc
- Setup environment
  - create prod and dev environment subnets
  - private and public subnets
  - 2 nat instance for private instance access to internet and back up external access to private subnets
  - basic security groups for external, public and private subnets

role instance
- Create instances
- Add DNS to instances


## Setup environment

For new setup:

```sh
wget --no-check-certificate https://bootstrap.pypa.io/get-pip.py && python get-pip.py
rm -f get-pip.py
git clone this project to your host server
pip install -r requirements.txt
```

# Configure environment

You need to configure the environment to point to the correct aws account
Contact sysops to get the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
 

```sh
vi ~/.boto
```

[profile prod]
AWS_ACCESS_KEY_ID=xxxx
AWS_SECRET_ACCESS_KEY=yyyy

[profile qa]
AWS_ACCESS_KEY_ID=xxxx
AWS_SECRET_ACCESS_KEY=yyyy

active profile
```sh
export AWS_PROFILE=prod
```

# Assumptions

You already created vpc with tag {{aws_environment}} e.g useast1
10.8.0.0/16 tag useast1

Already configured VPN Connection note the vpngateway_id 

# Configure vpc

edit group_vars/all
edit env_vars/useast1.yml


This script creates the entire VPC along with subnets, nat instances, routing and security groups.
```sh
ansible-playbook -i inventory/aws -e aws_environment=useast1 vpc.yml
```

# Define instances

edit roles/instances/vars/[prod/dev].yml

If your instance type does not exist, create a new instance type block:
```
  - name: app
    instances: 2
    subnet: app
    instance_type: t2.nano
    ami_image: "{{centos7_ami}}"
    volumes:
      - device_name: /dev/sda1
        volume_type: gp2
        volume_size: 40
```
If the instance type exists you can add additional instances of same type by incrementing "instances"


# Create instances

- vi roles/instances/var/[prod/dev].yml

Insure your instance count and instance type is corrrect.

- generate template
ansible-playbook -i inventory/aws instances.yml --tags=query -e aws_environment=[region]

region: useast1, uswest1

```sh
ansible-playbook -i inventory/aws instances.yml --tags=query -e aws_environment=useast1 
```

- create instances
```sh
ansible-playbook -i inventory/aws instances.yml --tags=build -e aws_environment=useast1
```

#debug

Add the following commandline options:
```
-vv
-e debug_play=true

``` 
# remove instances

- To insure automation does not accidentally remove instances, use the AWS console to remove instances

edit roles/instances/vars/[prod/dev].yml

Either remove the block of code, or make the 'instances: 0'
```
  - name: oldapp
    instances: 0
    subnet: app
    instance_type: t2.nano
    ami_image: "{{centos7_ami}}"
    volumes:
      - device_name: /dev/sda1
        volume_type: gp2
        volume_size: 40
```

