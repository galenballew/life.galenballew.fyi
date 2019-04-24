---
title: 'Everything About AWS EC2 Systems Manager'
date: 2019-02-24 00:00:00
excerpt: Exploring all 7 microservices offered in AWS SSM. 
categories: [cloud]
tags: [aws, s3, certification, devops, cicd]
featured_image: '/images/demo/demo-landscape.jpg'
---
# Abstract

https://www.aws.training/learningobject/curriculum?id=13830

---

# Run Command
Run command is a SSM service that allows you to securely perform configuration actions on your EC2 instances and on-prem virtual machines at scale. 

All managed instances require the SSM Agent to be installed. 

A Document is a series of steps that are executed in sequence - think Ansible playbook. In fact, you can literally use [Ansible Playbooks](https://aws.amazon.com/blogs/mt/running-ansible-playbooks-using-ec2-systems-manager-run-command-and-state-manager/). AWS provides pre-made Documents for common tasks and you can create your own custom documents. Documents can be versioned and shared across accounts as well. 

Run Command offers numerous features to identify and target specific groups of instances when deploying to a large fleet. It also has features like `max-errors` and `max-concurrency` to ensure that Documents are only being run on a limited number of instances at any given time. This can help reduce risk associated with patching or new deployments. 

Use cases:
    - Monitoring systems
    - Join instances to a domain
    - On-demand patching
    - Deploying code to instances
    - Bootstrapping instances
    - Starting/stopping processes on instances
    - User and account management (creating locals users, groups, permissions, etc)

Key benefits:
    - SSM Run Command is fully managed and offered at no additional charge. 
    - It provides a single view for all configuration tasks.
    - You no longer need SSH and RDP.
    - Integrates with IAM. 
    - CloudTrail gives full audit trail on all actions taken.

---

# State Manager
State Manager is a tool that gives control over how and when configurations are applied. State Manager is similar to Run Command, but offers the additional capability to apply configurations repeatedly on a set schedule. This helps to combat configuration drift and bolsters compliance. 

---

# Inventory
Inventory collects:
    - instance details
    - OS details
    - network configuration
    - installed software and patches

Inventory can be customized to collect any information you want. 

Common use cases:
    - Keeping track of the number of licenses deployed 
    - Identifying unpatched/vulnerable instances
    - Track configuration changes over time

Inventory works seamlessly with State Manager to collect inventory Policies from targeted instances on a given schedule. 

Inventory also ties into Config to record changes over time.


---

# Maintenance Window
Maintenance Window lets you define a schedule for when to perform potentially disruptive actions on your instances. Example actions are patching the OS, updating drivers, or installing software.

Each maintenance window has the following properties:
    - Schedule
      - When to run
    - Duration
      - How long to run for
    - Set of registered targets
      - What to run on
    - Set of registered tasks
      - Each task is given a subset of the Duration

---

# Patch Manager
Patch Manager is an automated tool that simplifies patching instance operating systems. Patch Manager helps in selecting which patches to deploy, timing for patch roll-outs, and controlling instance reboots. You can create policies to auto-approve new patches, or black/white list specific patches. 
---

# Automation
Automation exists to help streamline the process of creating new, up-to-date AMIs. Automations exist as Documents are are tightly integrated with the other services offered through Systems Manager. 

---

# Parameter Store
Parameter Store is a single location to store and retrieve credentials or other bits of data. By having it in a single location, it only needs to be updated in a single place, despite possibly being used in tens to thousands of places. Parameter Store offers full encryption, as well as integration with IAM for fine-grain access control and an complete audit trail.

Pro Tip: CloudFormation does not provide a way to define encrypted parameters because they would be stored as plaintext in the template. However, it's quite easy to create a [Custom Resource](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html) to create a Parameter Store SecureString that consumes a parameter from your template (be sure to use the `NoEcho` property on your [parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html)!) For extra credit, you could turn your Lambda Custom Resource into a [Macro](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-macros.html).
---