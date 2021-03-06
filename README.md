# Name Redactor Service

![Build Docker image on Push to Feature branches](https://github.com/locmai/name-redactor-service/workflows/Build%20Docker%20image%20on%20Push%20to%20Feature%20branches/badge.svg)

![Build Docker image with Tag](https://github.com/locmai/name-redactor-service/workflows/Build%20Docker%20image%20with%20Tag/badge.svg)

## Quickstart with Docker

```bash
docker run --rm -p 8000:8000 locmai/name-redactor-service:0.1.0
```

To try out the service, please go to http://localhost:8000/docs for API docs or use Postman to send the GET request (fetch from the given source) or POST request with the JSON body.

## Task 1: Build a service to redact the names of the people from JSON data

- Assuming we might have an NLP service to detect human's name in the data. I used [spaCy](https://spacy.io/) library with their default model for English. - The real model should be trained by a different process and store in cloud storage like S3 or Azure Storage and mounted to the container at runtime to minimize the Docker image and separate the concerns of NLP model with business logic.
- Implemented two methods, GET request (fetch from the given [source](http://therecord.co/feed.json)) or POST request with the JSON body using [fastAPI](https://fastapi.tiangolo.com/) framework - which I found it was similar to Flask but faster and more handy tools were given.
- I tried to recursively handle the JSON data which led to slow performance (and spaghetti code ...). Another solution could be parsing all the data to string and replace them all.

## Task 2: Package the service & deploy to a k8s cluster

- Packaged the service with Docker.
- Wrote a simple Helm chart for deploying the service to k8s.
- Initialized Skaffold skeleton for development enviroment and production

## Task 3: CI strategy for the service

I suggest keeping the CI workflows simple and lightweight.

### Assumption 1: No specific cloud-vendor with GitHub

Leverage GitHub Packages and GitHub Actions for the CI workflows:

#### Tools / Services:

- For workflow engine: GitHub Actions
- For package registries: GitHub Packages
- For code quality checking: SonarQube
- For packaging image: Docker
- Secrets and configurations will be stored in GitHub settings.

#### Workflows:

- Trigger on feature branchs to build staging images
- Trigger on tags to build release images

![Image of CI](https://raw.githubusercontent.com/locmai/name-redactor-service/develop/docs/CI.png)

A quick PoC with GitHub:

- https://github.com/locmai/name-redactor-service/actions
- https://github.com/locmai/name-redactor-service/tree/develop/.github/workflows

```
- Trigger when new commit pushed on 'feature' branches (branchs with the name "features/**"):
1. Run unit tests and code quality scanning (e.g. SonarQube) => Publish the result on GitHub / Send notification
2. Build image and tag with the git commit SHA.
3. Notify when the process is done

- Trigger when new tag pushed:
1. Run unit tests and code quality checking => Publish the result on GitHub / Send notification
2. Build image and tag with the git tag for the release version.
3. Notify when the process is done

- Notes:
+ Configurations could be passed throught the env_var or configuration files
+ Secrets must be hidden.
```

### Assumption 2: A specific cloud-vendor

Use almost every services provided by the vendor - example: Azure

#### Tools / Services:

- For workflow engine: Azure Pipeline
- For package registries: Azure Artifacts / Azure Container Registry
- For code quality checking: SonarQube
- For packaging image: Docker
- Secrets and configurations will be stored in Azure settings.

#### Workflows:

- Same as assumption 1

### Other suggestions:

- Use kaniko to build docker images - leverage caching from cloud storage and build images in no-docker environments.
- Notify or display the CI information in one place so everyone could keep track easily. (GitHub PR review section or Slack channels)

### How to handle zero-downtime upgrade of the services

- Prepare a failover instance to serve the traffic outside of Kubernetes cluster - it could be on another cloud.
- Deploy at least 3 instances. As we use Kubernetes, rolling update is a common solution. I have set the health check for the service.

---

## Setup Environment

### Prerequisites

- python3
- pip
- virtualenv
- kubectl
- helm2
- docker

### Install dependencies

With virtualenv:

```
virtualenv env
source ./env/bin/activate
```

Install dependencies:

```
pip install -r requirements.txt
```

### Start development

With uvicorn

```
uvicorn main:app --reload
```

With Skaffold

```
skaffold dev
```

### Swagger

![Image of Swagger 1](https://raw.githubusercontent.com/locmai/name-redactor-service/develop/docs/ScreenShot2020-03-10-1.png)

![Image of Swagger 2](https://raw.githubusercontent.com/locmai/name-redactor-service/develop/docs/ScreenShot2020-03-10-3.png)
