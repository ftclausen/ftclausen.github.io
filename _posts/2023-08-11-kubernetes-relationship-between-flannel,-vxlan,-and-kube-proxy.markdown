---
title: "Kubernetes: Relationship between Flannel, VXLAN, and kube-proxy"
date: 2023-08-11 11:45
classes: wide
use_math: false
categories: kubernetes
---

Very short post to put down my understanding of some Kubernetes topics. May elaborate later.

- Flannel (L3): "L3 network fabric". Allocates IP ranges. Doesn't forward packets, done by...
- VXLAN (L2 in L3, like VLANs): Encapsulates Ethernet frames in packets to be sent over the network to other hosts. Creates single logical network that
    "overlays" the physical and makes it look like the pods are all in the same, directly connected network.
- kube-proxy (L4): "Reflects" TCP ports around to make it look like, externally to a pod, that services are running on
    their standard ports. For example, an app might bind to port 443 in a pod and other apps can access it on port 443
    via a Service. *However*, kube-proxy is actually "listening" on port 443 on some IP then reflecting the requests to
    an arbitrary port associated with the Pod. This is commonly done with kernel iptables NATing
