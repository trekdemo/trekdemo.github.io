---
categories: CI/CD
date:   2022-12-13 9:00:01 +0100
title:  "You are missing errors in your Google Cloud Build steps"
excerpt: |-
  If you are using Google Cloud Build, you likely encountered cases where you needed to continue the process regardless of the result of a particular step.
  <br>If your step hides if it fails because it is OK to let it fail, I have a better option for you.
---

If you are using GCB, you likely encountered cases where you needed to continue the process regardless of the result of a particular step.
Naturally, builds fail if a step terminates with a non-zero exit code.

*[GCB]: Google Cloud Build

Here is an example that pulls a container image. If the image does not exist, it's okay to continue. Your steps may look different, more involved or use another command.

```yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    entrypoint: 'bash'
    args: ['-c', 'docker pull eu.gcr.io/project/myimage:$TAG || exit 0']
```


> *If your step hides if it fails because it is OK to let it fail*, I have a better option for you.

## What is the problem with the code above?
There are two problems with ignoring the exit code of the step.

1. The step will lie about the exit code, and you will not be able to see when the command fails. Seeing an always passing step provides a *false assurance* that everything is fine.
2. The step is convoluted by the error handling code. It is no longer possible to glance at the `args` and know what the step will do.

## Let it fail
The Google Cloud Build [configuration schema](https://cloud.google.com/build/docs/build-config-file-schema#allowfailure) has a ton of underutilised features.

There are two handy features for this situation. *Please note that both of them are experimental.*

- [`allowFailure: <boolean>`](https://cloud.google.com/build/docs/build-config-file-schema#allowfailure)
- [`allowExitCodes: <array|string>`](https://cloud.google.com/build/docs/build-config-file-schema#allowexitcodes)

These options allow you to tell the runner to ignore if the step fails. With the latter, you can even specify which error codes are allowed.

```yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['pull', 'eu.gcr.io/project/myimage:$TAG']
    allowFailure: true
  - name: 'gcr.io/cloud-builders/docker'
    args: ['pull', 'eu.gcr.io/project/myimage:$TAG']
    allowExitCodes: [1]
```

## Alternatives
If this solution is not for you, it may be worth looking at the [`script`](https://cloud.google.com/build/docs/build-config-file-schema#script) property and how you can use it to [run bash scripts](https://cloud.google.com/build/docs/configuring-builds/run-bash-scripts#using_the_script_field) .

```yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    script: |
        docker pull eu.gcr.io/project/myimage:$TAG || exit 0
```
