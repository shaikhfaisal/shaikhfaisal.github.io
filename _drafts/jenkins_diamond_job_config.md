---
title: Diamond job configuration in Jenkins Job Builder
layout: default
excerpt: <p>Parallelising job execution in Jenkins can lead to a significant reduction in build execution times. The time it takes to complete a build becomes the time taken by the longest executing job.</p> 
---
### {{page.title}}


<p>Parallelising job execution in Jenkins can lead to a significant reduction in build execution times. The time it takes to complete a build becomes the time taken by the longest executing job.</p> 

i.e.

        T-build  = max(T-job1, T-job2, T-job3,.....T-jobn)

instead of 

        T-build  = Sum(T-job1, T-job2, T-job3,.....T-jobn)

A jenkins job usually starts out looking like this:

      Job1:
      |=  make unittests
      |=  make stylechecks
      |=  make docuumentation
      |=  make messdetection
      |=  make finalJobInChain
      |
      |=  Build complete!

These jobs are all executed in one large monolithic Jenkins job. This approach has a number of disadvantages:

1. The total time for the completion of the build is the sum of each jobs execution times.

2. Developers do not get all errors in one go. If an error occurs in the first step - a developer needs to submit a fix for that and rerun the build before they get to see any errors in any of the subsequent steps. This can get tedious if a number of errors are seen in the job.

3. Monolithic jobs are not the easiest to reuse.

### Diamond configs:

If the various steps within a build do not depend on each other, consider splitting them out in multiple individual steps. e.g. running the unit tests and running the code style checks do not depend on each other and can be executed independently of each other.

An example of dependent steps would be if the last step in the chain produces a packaged/compiled version of the code ready for distribution. It does not make sense to this (especially if it is a resource intensive process) upfront or in parallel with the other jobs.


In a diamond configuration, the monolithic job is broken up into multiple components - each a very specific job that does exactly one thing. The components are then triggered in parallel and when each completes, it notifies the parent job on whether it completed sucessfully or not.

            ----- make unittests ------
            |                         |
            ----- make stylechecks ----
   Job1: ---|                         |---- make finalJobInChain------- Build complete!
            ----- make documentation --
            |                         |
            ----- make messdetection --


You can setup a diamond job configuration quite easily using Jenkins Job Builder.

{% highlight yaml %}
- job-template:
    name: 'Example job'
    description: 'Example description'
    project-type: freestyle
    block-downstream: true
    block-upstream: true
    scm:
      - git:
          url: 'http://example.com/project1.git'
          branches: 'master'
          credentials-id: 'jenkins_api_token'
    builders:
      - trigger-builds:
          - project: 'Example job unit tests,
                      Example job style checks,
                      Example job documentation,
                      Example job mess detection
                     '
            git-revision: true
            block: true
      
      - trigger-builds:
          - project: 'Final job in chain'
            git-revision: true
            block: true
{% endhighlight %}
