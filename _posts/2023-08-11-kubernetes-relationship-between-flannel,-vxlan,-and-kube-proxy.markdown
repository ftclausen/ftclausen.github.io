---
title: "Kubernetes: Relationship between Flannel, VXLAN, and kube-proxy"
date: 2023-08-11 11:45
classes: wide
use_math: false
categories: kubernetes
---

Very short post to put down my understanding of some Kubernetes topics. May elaborate later.

- Flannel (L3): "L3 network fabric". Allocates IP ranges. Doesn't forward packets, done by...
- VXLAN (L2 in L3, a bit like VLANs over multiple switches except now it's a broadcast domain routed via IP):
  Encapsulates Ethernet frames in packets to be sent over the network to other hosts. Creates single logical network
  that "overlays" the physical and makes it look like the pods are all in the same, directly connected network.
- kube-proxy (L4): "Reflects" TCP ports around to make it look like, externally to a pod, that services are running on
    their standard ports. For example, an app might bind to port 8443 in its container, which is then exposed to some
    arbitrary port on the host OS. Then the K8s
    [Service](https://kubernetes.io/docs/concepts/services-networking/service/) will route from the regular port 443 to this arbitrary port and
    onto the actual service in the container. Kube-proxy is the one "listening" on port 443 then reflecting to the
    container exposed port. This is commonly done in the Linux kernel via iptables redirections (maybe some NAT too?).
