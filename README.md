# NetflixOSS Ansible Playbooks

[Ansible](https://github.com/ansible/ansible/) is a configuration management system that is [very simple to learn](http://www.ansibleworks.com/docs/gettingstarted.html) and use.

This project is a set of Ansible Playbooks to configure instances to run some of the [NetflixOSS](http://netflix.github.io/) projects.

- [Asgard](#asgard)
- Aminator (coming soon)
- [Edda](#edda)
- [Eureka](#eureka)
- Ice (coming soon)
- Simian Army (coming soon)

## Prerequisites

1. [Ansible installed](http://www.ansibleworks.com/docs/gettingstarted.html) on your laptop
1. The [Ansible EC2 Inventory](http://www.ansibleworks.com/docs/api.html/#example-aws-ec2-external-inventory-script) configured
1. Clone this repository

## Features

These playbooks are built to be run on the following operating systems:
- Ubuntu 12.04 LTS
- Amazon Linux

They have also been written in a way where you can use the same playbook to configure a running server, or build a custom AMI.

### Base configuration

The base configuration is gets a system ready for production. You can find the [base tasks here](https://github.com/awsanswers/noss-ansible/tree/master/playbooks/roles/base/tasks), but in summary it:
- installs some packages
 - Python with the latest Boto
 - AWS CLI
 - security packages including fail2ban
 - Emacs and Vim (no religion here)
- does some basic system hardening

## Projects

### Asgard

[Asgard](https://github.com/Netflix/asgard) is an application deployments and cloud management web interface for AWS. Before running the playbook, there are a few things we need to do:

1. Create an Asgard security group
 - Allow port 22 for SSH
 - Allow port 80 for HTTP
1. If you don't already, create a new Key pair, and add it to your keychain or SSH agent so you don't need to specify it later:
```
$ ssh-add mykey.pem
```
1. Launch a new EC2 instance using the above Security Group and key pair. Use Ubuntu 12.04 LTS as the AMI.
1. Set the `Name` tag of the instance to `Asgard`
1. Confirm you can see the instance using the Ansible EC2 inventory
```
$ /etc/ansible/hosts | grep 'Asgard'
```

Now you can run the playbook

```
$ ansible-playbook playbooks/asgard-ubuntu.yml -l 'tag_Name_Asgard'
```

This will configure the instance to be running the [latest snapshot build](https://netflixoss.ci.cloudbees.com/job/asgard-master/lastSuccessfulBuild/artifact/target/) of Asgard. If you prefer to run the stable version of Asgard, you want to build the WAR file yourself, just specify the path to the WAR file:

```
$ ansible-playbook playbooks/asgard-ubuntu.yml -l 'tag_Name_Asgard' -e "local_war=$HOME/Downloads/asgard.war"
```

Once the playbook is finished, you will have Asgard running inside Tomcat on your EC2 instance. You can access Asgard via HTTP as the ROOT application (no directory). Example:

```
http://ec2-54-245-157-159.us-west-2.compute.amazonaws.com/
```

### Eureka

[Eureka](https://github.com/Netflix/eureka) is a service registry for resilient mid-tier load balancing and failover. Before running the playbook, there are a few things we need to do:

1. Create a Eureka security group
 - Allow port 22 for SSH
 - Allow port 80 for HTTP
1. If you don't already, create a new Key pair, and add it to your keychain or SSH agent so you don't need to specify it later:
```
$ ssh-add mykey.pem
```
1. Launch a new EC2 instance using the above Security Group and key pair. This time, we will use Amazon Linux.
1. Set the `Name` tag of the instance to `Eureka`
1. Confirm you can see the instance using the Ansible EC2 inventory
```
$ /etc/ansible/hosts | grep 'Eureka'
```

Now you can run the playbook

```
$ ansible-playbook playbooks/eureka-amazon-linux.yml -l 'tag_Name_Eureka'
```

This will configure the instance to be running the [latest snapshot build](https://netflixoss.ci.cloudbees.com/job/eureka-master/lastSuccessfulBuild/artifact/eureka-server/build/libs/eureka-server-1.1.98.war) of Eureka. If you prefer to build your own WAR file yourself, just specify the path to the WAR file:

```
$ ansible-playbook playbooks/eureka-amazon-linux.yml -l 'tag_Name_Eureka' -e "local_war=$HOME/Downloads/eureka-server.war"
```

Once the playbook is finished, you will have Eureka Server running inside Tomcat on your EC2 instance. You can access it via HTTP. Example:

```
http://ec2-54-245-157-159.us-west-2.compute.amazonaws.com/eureka/
```

### Edda

[Edda](https://github.com/Netflix/edda) is a service to track changes in an AWS region, multiple regions and/or multiple accounts. Before running the playbook, there are a few things we need to do:

1. Create an Edda security group
 - Allow port 22 for SSH
 - Allow port 80 for HTTP
1. If you don't already, create a new Key pair, and add it to your keychain or SSH agent so you don't need to specify it later:
```
$ ssh-add mykey.pem
```
1. Create an [IAM Role](https://console.aws.amazon.com/iam/home?#roles) called 'edda' with [this policy](https://github.com/Netflix/edda/wiki/AWS-Permissions#example-policy)
1. Launch a new EC2 instance using the above Security Group, key pair and IAM Role. You can use with Ubuntu or Amazon Linux for the OS.
1. Set the `Name` tag of the instance to `Edda`
1. Confirm you can see the instance using the Ansible EC2 inventory
```
$ /etc/ansible/hosts --refresh-cache | grep 'Edda'
```

Now you can run the playbook

```
$ ansible-playbook playbooks/edda-amazon-linux.yml -l 'tag_Name_Edda'
 or
$ ansible-playbook playbooks/edda-ubuntu.yml -l 'tag_Name_Edda'
```

This will configure the instance to be running the [latest snapshot build](https://netflixoss.ci.cloudbees.com/job/edda-master/lastSuccessfulBuild/artifact/build/libs/edda-2.1-SNAPSHOT.war) of Edda. If you prefer to build your own WAR file yourself, just specify the path to the WAR file:

```
$ ansible-playbook playbooks/edda-amazon-linux.yml -l 'tag_Name_Edda' -e "local_war=$HOME/Downloads/edda.war"
```

Once the playbook is finished, you will have Edda running inside Tomcat with MongoDB on your EC2 instance. You can access then [make queries to it via HTTP](https://github.com/Netflix/edda/wiki/REST). Example:

```
http://ec2-12-212-12-121.us-west-2.compute.amazonaws.com/edda/api/v2/view/instances;_pp
```

_NOTES_:

1. This is not production quality. If the instance dies, you loose your history. This is meant as a quick way to get Edda up and running and see if you like it. Have a look at [this wiki page for running Edda in production](https://github.com/Netflix/edda/wiki/Resiliency).
1. The default configuration for Edda will only look at the `us-east-1` region. You can change `edda.region` config parameter (and other [configuration settings](https://github.com/Netflix/edda/wiki/Configuration#wiki-eddaregion)) by editing `/usr/local/tomcat/webapps/edda/WEB-INF/classes/edda.properties`.
## Feedback

If you have feedback, comments or suggestions, please feel free to contact Peter at AWS Answers, create an Issue, or submit a pull request.

## About AWS Answers

These playbooks were written by [Peter Sankauskas](https://twitter.com/pas256), founder of [AWS Answers](http://awsanswers.com/) - a consulting company focused on helping business get the most out of AWS. If you are looking for help with AWS, please [contact us](http://awsanswers.com/contact/). 

## LICENSE

Copyright 2013 AWS Answers LLC

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0 Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
