---
title: "Using JobDSL Configuration Blocks to Set Additional Branch Source Options"
date: 2020-04-30 13:00
classes: wide
categories: general
---

This post outlines some notes on how to control the generated Jenkins job XML via the `configure` block in JobDSL. This
is necessary when the plugin does not natively support a particular configuration option.

The main [Jenkins configure block](https://github.com/jenkinsci/job-dsl-plugin/wiki/The-Configure-Block) documentation
is great but lacks some specific examples for Multi Branch Pipeline jobs. For this I mean things like "Git LFS Pull after
checkout" and "Advanced Checkout behaviours":

<img src="/images/git_options.png" width="800" height="600"/>

Below is my snippet that works for me - full JobDSL to follow at end:

```
configure {
  def traits = it / sources / data / 'jenkins.branch.BranchSource' / source / traits
  traits << 'jenkins.plugins.git.traits.GitLFSPullTrait' {}
  traits << 'jenkins.plugins.git.traits.CheckoutOptionTrait' {
    extension( class: 'hudson.plugins.git.extensions.impl.CheckoutOption' ) {
      timeout( 60 )
    }
  }
}
```

I drew the following lessons:

- The BranchSource plugin does not support specifying a `configure` block so we have to add the block at the base of the
    JobDSL tree.
- Flag type options like LFS pull need a blank block `{}` to take effect.
- Use `<<` to add to existing traits rather than overwrite the whole lot with just your own trait
- For traits with nested parts you need to specify the class as well. E.g. in the `CheckoutOption` example above.
    Otherwise it's not set.
- Make sure you know where you are in the XML tree - it's useful to have it open side-by-side with the JobDSL file

Anyway, here is a full file showing that snippet in context:

```
multibranchPipelineJob( 'example-job' ) {
  description( 'My super job' )
  branchSources {
    branchSource {
      source {
        git {
          id( 'my-super-job' )
          remote( 'ssh://git.example.com/my_repo.git' )
          credentialsId( 'somecred' )
            cloneOptionTrait {
              extension {
                shallow( true )
                noTags( true )
                reference( '/path/to/reference_repo.git' )
                depth( 100 )
                timeout( 60 )
              }
            }
            pruneStaleBranchTrait()
            branchDiscoveryTrait()
            ignoreOnPushNotificationTrait()
          }
        }
      }
    }
  }

  configure {
    def traits = it / sources / data / 'jenkins.branch.BranchSource' / source / traits
    traits << 'jenkins.plugins.git.traits.GitLFSPullTrait' {}
    traits << 'jenkins.plugins.git.traits.CheckoutOptionTrait' {
      extension( class: 'hudson.plugins.git.extensions.impl.CheckoutOption' ) {
        timeout( 60 )
      }
    }
  }

  orphanedItemStrategy {
    defaultOrphanedItemStrategy {
      pruneDeadBranches( true )
      daysToKeepStr( '60' )
      numToKeepStr( '-1' )
    }
  }

  triggers {
    periodicFolderTrigger {
      interval( '60m' )
    }
  }
}
```


