---
id: 01J5HDXGER568CW5KAKCQ9XM7H
title: TiDB Cloud Notes
tags:
  - tidb
  - hackathon
  - machine-learning
  - artificial-intelligence
  - cloud
  - db
  - 2024-journal
description: Learning how to use TiDB Cloud
modified: 2024-08-17T20:27:11-04:00
---
# TiDB Cloud Notes
- [TiDB Cloud](https://www.pingcap.com/tidb-cloud/) is a fully-managed Database-as-a-Service (DBaaS) that brings [TiDB](https://docs.pingcap.com/tidb/stable/overview), an open-source Hybrid Transactional and Analytical Processing (HTAP) database, to your cloud. TiDB Cloud offers an easy way to deploy and manage databases to let you focus on your applications, not the complexities of the databases. You can create TiDB Cloud clusters to quickly build mission-critical applications on Google Cloud and Amazon Web Services (AWS).

![TiDB Cloud Overview](https://download.pingcap.com/images/docs/tidb-cloud/tidb-cloud-overview.png)

## Why TiDB Cloud
- TiDB Cloud allows you with little or no training to handle complex tasks such as infrastructure management and cluster deployment easily.
- Developers and database administrators (DBAs) can handle a large amount of online traffic effortlessly and rapidly analyze a large volume of data across multiple datasets.
- Enterprises of all sizes can easily deploy and manage TiDB Cloud to adapt to your business growth without prepayment video to learn more about TiDB Cloud:
- With TiDB Cloud, you can get the following key features:

- **Fast and Customized Scaling**
    
    Elastically and transparently scale to hundreds of nodes for critical workloads while maintaining ACID transactions. No need to bother with sharding. And you can scale your performance and storage nodes separately according to your business needs.
    
- **MySQL Compatibility**
    
    Increase productivity and shorten time-to-market for your applications with TiDB's MySQL compatibility. Easily migrate data from existing MySQL instances without the need to rewrite code.
    
- **High Availability and Reliability**
    
    Naturally high availability by design. Data replication across multiple Availability Zones, daily backups, and auto-failover ensure business continuity, regardless of hardware failure, network partition, or data center loss.
    
- **Real-Time Analytics**
    
    Get real-time analytical query results with a built-in analytics engine. TiDB Cloud runs consistent analytical queries on current data without disturbing mission-critical applications.
    
- **Enterprise Grade Security**
    
    Secure your data in dedicated networks and machines, with support for encryption both in-flight and at-rest. TiDB Cloud is certified by SOC 2 Type 2, ISO 27001:2013, ISO 27701, and fully compliant with GDPR.
    
- **Fully-Managed Service**
    
    Deploy, scale, monitor, and manage TiDB clusters with a few clicks, through an easy-to-use web-based management platform.
    
- **Multi-Cloud Support**
    
    Stay flexible without cloud vendor lock-in. TiDB Cloud is currently available on AWS and Google Cloud.
    
- **Simple Pricing Plans**
    
    Pay only for what you use, with transparent and upfront pricing with no hidden fees.
    
- **World-Class Support**
    
    Get world-class support through our support portal, [email](mailto:tidbcloud-support@pingcap.com), chat, or video conferencing.
    

## [](https://docs.pingcap.com/tidbcloud/tidb-cloud-intro#deployment-options)Deployment options

TiDB Cloud provides the following two deployment options:

- [TiDB Serverless](https://www.pingcap.com/tidb-serverless)
    
    TiDB Serverless is a fully managed, multi-tenant TiDB offering. It delivers an instant, autoscaling MySQL-compatible database and offers a generous free tier and consumption based billing once free limits are exceeded.
    
- [TiDB Dedicated](https://www.pingcap.com/tidb-dedicated)
    
    TiDB Dedicated is for production use with the benefits of cross-zone high availability, horizontal scaling, and [HTAP](https://en.wikipedia.org/wiki/Hybrid_transactional/analytical_processing).
    

For feature comparison between TiDB Serverless and TiDB Dedicated, see [TiDB: An advanced, open source, distributed SQL database](https://www.pingcap.com/get-started-tidb).
## Architecture
![[Pasted image 20240817202700.png]]