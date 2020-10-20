---
title: "Terraform: Solving the double slash mystery with Terragrunt"
date: 2020-10-20 11:36
classes: wide
use_math: true
categories: dev infra
---

The solution I found to my particular case of "Error: Unreadable module directory" when using Terragrunt.

# Terms

A quick note on wording

- _Concrete service_: A service in TF _using_ a shared module. Example: a specific web site site.example.com
- _Shared module_ or _module_: Encapsulated TF provisioning logic to be reused as required. Example: the "webserver" module
    used by the above web site concrete service

# Details

Where: Concrete service definition.

I was baffled by the following error message from Terraform when standing up a concrete service (using contrived module
names):

```
Initializing modules...
- sibling1_mod in
- sibling2_mod in
-
Error: Unreadable module directory

Unable to evaluate directory symlink: lstat ../sibling1: no such file or directory
```

This is triggered when including a module as follows when trying to instantiate the concrete service

```
terraform {
  source = "../../../../../modules/example1"
}
```

The fix was **simply to add a missing slash (`/`)** to the base of the repository relative path above. So if
`../../../../../` takes you to the Git repo root then you need a double slash at the end `../../../../..//` followed by
the module path. Full example:

```
terraform {
  source = "../../../../..//modules/example1"
  #                       ^^ Double slash here!
}
```

I should say Terragrunt also offers a useful hint; the implications of which I did not fully understand initially:

> WARNING: no double-slash (//) found in source URL /Users/fclausen/freddiegit/tf12-mod-source-issue/modules/example1. Relative paths in downloaded Terraform code may not work.

# Circumstances in which this happen

This will happen under the following conditions

- Using in-repo Terraform modules; either part of the repo or placed there some other way
- Forgetting the double slash as described above (obviously)
- The module you are `source`ing includes transitive dependencies at the same level in the directory tree. E.g.
```
.
├── accounts
│   └── account1
│       └── earth
│           └── dev
│               └── service1 # <-- source =  "../../../../..//modules/example1"
├── common
└── modules
    ├── example1 # Depends on sibling1 and sibling2 via `source ../sibling1`, `source ../sibling2`
    ├── sibling1
    └── sibling2
```

# Example Repo

I have put an example repo to demonstrate this issue here:

https://github.com/ftclausen/tf12-mod-source-issue

Specifically you can check out the `demonstrate_incorrect_slash_usage` tag to replicate
