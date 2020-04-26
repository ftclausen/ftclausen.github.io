---
title: "Gradle Sync task is not incremental"
date: 2016-09-21 12:09
classes: wide
categories: general
---

While this may seem obvious to some it came as a surprise to learn that Gradle's
[Sync](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.Sync.html) task
will copy the entire source even if only one file has changed.

For example, with this task in our `build.gradle`

    task syncToTarget(type: Sync) {
      from "sourceDir"
      into new File("targetSync/destDir")
      eachFile { details ->
        println "Syncing file: ${details.name}"
      }
    }

and the following project layout

    .
    ├── build.gradle
    ├── sourceDir
    │   ├── file1.txt
    │   ├── file2.txt
    │   └── file3.txt
    └── targetSync

running **the first `Sync`** task produces the following output

    :syncToTarget
    Syncing file: file1.txt
    Syncing file: file2.txt
    Syncing file: file3.txt

    BUILD SUCCESSFUL

    Total time: 2.531 secs

and then **the next sync** with no changes works as expected

    :syncToTarget UP-TO-DATE

    BUILD SUCCESSFUL

    Total time: 2.119 secs

If we then **add a single file** to the source directory

    $ echo $RANDOM > sourceDir/file4.txt

And re-run the Sync task we'll see that all files are copied again

    :syncToTarget
    Syncing file: file1.txt
    Syncing file: file2.txt
    Syncing file: file3.txt
    Syncing file: file4.txt

    BUILD SUCCESSFUL

    Total time: 2.927 secs

Checking the time stamps confirms that all files have been copied again. 
Thus, in conclusion, **the Gradle Sync task is not an rsync replacement**.
