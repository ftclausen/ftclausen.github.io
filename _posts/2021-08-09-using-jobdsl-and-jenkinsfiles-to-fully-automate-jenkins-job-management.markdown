---
title: "Using JobDSL and Jenkinsfiles to fully automate Jenkins job management"
date: 2021-08-09 14:48
classes: wide
use_math: false
categories: jenkins
---

By using JobDSL and Jenkinsfiles we can fully automate all aspects of Jenkins job management. The two components play the following roles

- *JobDSL*: The skeletal framework of a job that "registers" it in the Jenkins UI but does not contain any core job logic
- *Jenkinsfiles*: Contains all the project specific steps to accomplish the project goals such as building or deploying code

![Sandwhich DSL goodness](/images/sandwhich.png){:height="600px" width="600px"}

It does not have to be JobDSL vs. Jenkinsfiles but, rather, JobDSL **and** Jenkinsfiles.

# JobDSL

The code statements in JobDSL map directly to what is in the Jenkins UI as shown below

![Yummy JobDSL FRED_AI](/images/job_dsl.png)

# Jenkinsfiles

The "meat" of the job, that is, the core logic and all logic specific to the project in question is contained in the
Jenkinsfile. Quite often this is just called "Jenkinsfile" or "Jenkinsfile.groovy" (if you want to clue in your
editor/IDE) but it can be called anything. 

Below are some Jenkinsfile snippets showing, roughly, how the correspond to what you might see in the Jenkins UI.

![Yummy Jenkinsfiles](/images/jenkinsfile.png)
