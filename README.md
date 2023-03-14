# NEDSS Reusable Workflows
## Overview
This repository is a central location for managing reusable workflows to be used in microservices developed as a part of the NBS modernization project. GitHub Actions is the tool used to create these workflows which are intended to be adopted by any team who needs a container built and stored.

## Prerequisites
1. Request your repository be granted access to the environment containing the Elastic Container Registry (ECR).
2. Request and received confirmation that an ECR was created to store your artifact (microservice container image).

## Workflows
There are 2 workflows currently provided.
- Build-microservice-container.yaml - is intended to be triggered automatically whenever a specific tag is created. Results in a container being built and deployed to an AWS ECR. That container will be tagged identically with the tag created in the GitHub repository. Sample git cli commands to create a tag where aaa12342 corresponds to the GitHub commit SHA:
  - `git tag 0.0.2-SNAPSHOT.aaa12342 aaa12342`
  - `git push --tags`
- Build-microservice-container.yaml - is intended to be triggered manually. Can result in either:
  - An existing container image in AWS ECR will be tagged with a new specified tag.
  - OR a new image is created with the specified tag.

## Sample Templates
Sample templates for calling these reuseable workflows can be found within [sample_templates](./sample_templates/). Copy both sample templates in your GitHub repository under .github/workflows and rename them. These templates can be used with little modification and each is described below.

### Sample-call-build-workflow.yaml
#### Template Changes
1. `<microservice_name>` - this name should properly describe your microservice and must match the repository created in AWS ECR.
2. `<dockerfile_relative_path>` - this is the path from the root of your GitHub repository to your dockerfile (not including the file itself). Example: ./microservice_directory

#### Variables
This workflow requires no additonal input variables.

#### Required GitHub Secrets
1. `CDC_NBS_SANDBOX_SHARED_SERVICES_ACCOUNTID` - account id for container storage.
2. `ECR_REPO_BASE_NAME` - base name for ECR to store the container image.

### Sample-call-release-workflow.yaml
Template Changes
1. `<microservice_name>` - this name should properly describe your microservice and must match the repository created in AWS ECR.
2. `<dockerfile_relative_path>` - this is the path from the root of your GitHub repository to your dockerfile (not including the file itself). Example: ./microservice_directory

#### Variables
1. `existing-image-tag` - Image tag of existing container in ECR (not used if build-new-container=true).
2. `new-image-tag` - New image tag name.
3. `build-new-container` - Check the box to create a new container tagged with new-image-tag. 
   -(NOTE: this variable is rendered as a check-box when running manually.)

#### Required GitHub Secrets
1. `CDC_NBS_SANDBOX_SHARED_SERVICES_ACCOUNTID` - account id for container storage.
2. `ECR_REPO_BASE_NAME` - base name for ECR to store the container image.
