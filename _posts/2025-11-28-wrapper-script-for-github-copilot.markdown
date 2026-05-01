---
title: "Wrapper script workaround for GitHub Copilot to persist tool approvals"
date: 2025-11-28 17:43
classes: wide
use_math: true
categories:
---

# Update

GitHub Copilot does now have global config for allowable tools. Original post
follows.

# Original Post

It would seem GitHub Copilot does not have a global configuration option to persist allowed tools. I often want to run
something like `kubectl` or `jq` with Copilot and having to specify it on the command-line each time is tedious. Note,
if you are reading this a while after I wrote this post then it may well support this properly. What I did was create a
wrapper script, say `~/bin/copilot` and put that ahead of the NVM path in my shell config.

The script looks like:

```shell
#!/usr/bin/env bash
# Note: This script assumes you already have Nodejs installed via NVM.
#       Easy enough to adjust if this is not the case for you.

if [[ -z $NVM_DIR ]]; then
  NVM_DIR="$HOME/.nvm"
fi

node_version=$(node --version)
binary="$NVM_DIR/versions/node/$node_version/bin/copilot"
if [[ ! -f "$binary" ]]; then
  source "$NVM_DIR/nvm.sh"
  npm install -g @github/copilot
fi

# Add any other tools you may need
"$binary" --allow-tool 'shell(git)' \
  --allow-tool 'write' \
  --allow-tool 'shell(kubectl)' \
  --allow-tool 'shell(jq)' \
  --deny-tool 'shell(git push)' \
  "$@"
```

Which seems to work for me as a low-tech way to customise this.
