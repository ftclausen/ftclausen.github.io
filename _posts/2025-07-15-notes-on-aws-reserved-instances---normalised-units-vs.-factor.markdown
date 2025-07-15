---
title: "Notes on AWS Reserved Instances - Normalised units vs. factor"
date: 2025-07-15 11:53
classes: wide
use_math: true
categories: aws
---

## Reserved Instances Normalised Units and Normalisation Factor

I had some confusion about exactly how Reserved Instances and just jotting down some notes here to try and cement my
understanding. For a little while anyway :D.

The key is noting the different between **Normalised Units** and **Normalisation Factor**. As I understand it, they are:

- **Normalised Units** - These are an abstract cost to run a given instance per hour. For example,  a `small` size of
instance costs 1 normalised unit per hour.
- **Normalisation Factor** - Now, obviously, there are many kinds of instances and they can't all cost the same. So the
  "base level" of `small` costing 1 unit per hour is them multiplied by a Normalisation Factor which can be < 1 for
small instances and > 1 for most instances.

In other words, the normalisation factor is a scaling factor applied from the "base unit" of `small`. It says how fast
an instance will use up its "normalised fuel".

## Examples

[These examples](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/apply_ri.html#ri-usage-examples) are quite useful
as an exercise to try and work it out yourself. Another one is:

- Currently running 9 instances of `cache.r7g.xlarge` and Billing and Cost Management Recommendation is to purchase 18 cache.r7g.large
- `large` normalisation factor is 4. Thus the recommendation is for 18 * 4 = 72 units per hour
- `xlarge` normalisation factor is 8. Thus  we are using 9 * 8 = 72 units per hour

*Therefore* the recommendation is spot on and will provide us with 100% coverage.

Here is this relationship of `large` vs `xlarge` in a diagram:

![cache-r7g-large-units](/images/cache-r7g-norm-factor.png | width=500)

## References

- [Instance Size
Flexibility](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/apply_ri.html#ri-instance-size-flexibility) - See
normalisation factor table and diagrams that follow.

