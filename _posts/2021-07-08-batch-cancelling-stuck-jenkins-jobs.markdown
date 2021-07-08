---
title: "Batch cancelling stuck Jenkins jobs"
date: 2021-07-08 09:43
classes: wide
use_math: true
categories: jenkins
---

We use Kubernetes to run Jenkins jobs on disposable pods and, if Jenkins is restarted while jobs are running, the pods
go away and Jenkins attempts to restart the jobs on the now missing pods. This results in stuck jobs in the build queue with
the following status

> There are no nodes with the label \<some label>
>
> Waiting for \<some long period>

There is a handy
[hudson.model.Queue.Item#isStuck()](https://javadoc.jenkins-ci.org/hudson/model/Queue.Item.html#isStuck--) method for
this however, in our Kubernetes environment, we find that it returns very newly created tasks are also marked as "stuck"
even when they have only been in the queue for milliseconds. So I'm using `isStuck()` as a pre-filter and then further
checking by our own criteria.

Without further ado, here is a [Groovy script console](https://www.jenkins.io/doc/book/managing/script-console/) snippet
to clean up jobs queued for more than 6 hours. Tweak for your environment and use at your own risk:

```groovy
import hudson.model.*
import groovy.time.TimeCategory

def q = Jenkins.instance.queue
def now = new Date();
def maxAgeHours = 1

q.items.findAll { it.stuck }.each {
  def taskStart = new Date(it.inQueueSince)
  def age = TimeCategory.minus(now, taskStart)

  if (age.hours > maxAgeHours) {
    println "${it.task.name} is stuck for more than $maxAgeHours hours (${age.hours} hours), cancelling."
    q.cancel(it.task)
  }

}

return "DONE"
```

