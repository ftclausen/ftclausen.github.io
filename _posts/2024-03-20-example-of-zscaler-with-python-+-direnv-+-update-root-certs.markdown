---
title: "Example of ZScaler with Python + direnv + update root certs"
date: 2024-03-20 10:27
classes: wide
use_math: false
categories: infra
---

For those using ZScaler security products to intercept and scan TLS traffic this post shows an approach that worked for
me to get Python working. ZScaler intercepts TLS traffic so, obviously, this requires overriding the root certificates
and ZScaler impersonating all sites that I connect to. Basically, an authorised person-in-the-middle that work uses to
check for malicious traffic.

For Python specifically, I used the [ZScaler
instructions](https://help.zscaler.com/zia/adding-custom-certificate-application-specific-trust-store) combined with
[direnv](https://github.com/direnv/direnv) and a custom script to set up individual Python projects to allow the
certificate impersonation not to break all communication.

NOTE: This is all not beautiful (nor is SSL interception) but putting it out there in case it helps someone.

# Assumptions

This post is for a Homebrew and pyenv type of situation. It could be adapted for other environments. Other assumptions:

- ZScaler root CA at `~/etc/zscaler_root_ca.pem`
- Homebew ca-certificates at `/usr/local/Cellar/ca-certificates`
- Python venv is at `venv`

# Direnv

The first step is setting up [direnv](https://github.com/direnv/direnv) as described on the README there.

# Script to configure root certificate store

Then I created a script to update the Homebrew root certificate store with the ZScaler root CA cert:

```shell
# ~/etc/ca_bundle
# This file is to be sourced, not run directly.
prefix="/usr/local/Cellar/ca-certificates"
latest="$(ls -1tr /usr/local/Cellar/ca-certificates)"
base_path="$prefix/$latest/share/ca-certificates"

export REQUESTS_CA_BUNDLE="$base_path/cacert.pem"

# This confusing array of paths taken from:
# https://help.zscaler.com/zia/adding-custom-certificate-application-specific-trust-store#pip
export CERT_PATH="$REQUESTS_CA_BUNDLE"
export CERT_DIR="$base_path"
export SSL_CERT_FILE=${CERT_PATH}
export SSL_CERT_DIR="$base_path"

echo "DEBUG: Latest bundle at $REQUESTS_CA_BUNDLE"
if grep -i zscaler $REQUESTS_CA_BUNDLE; then
  echo "ZScaler cert already present, skipping"
else
cat <<END >> $REQUESTS_CA_BUNDLE

ZScaler Interception Shenanigans
================================

$(cat ~/etc/zscaler_root_ca.pem)
END
fi
```

# Tying it all together

Then, in the base project dir, create a `.envrc` and make use what we created above:

```
# Activate the virtual environment as usual
source venv/bin/activate
# Set environment variables to override CA certificate store and add ZScaler
# root certificate to the store
source $HOME/bin/ca_bundle
```

This works for me in that `pip-sync` can do its job. Of course this may have mistakes or will not work for you.
