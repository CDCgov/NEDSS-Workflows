# NEDSS Reusable Workflows
## Overview
This repository is a central location for managing reusable workflows to be used in microservices developed as a part of the NBS modernization project. GitHub Actions is the tool used to create these workflows which are intended to be adopted by any team who needs a container built and stored.

## Prerequisites
1. Request your repository be granted access to the environment containing the Elastic Container Registry (ECR).
2. Request and received confirmation that an ECR was created to store your artifact (microservice container image).

## Workflows
There are 2 processes provided build/store and updating helm charts.

## Build/Store workflows
The build/store workflows interact with container images and AWS ECR. They can be used to create and store containers in ECR or add tags to existing container within ECR. There are currently 2 flavors of each workflow: 1 for gradle projects, and 1 for 3rd party containers with custom configuration.

- Build-gradle-microservice-container.yaml - is intended to be triggered automatically whenever a push is made to the trunk (main branch). Results in a container being built and deployed to an AWS ECR. That container will be tagged identically with the gradle version, input environment classifier, and Github commit SHA. Gradle version is obtained from the gradlew printVersion command.
- Build-other-microservice-container.yaml - is intended to be triggered automatically whenever a push is made to the trunk (main branch). Results in a container being built and deployed to an AWS ECR. That container will be tagged identically with the application version found in the **FROM** section of the Dockerfile, input environment classifier, and Github commit SHA. Gradle version is obtained from the gradlew printVersion command.
  - NOTE: only the first **FROM** section is considered. 
- Release-gradle-microservice-container.yaml - is intended to be triggered manually. Can result in either:
  - An existing container image in AWS ECR will be tagged with a new tag created by the same process as Build-gradle-microservice-container.yaml.
  - OR a new image is created with the new tag created by the same process as Build-gradle-microservice-container.yaml.
- Release-gradle-microservice-container.yaml - is intended to be triggered manually. Can result in either:
  - An existing container image in AWS ECR will be tagged with a new tag created by the same process as Build-other-microservice-container.yaml.
  - OR a new image is created with the new tag created by the same process as Build-other-microservice-container.yaml.
- Update-helm-charts - is intended to be triggered after the successful completion of either build workflow. This workflow will take a newly created image tag and update the specified values file in the helm chart repository.

## Sample Templates
Sample templates for calling these reuseable workflows can be found within [sample_templates](./sample_templates/). Copy both sample templates in your GitHub repository under .github/workflows and rename them. These templates can be used with little modification and each is described below.

### Sample-call-build-workflow.yaml
#### Template Changes
Ensure "uses" section is calling the desired workflow for your project type. I.e. if you have a gradle project select the Build-gradle-microservice-container.yaml for development. 
   
1. `<microservice_name>` - this name should properly describe your microservice and must match the repository created in AWS ECR.
2. `<dockerfile_relative_path>` - this is the path from the root of your GitHub repository to your dockerfile (not including the file itself). Example: ./
microservice_directory

#### Variables
1. `<environment_classifier>` - metadata you wish to add to the result container artifact (eg. SNAPSHOT, TEST, TEST2, DEMO).
2. `<update_helm_chart>` - (true or false) Should the image tag be updated within the Helm chart?
3. `<values_file_with_path>` - What is the path to the helm chart in NBS helm chart repository? (ex. charts/elasticsearch/values.yaml).
4. `<java_version>` - Version of java which you are using to build you code.

#### Required GitHub Secrets
1. `CDC_NBS_SANDBOX_SHARED_SERVICES_ACCOUNTID` - account id for container storage.
2. `ECR_REPO_BASE_NAME` - base name for ECR to store the container image.
3. `GIT_USER_EMAIL` - Secret named GIT_USER_EMAIL for the CI user email.
4. `GIT_USER_NAME` - Secret named ECR_REPO_BASE_NAME for the CI user name.
5. `HELM_TOKEN` - Secret named HELM_TOKEN for access to helm chart repository.

### Sample-call-release-workflow.yaml
Template Changes
Ensure "uses" section is calling the desired workflow for your project type. I.e. if you have a gradle project select the Release-gradle-microservice-container.yaml for release to upper environments.  
  
1. `<microservice_name>` - this name should properly describe your microservice and must match the repository created in AWS ECR.
2. `<dockerfile_relative_path>` - this is the path from the root of your GitHub repository to your dockerfile (not including the file itself). Example: ./microservice_directory
3. `<environment_classifier>` - metadata you wish to add to the result container artifact (eg. SNAPSHOT, TEST, TEST2, DEMO).
4. `<update_helm_chart>` - (true or false) Should the image tag be updated within the Helm chart?
5. `<values_file_with_path>` - What is the path to the helm chart in NBS helm chart repository? (ex. charts/elasticsearch/values.yaml).
6. `<java_version>` - Version of java which you are using to build you code.

#### Variables
1. `existing-image-tag` - Image tag of existing container in ECR (not used if build-new-container=true).
3. `build-new-container` - Check the box to create a new container tagged with new-image-tag. 
   -(NOTE: this variable is rendered as a check-box when running manually.)

#### Required GitHub Secrets
1. `CDC_NBS_SANDBOX_SHARED_SERVICES_ACCOUNTID` - account id for container storage.
2. `ECR_REPO_BASE_NAME` - base name for ECR to store the container image.
3. `GIT_USER_EMAIL` - Secret named GIT_USER_EMAIL for the CI user email.
4. `GIT_USER_NAME` - Secret named ECR_REPO_BASE_NAME for the CI user name.

