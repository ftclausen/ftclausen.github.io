---
title: "Cleaning up old Kubernetes deployments"
date: 2016-06-27 14:06
categories: general
---

We run a lot of transient deployments on Kubernetes as part of Jenkins
end-to-end (E2E) testing. Usually the Jenkins job cleans this up but sometimes
it fails, for example, if the Jenkins slave Pod is ungracefully killed. But this
approach can be applied to any kind of general "reaping" of old Kubernetes
deployments.

For this we have defined a [Kubernetes
CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
and in the below sections I'll explain how to

- Look for failed pods and clean them
- Look for orphaned ingresses
- Look for deployments and services over a certain age (for us 12 hours)

I'll first outline the shell snippets and then tie it all together in a CronJob
configuration.

{:toc}

### Prerequisites

- Working Kubernetes environment with CronJob support
- Container image with kubectl and jq
- Authentication configured so that kubectl commands work from transient CronJob
    pods.
- **All transient items are labelled as being transient for extra safety. For
    example a label like `ci-transient=true`**

This doc also leaves out namespace details so adjust as necessary to make things
run in the appropriate namespace for your situation.

### Finding failed pods

Firstly we want to clean up failed pods so we don't hit any K8s limits with
regards to pods in failed states:


```
echo "Looking for failed pods"
records=$(kubectl get pods --show-all --include-uninitialized | grep -e Error -e Completed -e ContainerCannotRun | grep -e h$ -e d$)
if [[ -n "$records" ]]; then
  echo "$records"
  echo "$records" | awk '{print $1}' | xargs kubectl delete pods
fi
```

### Finding orphaned ingresses

Sometimes ingresses can be left lying around since these are separately managed
so let's clean them up too

```
echo "Looking for orphaned ingresses"
records=$(kubectl get svc,ing -l ci-transient=true | grep -e d$)
if [ -n "$records" ]; then
  echo "$records"
  echo "$records" | awk '{print $1}' | xargs kubectl  delete
fi
```

### Finding deployments and services older than a certain age

The basic process is

- Find old deployments and ingresses older than max age and save their
    timestamps
- Use the saved timestamps to query K8s to get the names associated with the
    saved timestamps
- Use the names so retrieved to delete the old services and pods

This requires some more gymnastics in the form of using jq to parse the JSON
output from kubectl. First we have to get a list of all services and
deployments, then filter out the age and convert it to Unix time for easy
comparison

```
echo "Looking for orphaned deployments and services"
all_items=$(mktemp) # We'll save all items here
old_entries=$(mktemp) # Cleanup candidates saved in this file
max_age=43200 # 12 hours
kubectl get -o json deployments,service -l ci-transient=true > "$all_items"
# First find all items older than 12 hours by subtracting max_age from now, now
# being defined as $(date +%s).
jq --argjson max_age $(( $(date +%s) - $max_age )) -r \
  '[ .items[].metadata | .creationTimestamp | fromdate | tonumber | select( . < $max_age ) ] | unique' \
  "$all_items" > "$old_entries"
```

The `jq` filter could do with some clarification:

- Firstly wrap the filter part in an array `[ .items[].metadata ... ]` so that we can use `unique` to get
    weed out duplicates
- `.items[].metadata` then filter on the service and deployment metadata
- `.creationTimestamp` get the item creation times as presented by K8s
- `.fromdate` convert to Unix time i.e. seconds since 1970
- `.tonumber` convert to integer so numeric comparisons work
- `select( . < $max_age )` only select items older than our max age criteria
- `| unique` just get the unique names since they may be repeated if a
    deployment and service share the same name

Now we use the list potentially stale timestamps to again query the kubectl
output to find the names of the targets for cleanup

```
# Then use the timestamps from above to get the names of the orphan items
deletion_targets=$(mktemp)
jq -r '.[] | todate' "$old_entries" | while read -r timestamp; do
  jq -r --arg ts "$timestamp" \
    '.items[].metadata | select( .creationTimestamp == $ts ) | .name' \
    < "$all_items"
done > "$deletion_targets"
```

*Finally* perform the deletion:

```
cat "$deletion_targets" | while read -r item; do
  echo "kubectl delete deployment,service $item"
done
rm "$old_entries" "$deletion_targets" "$all_items"
```


### Tying it all together

Below is the full Kubernetes CronJob YAML definition:

```
#
# Monitors the and removes problematic objects
#
# To update:
#   kubectl apply -f clean-kubernetes-slaves-cronjob.yaml
#
# To view status:
#   kubectl describe cronjob/clean-kubernetes-slaves
#
# To view output from a specific cleanup run:
#   kubectl logs job/clean-kubernetes-slaves-12345
#
apiVersion: batch/v2alpha1
kind: CronJob
metadata:
  name: clean-kubernetes-slaves-fclausen
spec:
  schedule: "*/10 * * * *"
  concurrencyPolicy: Replace
  successfulJobsHistoryLimit: 12
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: jenkins-kubernetes-serviceaccount
          containers:
          - name: kubectl
            image: 392477962641.dkr.ecr.us-east-1.amazonaws.com/saas/kubectl-docker:1219
            command:
            - /bin/bash
            args:
            - -c
            - |
              set -u

              echo "Looking for failed pods"
              records=$(kubectl get pods --show-all --include-uninitialized | grep -e Error -e Completed -e ContainerCannotRun | grep -e h$ -e d$)
              if [[ -n "$records" ]]; then
                echo "$records"
                echo "$records" | awk '{print $1}' | xargs kubectl delete pods
              fi

              echo "Looking for orphaned ingresses"
              records=$(kubectl get svc,ing -l ci-transient=true | grep -e d$)
              if [ -n "$records" ]; then
                echo "$records"
                echo "$records" | awk '{print $1}' | xargs kubectl  delete
              fi

              echo "Looking for orphaned deployments and services"
              all_items=$(mktemp) # We'll save all items here
              old_entries=$(mktemp) # Cleanup candidates saved in this file
              max_age=43200 # 12 hours
              kubectl get -o json deployments,service -l ci-transient=true > "$all_items"
              # First find all items older than 12 hours by subtracting max_age from now, now
              # being defined as $(date +%s).
              jq --argjson max_age $(( $(date +%s) - $max_age )) -r \
                '[ .items[].metadata | .creationTimestamp | fromdate | tonumber | select( . < $max_age ) ] | unique' \
                "$all_items" > "$old_entries"
              # Then use the timestamps from above to get the names of the orphan items
              deletion_targets=$(mktemp)
              jq -r '.[] | todate' "$old_entries" | while read -r timestamp; do
                jq -r --arg ts "$timestamp" \
                  '.items[].metadata | select( .creationTimestamp == $ts ) | .name' \
                  < "$all_items"
              done > "$deletion_targets"


              cat "$deletion_targets" | while read -r item; do
                echo "kubectl delete deployment,service $item"
              done
              rm "$old_entries" "$deletion_targets" "$all_items"
          restartPolicy: OnFailure
```
