# Concourse for Spring Boot & Maven

This is an example Concourse pipeline for a Spring Boot web app built with Maven.

## Check the web app runs locally

```sh
./mvnw clean spring-boot:run
open http://localhost:8080
```

## Create a Concourse pipeline

Concourse pipelines consist of three [primitives]: jobs, resources and tasks.

### Job

The first job in the pipeline will get the source code and package using Maven.

```yaml
# ci/jobs.yml

jobs:
- name: package
  plan:
  - get: source-code
    trigger: true
  - task: package
    privileged: true
    file: source-code/ci/tasks/package.yml
```

### Resource

The `package` job gets `source-code`, which is a git resource:

```yaml
# ci/resources.yml

resources:
- name: source-code
  type: git
  source:
    uri: REPLACE_WITH_YOUR_GIT_REPOSITORY_URL
    branch: master
```

### Task

Then it runs the `package` task:

```yaml
# ci/tasks/package.yml

platform: linux

image_resource:
  type: docker-image
  source:
    repository: java
    tag: "8"

inputs:
- name: source-code

run:
  path: source-code/ci/tasks/package.sh
```

Which tells Concourse to:

- create a linux container
- using the [Java 8 Dockerfile] published on Docker Hub
- add the `source-code` resource to the container
- and run the package script in the container

```bash
# ci/tasks/package.sh

#!/bin/bash

set -e -u -x

cd source-code/
./mvnw package
```

> Note: there is no need to run `./mvnw clean pacakge` because Concourse creates a new container for every task.

## Start Concourse

The Concourse team provides a docker image that can be run locally:

```sh
docker-compose up -d
```

## Download the Concourse CLI

```sh
open http://localhost:8080
```

Download the `fly` binary from the Concourse, by clicking on the Apple icon.

Then make it executable and put it in your `$PATH`, for example:

```sh
install ~/Downloads/fly /usr/local/bin/fly
```

## Create the Concourse pipeline

Create a new Concourse target for `fly` called `ci` and login:

```sh
fly --target ci login --concourse-url http://localhost:8080
```

Create a Concourse pipeline called `spring-boot-maven` using the resources and jobs.

```sh
fly --target ci set-pipeline --pipeline spring-boot-maven --config <(cat ci/resources.yml ci/jobs.yml)
```

> NB: the resources and jobs can be combined in one file called `pipeline.yml`. However, separating them into two files is useful as the pipeline grows in size.

Unpause the new pipeline using `fly`, or click Play in the web UI.

```sh
fly --target ci unpause-pipeline --pipeline spring-boot-maven
```