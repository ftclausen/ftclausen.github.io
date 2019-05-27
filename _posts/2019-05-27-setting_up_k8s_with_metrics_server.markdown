---
title: "Kubernetes metrics-server with TLS verification"
date: 2019-05-27 13:00
categories: general
---

These are my notes on what it took to integrate [metrics-server](https://github.com/kubernetes-incubator/metrics-server)
into my sandbox [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) learning cluster.
Note my cluster is on VirtualBox locally hence some IP network ranges might look different.

tl;dr - you can skip all this by running with `--kubelet-insecure-tls` but for something approaching what you'd use in
production read on.

I initially installed metrics-server using [Helm](https://github.com/helm/charts/tree/master/stable/metrics-server) thinking
it would just work out of the box. This is not the case beccause Helm won't configure the PKI infrastructure necessary
to make this work. As I understand it you need a new CA specifically for handling extension apiserver traffic. In this
case the metrics-server

## Prerequisites

Obviously a running K8s cluster of any type but I set one up using the above Kubernetes the Hard Way. Actually
installing metrics-server can easily be done via Helm but this leaves it in a non-working state; this post attempts to
address how to get it from non-working -> working.

## Terms

- _Main apiserver_ - The core kube-apiserver
- _Extension apiserver_ - The extension api-server, in this case metrics-server

## High level overview

The parameters below are not the only ones you need to get things working; we're just focusing on the thorniest side of
things: PKI based trust.

**Overview of where options are used**

![k8s_aggregation.png](/images/k8s_aggregation.png)

**Main apiserver**

The apiserver, as usual, has the cluster CA certificate to verify incoming requests. This is done with
`--client-ca-file`.

The key is to understanding that the main apiserver is acting as a proxy so it become a TLS client _of the extension
apiserver_. Thus we need to inform it of the appropriate CA and cert details with:

- `--requestheader-client-ca-file` - The is the CA we trust when connecting to the extension apiserver.
- `--proxy-client-cert-file` - This is used by the main apiserver when calling out to the extension apiserver. Must be
    signed by a CA the extension trusts, I think it would have to be the same one specified in
    `--requestheader-client-ca-file`
- `--proxy-client-key-file` - Key belonging to above cert

If you're not running kube-proxy on your masters then you'll need to set up routing so that your masters can reach the
workers. Once that is done add `--enable-aggregator-routing=true` to the kube-apiserver arguments.

**Extension apiserver**

The extension apiserver, metrics-server in this case, also needs to trusts the incoming requests proxied by the main
apiserver. We provide similar arguments to the main apiserver and some extra:

- `--requestheader-client-ca-file` - The is the CA we trust when receiving connections from the main apiserver
- `--client-ca-file` - Once the above requestheader check is OK we take the CN from certificates signed with this CA (in
    my learning cluster the same CA as `--requestheader-client-ca-file`) and impersonate that CN instead of the request
    being from the main apiserver. I think.
- `--tls-cert-file` - HTTPS cert file signed by the aggregation CA in my case
- `--tls-proviate-key-file` - Key for above
- `--kubelet-certificate-authority` - The main apiserver CA certicate so that we can speak to the worker nodes

## Full argument lists

Your paths and cert names will vary.

**Main apiserver**

```
/usr/local/bin/kube-apiserver \
  --advertise-address=192.168.99.11 \
  --allow-privileged=true \
  --apiserver-count=3 \
  --audit-log-maxage=30 \
  --audit-log-maxbackup=3 \
  --audit-log-maxsize=100 \
  --audit-log-path=/var/log/audit.log \
  --authorization-mode=Node,RBAC \
  --bind-address=0.0.0.0 \
  --client-ca-file=/var/lib/kubernetes/ca.pem \
  --enable-admission-plugins=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \
  --enable-swagger-ui=true \
  --etcd-cafile=/var/lib/kubernetes/ca.pem \
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \
  --etcd-servers=https://192.168.99.11:2379,https://192.168.99.12:2379,https://192.168.99.13:2379 \
  --event-ttl=1h \
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \
  --kubelet-https=true \
  --runtime-config=api/all \
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \
  --service-cluster-ip-range=10.32.0.0/24 \
  --service-node-port-range=30000-32767 \
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \
  --requestheader-client-ca-file=/var/lib/kubernetes/requestheader-client-ca.crt \
  --requestheader-allowed-names="" \
  --requestheader-extra-headers-prefix=X-Remote-Extra- \
  --requestheader-group-headers=X-Remote-Group \
  --requestheader-username-headers=X-Remote-User \
  --proxy-client-cert-file=/var/lib/kubernetes/request-header.pem \
  --proxy-client-key-file=/var/lib/kubernetes/request-header-key.pem \
  --enable-aggregator-routing=true \
  --v=2
```

**metrics-server**

```
  /metrics-server
  --cert-dir=/var/lib/kubernetes
  --client-ca-file=/var/lib/kubernetes/requestheader-client-ca.crt
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem
  --requestheader-allowed-names=""
  --requestheader-client-ca-file=/var/lib/kubernetes/requestheader-client-ca.crt
  --tls-cert-file=/var/lib/kubernetes/metrics-server.pem
  --tls-private-key-file=/var/lib/kubernetes/metrics-server-key.pem
  --requestheader-extra-headers-prefix=X-Remote-Extra-
  --requestheader-group-headers=X-Remote-Group
  --requestheader-username-headers=X-Remote-User
  --logtostderr
  --v=5
  --secure-port=8443
```
