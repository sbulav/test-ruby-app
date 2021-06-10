# Test ruby application

This repository provides a test Ruby application which will listen on port 80,
helm chart and CI/CD to build and deploy it to minikube. Application is located in the `app`
folder; check out [application readme](app/README.md) for more information

[![GitHub Super-Linter](https://github.com/sbulav/test-ruby-app/workflows/Lint%20Code%20Base/badge.svg)](https://github.com/marketplace/actions/super-linter)

## Prerequisites

To reproduce the setup, following tools are required:
- [Docker](https://docs.docker.com/get-docker/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Docker hub Account](https://hub.docker.com/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm v3](https://helm.sh/docs/intro/install/)

## Usage

### Continous Integration

Docker images are built automatically, via GitHub Action 'Build docker image'
This workflow is triggered on following events:
- Push to master when file in app folder is modified
- Push of new tags
- Push to pull request to master, when file in app folder is modified

CI workflow uses SEMVER approach for tagging Docker images. Tags
will be assigned in a following way:

| Event           | Ref                           | Docker Tags                         |
|-----------------|-------------------------------|-------------------------------------|
| `pull_request`  | `refs/pull/2/merge`           | `pr-2`                              |
| `push`          | `refs/heads/master`           | `master`                            |
| `push`          | `refs/heads/releases/v1`      | `releases-v1`                       |
| `push tag`      | `refs/tags/v1.2.3`            | `1.2.3`, `1.2`, `latest`            |
| `push tag`      | `refs/tags/v2.0.8-beta.67`    | `2.0.8-beta.67`                     |

Also on each push GitHub Action 'Lint codebase' will be triggered. This action
uses GitHub Super-Linter under the hood and will run a list of linters against
detected languages.

### Continous Deployment

Since this is a test application, intended to be deployed to minikube, it's not
possible to realize automated deployment. In real life, I would use GitHub Actions/
Jenkins/Terraform or other tools, depending on requirements.

Helm chart can be deployed manually via:
