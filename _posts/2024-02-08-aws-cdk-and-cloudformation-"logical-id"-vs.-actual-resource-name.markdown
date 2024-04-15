---
title: AWS CDK and CloudFormation "Logical ID" vs. actual resource name
date: 2024-02-08 11:18
classes: wide
use_math: true
categories: infra
---

When I started out with CDK I was confused by the ubiquitous `id` parameter that all CDK resources take. I now realise
this is just a handle for identifying resources in the eventual CloudFormation template tree. It **is not** the name of
the actual resource as it exists as eventual AWS infra.

For example:

```python
bucket = s3.Bucket(self,
                   "MyFirstBucket",
                   bucket_name="my-bucket-name",
                   versioned=True)
```

Here `MyFirstBucket` is the CloudFormation handle used in the resource tree in CloudFormation AWS console. However,
`bucket_name` sets the actual S3 bucket name; in this example `s3://my-bucket-name`.
