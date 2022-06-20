---
title: "Steps for importing PEM files into JKS keystore stored within Kubernetes secrets"
date: 2022-06-20 15:01
classes: wide
use_math: false
categories: jenkins java
---

This post goes through the steps required specifically for creating or updating
a certificate + key for use with Tomcat running in Kubernetes. There are some
oddities because most certificates are distributed as PEM files (containing
x509 cert/key representation). However Java uses its own keystore either as JKS
or PKCS12.

High level the steps are

- Generate key or use old key again (not shown in this post)
- Create CSR and get back signed certificate (not shown in this post)
- Also get CA intermediate certificate - usually comes back with signed certificate (this step might be optional)
- Concatenate your certificate with CA intermediate and all root certs
- Create PKCS12 file
- Create Java keystore file
- Import as Kubernetes secret

# Concatenate your certificate with CA intermediate and all root certs

Usually an up to date ca-certificates package will have a file containing all root certs. This package is available on
Homebrew or on Linux. E.g. it could be in:

- Homebrew - `<homebrew base>var/homebrew/linked/ca-certificates/share/ca-certificates/cacert.pem`
- Linux - Commonly somewhere like `/etc/ssl/certs/ca-certificates.crt`
- Or get it from Firefox or some other browser

Once you have the ca-certificates run

```
cat CA.crt /usr/local/homebrew/var/homebrew/linked/ca-certificates/share/ca-certificates/cacert.pem > allcerts.crt
```

Where `CA.crt` is your CA intermediate certificate, be sure to also check the
path to the `ca-certificates` full CA certificate list. It will probably be
different to above.

# Create PKCS12 file

Then create a PKCS12 file containing your own cert as well as the CA chain:


```
openssl pkcs12
  -export -in example.com.crt \
  -inkey ~/somewhere/safe/example.key \
  -out example.p12 \
  -name <some alias> \
  -CAfile allcerts.crt -caname root \
  -chain
```

Replace the example names as appropriate and also `<some alias>` to something meaningful. E.g. if using Tomcat this is commonly set to `tomcat`.

# Create java keystore file

```
keytool -deststoretype JKS \
  -importkeystore \
  -srckeystore example.p12 \
  -srcstoretype PKCS12 \
  -srcstorepass changeme \
  -alias tomcat # Or any other alias that makes sense for your use case \
  -deststorepass changeme \
  -destkeypass changeme \
  -destkeystore example.keystore
```

Note - `keytool` complains if you use JKS due to it being deprecated but it has the greatest compatibility e.g. with Gradle.

# Import as a Kubernetes Secret

## Save current secret

First save the current secret (if there is one) so that it can be referred to later if need be

```
kubectl get -o yaml secret example-keystore > ~/somewhere/safe
```

## Delete current secret

Then delete the current secret (coordinating as appropriate if this is a heavily used one)

```
kubectl delete secret example-keystore
```

Because you cannot update secrets in-place.

## Re-create secret with new cert + key + certificate chain

```
kubectl create secret generic example-keystore --from-file example.keystore
```

Once this is done systems (like services in K8s) that mount in that secret volume will be able to use the updated cert. Deployments may need scaling down and then up again or just delete the pods so they are recreated.

# Credits

Thanks to these for helping me along my way:

- https://stackoverflow.com/a/8224863/1300307
- https://superuser.com/questions/1142555/openssl-p12-generation-failing-with-ca-bundle-chain-option
