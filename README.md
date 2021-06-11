# Test ruby application

This repository provides a test Ruby application which will listen on port 80,
Helm Chart and CI/CD to build and deploy it to minikube. Application is located
in the `app` folder; check out [Application Readme](app/README.md) for more
information

[![GitHub Super-Linter](https://github.com/sbulav/test-ruby-app/workflows/Lint%20Code%20Base/badge.svg)](https://github.com/marketplace/actions/super-linter)

## Prerequisites

To reproduce the setup, following tools are required:
- [Docker](https://docs.docker.com/get-docker/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Docker hub Account](https://hub.docker.com/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm v3](https://helm.sh/docs/intro/install/)

## Architecture

Application is packed into Docker container and intended to be installed into
Kubernetes via Helm Chart. Helm chart installs following Kubernetes resources:
- Deployment
- Service (for minikube I use NodePort type)
- Ingress (if enabled)

High Availability is reached by putting application pods behind the load
balancer(in our case Kubernetes Service and/or Ingress) and scaling them to
the desired amount(i.e. 3 replicas)

Security is achieved by running the application as a non-root user and enforcing
containers to be immutable.

### Continous Integration

Docker images are built automatically, via GitHub Action 'Build docker image'
This workflow is triggered on the following events:
- Push to master when a file in `app` folder is modified
- Push of new tags
- Push to pull request to master, when a file in `app` folder is modified

CI workflow uses `SEMVER` approach for tagging Docker images. Tags
will be assigned in the following way:

| Event           | Ref                           | Docker Tags                         |
|-----------------|-------------------------------|-------------------------------------|
| `pull_request`  | `refs/pull/2/merge`           | `pr-2`                              |
| `push`          | `refs/heads/master`           | `master`                            |
| `push`          | `refs/heads/releases/v1`      | `releases-v1`                       |
| `push tag`      | `refs/tags/v1.2.3`            | `1.2.3`, `1.2`, `latest`            |
| `push tag`      | `refs/tags/v2.0.8-beta.67`    | `2.0.8-beta.67`                     |

Also on each push GitHub Action 'Lint Codebase' will be triggered. This action
uses GitHub Super-Linter under the hood and will run a set of linters against
detected languages.

### Continous Deployment

Since this is a test application, intended to be deployed to minikube, it's not
possible to realize automated deployment. In real life, I would use GitHub Actions/
Jenkins/Terraform or other tools, depending on requirements.

Helm chart can be deployed manually via:
```bash
# Verify that Helm Chart renders correctly:
helm install --dry-run http-server helm/chart
# Install the chart
helm install http-server helm/chart
```

## Steps to reproduce the setup

1. Install Prerequisites
2. Fork my github repo `https://github.com/sbulav/test-ruby-app`
3. Create new [Personal Access Token](https://docs.docker.com/docker-hub/access-tokens/)
4. Create 2 repository secrets:
- DOCKERHUB_USERNAME - Username of Docker hub user
- DOCKERHUB_TOKEN - Personal Access Token of Docker Hub user
5. Start minikube by running following command:
```bash
minikube start
```
6. Enable minikube Ingress:
```bash
minikube addons enable ingress
```
7. Clone repository to your local development machine:
```bash
git clone git@github.com:<USERNAME>/test-ruby-app.git
```
8. Amend Helm values `helm/chart/values.yaml` to your need
9. Issue a new tag:
```bash
git tag v1.0.0
git push --tags
```
Check out repository Github Actions and make sure that new Docker image is
built and pushed to docker hub.
10. Install Helm chart.
```bash
# Verify that Helm Chart renders correctly:
helm install --dry-run http-server helm/chart
# Install the chart
helm install http-server helm/chart
```
Or install specific Image tag:
```bash
helm install http-server helm/chart --set tag=1.0.0
```
11. Wait until the application is up and running and open web page in browser:
```bash
minikube service http-server
```
12. (Optional) Access application via Ingress:
```bash
# Create DNS entry OR entry in /etc/hosts
sudo echo "$(minikube ip) http-server.local" >> /etc/hosts
```
Access application via browser or curl:
```bash
$ curl http-server.local.
Well, hello there!
```
