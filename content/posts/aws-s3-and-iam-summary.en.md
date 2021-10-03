---
title: "Aws S3 and Iam Summary"
date: 2021-10-03T09:21:59+09:00
draft: false
author: Torres
tags: ["S3", "IAM"]
categories: ["AWS"]
toc:
  enable: true
  auto: true
share:
  enable: true
comment:
  enable: true

---

# IAM - Identity Access Management

IAM consists of the followings:

- Users
- Groups
- Roles
- Policies



We can use follow json data to custom policies:

```json
{
  "Version": "2020-03-14",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```

## IAM Knowledge Point

- IAM is universal. Does not apply to regions at this time
- The “root account” is simply the account created when first setip your AWS account. It has complete Admin access.
- New Users have No Permissions when first created
- New Users are assigned **Access Key ID** and **Access Keys** when first created
- **Access Key ID** and **Access Keys** are not same as passwords. We can not use them to login in to the AWS console. But we can use them to access AWS via then APIs nad Command Line.
- We can only view **Access Keys** once. If we lose them, we have to regenerate them.
- IAM requires us to setup Mutifactor Authentication on our root account
- We can create and customise our own password rotation policies

# S3- Simple Storage Service

## S3 Knowledge Point

- S3 is **Object-based**, allows us to upload files.
- Files can be from 0 Bytes to 5 TB.
- unlimited storage.
- Files are stored is Buckets.
- **S3 is a universal namespace**. So the name of buckets have to be unique globally.
- This is what the S3 url looks like: [https://mscloudplatform.s3.amazonaws.com](https://mscloudplatform.s3.amazonaws.com/)
- Not suitable to install an operation system on like EBS.
- Successful uploads will generate a HTTP 200 status code.

By default, all newly created buckets are **PRIVATE**. We can setup access control to our buckets using:

- Bucket Policies
- Access Control Lists

S3 Buckets can be configured to create access logs which log all requests made to the S3 buckets. This can be sent to another bucket and even another bucket in another account.

## The Key Fundamentals of S3

- Key (This is simply te name of the object)
- Value (This is simply the data and is made up of a sequence of bytes)
- Version ID (Important for versioning)
- Metadata (Data about data you are storing)
- Subresources:
  - Access Control Lists
  - Torrents
- **Read after Write consistency for PUTS of new Objects**
- **Eventual Consistency for overwrite PUTS and DELETES (can take some time to propagate)**



![S3 Storage Level](/images/posts/aws-s3-and-iam-summary/s3_iam_1.png)

**S3 Storage Level**



There are some different level of S3 storage we can choose when we upload Objects to S3 Buckets.

## Understand how to get the best value out of S3

Storage Cost (High to Low):

1. S3 Standard
2. S3 - IA (Infrequently Accessed)
3. S3 - Intelligent Tiering
4. S3 One Zone - IA (not recommend)
5. S3 Glacier
6. S3 Glacier Deep Archive

## S3 Encryption

**Encryption in Transit is achieved by:**

- SSL/TLS

**Encryption at Rest (Server Side) is achieved by**

- S3 managed Keys - SSE-S3
- AWS Key Management Service, Managed Keys - SSE-KMS
- Server Side Encrytion With Customer: Provided Keys - SSE-C

**Client Side Encryption**

## Best Practices with AWS Organizations(Enterprise only)



![Best Practices with AWS Organizations](/images/posts/aws-s3-and-iam-summary/s3_iam_2.png)

**Best Practices with AWS Organizations**



## Share S3 buckets across accounts

- Using Bucket Policies & IAM (applies across the entire bucket). Programmatic Access Only (APIs 访问).
- Using Bucket ACLs & IAM (individual objects). Programmatic Access Only.
- Cross-account IAM Roles. Programmatic and Console access.

## Cross Region Replication

- Versioning must be enbaled on both the source and destination buckets.(版本控制必须要在source和target同时启用)
- Regions must be unique.
- Files in an existing bucket are not replicated automatically. (已经存在于桶的文件不会自动replicate)
- All subsequent updated files will be replicated automatically. (所有之后的文件上传或者更新都会自动replicate)
- Delete markets are not replicated. (删除标记不会replicate)
- Deleting individual versions or delete markers will not be replicated. (删除单独的版本或删除delete markers不会被replicate)
- Understand what Cross Region Replication is at a high level.

## Lifecycle Policies

- Automates moving your objects between the different storage tiers. (可以将你的objects自动在不同的存储级别之间移动)
- Can be used in conjuction with versioning. (可以和版本控制结合使用)
- Can be applied to current versions and previous versions.(可以应用于当前版本和旧版本)

# CloudFront

## CloudFront Knowledge Point

- **Edge Location** - This is the location where content will be cached. This is separate to an AWS Region/AZ (content被缓存的地方，独立于AWS地区和可用区)
- **Origin** - This is the origin of all the files that the CDN will distribute. This can be either an S3 Bucket, an EC2 Instance, an Elastic Load Balancer, or Route53. (这是CDN分发的所有文件资源的来源。可以是S3 Bucket，可以是一个EC2实例，也可以是ELB，或者Route53)
- **Distribution** - This is the name given the CDN which consists of a collection of Edge Locations. (这是给定的由Edge Locations集合组成的CDN的名称)
- **Web Distribution** - Typically used for Websites. (通常用于网站)
- **RTMP** - Used for Media Streaming.
- Edge locations are not just **READ** only - you can write to them too. (Edge locations 不止是只读，我们也可以将数据写入edge location)
- Objects are cached for the life of the **TTL**(Time To Live). (对象只在TTL的生命周期中被缓存)
- You can clear cached objects, but you will be charged. (我们可一清楚缓存的对象，但是需要收费)

# Storage Gateway

There are some kinds of gateways:

**File Gateway**

- For flat files, stored directly on S3

**Volume Gateway**

- **Stored Volumes** - Entire Dataset is stored on site and is aynchronously backed up to S3. (**存储卷** - 整个数据集都存储在站点上并异步备份至S3)
- **Cached Volumes** - Entire Dataset is Stored on S3 and the most frequently accessed data is cached on site. (**缓存卷** - 整个数据集都存储在S3中，最常访问的数据被缓存在站点上)

**Gateway Virtual Tape Library**

- USed for backup and uses popular backup applications like NetBackup, Backup Exec, Veeam etc.
