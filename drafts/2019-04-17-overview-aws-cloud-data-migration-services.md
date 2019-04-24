---
title: 'An Overview of AWS Cloud Data Migration Services'
date: 2019-04-17 00:00:00
excerpt: TL;DR on this AWS whitepaper.
categories: [tldr]
tags: [tldr, cloud, aws]
featured_image: '/images/feature/2.jpg'
#scroll_image: '/images/'
comments: true
share: true
---

Written May 2016 

** whenever possible, I have tried to include up-to-date information for anything that has changed since this whitepaper was published in May 2016. I cannot guarantee that exact details are reflective of the current state of AWS services **
# Abstract
- main concerns
  - how much data is being moved
  - how long the transfer will take over your existing internet connection
  - security level

- data migration tools come in 2 flavors
  - managed
    - connection speed more than 10Mbps
    - more than 500GB data
  - unmanaged
    - less than 10Mbps connection
    - less than 500GB data
    - this is basically the AWS CLI for S3 and Glacier using (possibly) multi-part upload


# Services 
- direct connect
  - dedicated, physical connction directly to AWS
  - traffic does not go over the internet
  - potential to reduce network costs, increase bandwidth throughput, and have more consistent service/performance
  - comes in 1gb or 10gb
  - direct connect in itself is not a data transfer service. however, it can help achieve security and performance requirements. 
  - you are only charged for what you use and there is no minimum fee. however, renting a box in a networking co-location has it's own costs. 


- snowball (import/export)
  - incredibly robust devices that are purpose built for data transfer
  - 50 TB or 80 TB
  - imports data into S3, with a maximum file size of 5TB (same as s3)
  - devices are shipped (1 or 2 day)
  - integrates with AWS KMS for encryption in-transit and at-rest
  - uses Trusted Platform Module (TPM) to detect any unauthorized modifications to the hardware, firmware, or software in order to physically secure the device
  - on-prem, transfer speed is dependent on your internal network. when the devices lands back at AWS, it is offloaded using AWS's high-speed internal network. 
  - once data is in s3, it can be copied to other services likes Glacier or EBS (elastic block storage)
  - *generally, if your data transfer will take a week or more, you should consider using Snowball*
  - scenarios where snowball is a preferred method:
    - you do not want to make expensive upgrades to network infrastructure
    - you frequently experience large backlogs of data
    - you are located in a physically isolated environment
    - you are in an area where high-speed internet is not available or cost-prohibitive
  - common use cases:
    - cloud migration 
    - disaster recovery
    - data center decommission
    - content distribution

- storage gateway
  - connects on-premise software appliance with cloud-based storage
    - is deployed as a VM (virtual machine)
      - VMware ESXi
      - Microsoft Hyper-V
      - EC2
    - is available as a hardware appliance as of September 2018
    - installed in data center or on EC2
    - data in-transit is encrypted using SSL. data at rest is encrypted using AWS KMS SS3-S3 (AES-256)
  - uses industry-standard storage protocols that work with your existing applications
  - 3 types of gateways: **_this whitepaper uses dated language. I have included notes on the most up-to-date documentation available_**
    - file gateway
      - presents a file-based interface to S3. appears as a network file share. 
      - s3 buckets will be available as Network File System (NFS) mounts or Server Message Block (SMB) file shares
      - recently used data is cached on the gateway for low-latency access
      - once data is in S3, you can manage it directly or use S3 features such as lifecycle policies, object versioning, or cross-region replication
    - volume gateway
      - enables you to create block storage volumes and mount them as iSCSI devices
      - runs in either cached or stored mode:
        - cached mode
          - primary data is written to s3
          - frequently accessed data is cached for low-latency access
          - supports up to 32 volumes. volumes can be a maximum of 32 TB, for a maximum of 1 PB data per cached volume gateway. 
        - stored mode 
          - primary data is stored locally and your entire dataset is available for low-latency access
          - dataset is asynchronously backed up to S3
          - supports up to 32 volumes. each volume can be up to 16 TB, for 512 TB maximum per stored volume gateway.
      - both modes support point-in-time snapshots of volumes. these are stored as EBS snapshots.
    - tape gateway
      - cloud-based Virtual Tape Library (VTL)
      - your backup application discovers virtual tapes using its standard media inventory procedure
      - tapes are available for immediate access and are backed by S3
      - tapes can also be archived in Glacier or S3 Glacier Deep Archive
      - When creating virtual tapes, you select one of the following sizes: 100 GB, 200 GB, 400 GB, 800 GB, 1.5 TB, and 2.5 TB. Please note, you only pay for the amount of data stored on each tape, and not for the size of the tape.
      - A single tape gateway can have up to 1,500 virtual tapes in the VTL with a maximum aggregate capacity of 1 PB

- S3 Transfer Acceleration (S3-XA)
  - routes uploads to an AWS edge location rather than directly to s3
  - helps mitigate the effect of distance on network throughput
  - there is an online speed comparison tool to measure the benefits of using S3 Transfer Acceleration
  - an s3 bucket with XA enabled will route traffic from any user, anywhere in the world, to the nearest edge location
  - only pay for what you use

- AWS Kinesis Firehose
  - used for capturing and loading streaming data into s3, redshift, and elasticsearch
  - fully managed. automatically scales to meet throughput needs
  - can batch, compress, and encrypt data before loading
  - only pay for what you use


# Use Cases


























































































