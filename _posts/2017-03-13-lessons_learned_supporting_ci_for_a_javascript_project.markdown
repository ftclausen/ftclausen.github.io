---
title: "Lessons Learned Supporting Continuous Integration (CI) for a Javascript Project"
date: 2017-03-13 14:00
categories: general
---

I've been supporting the CI needs of a JS project and would like to assemble the
lessons learned from the app here so that I can more clearly advise future JS
projects on how to make their efforts more "CI friendly". 

The main things I've noticed are summarised below and covered in the section
that follow. I include a "why" and a section on possible approaches to
accomplishing this.

* TOC
{:toc}

## Do More Processing Ahead of Time

**Why?** Speed.  Allow compilation-like tasks \(CSS transforms, TS to JS
transpilation, image processing, etc\)
to be done completely ahead of time.

In a containerised environment create a "cache" image that has all the prep work
mentioned above applied and then use it as a base image for your deployments
(whether for running tests or the service proper). In a similar fashion a VM
image (VMware or an AWS AMI) can also be created ahead of time in a
non-containerised environment.

To facilitate re-use of the ready-to-run code it could also be packaged into an
OS package such as Debian's .deb, RPM or Chocolatey. This can allow people to
install particular version locally in VMs.

## Single command to start App or Tests

**Why?** Simplicity. Simple startup makes CI easier and less error prone. It
also makes it much easier for people to set up local development environments.

Consolidate tasks into, for example, a single grunt task so that CI folks can
just use that to get things going. Even better would be for the app to know how
to query environment variables to find its dependent services rather than
relying on config files - this is very useful in containerised environments like
Mesos with Marathon or Kubernetes.

## Incremental Processing

**Why?** Speed. Unnecessarily processing unchanged files greatly adds to the
time it takes to run/test your app and also adds compute expenses.

Only transform/compile/process what has changed rather than everything. For
example, if one typescript file has changed then only transform that single file
to JS rather than everything. Likewise for binary processing of items such as
images. I am still research this so can hopefully post more on this topic or
point to the work of others.

## Avoid Unmergable Custom Forks of Upstream Dependencies

**Why?** Avoid lock-in. By using a custom dependency only useful for your team
you are locked into that dependency and cannot easily upgrade to get new
features and stability improvements.

If a framework or library you are using does not meet your needs but can be
easily modified then by all means do so. But then don't sit on the changes and
neglect to submit a PR upstream. If you make changes that look like upstream
won't accept them then question the direction you're taking the code. Likely
there is a more elegant way to accomplish what you want to do.

## Use Packages Instead of Cloning Repos

**Why?** Speed. Simplicity. It is quick and easy to install a package.

By using packages you can more simply pull down dependencies and also
make your build more repeatable by depending on particular versions rather than
what happens to be in a repo's branch. Also, in CI environments, using a package
repo like NPM is easier than arranging access to (often private) Git
repositories which require juggling SSH keys or other credentials.

## Have an Explicit Version

**Why?** Simplicity. Having a single version for your whole project makes it
easier to package and deploy.

It may seem obvious but sometimes projects just organically grow and are built
from their SCM repo without an explicit version. Put in place something so that
each build has some kind of unique version even if it is just the Git commit
hash or simply an incrementing number. Or you can do something more
sophisticated like [semantic versioning](http://www.semver.org).
