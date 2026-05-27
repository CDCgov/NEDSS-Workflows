# NEDSS Reusable Workflows and custom GitHub actions
## Overview
This repository is a central location for managing reusable workflows to be used in microservices developed as a part of the NBS modernization project for consistent CI/CD processes. GitHub Actions is the tool used to create these workflows which are intended to be adopted by any team who needs any of the services provided below.

## Prerequisites for container related workflows
1. Request your repository be granted access to the environment containing the Elastic Container Registry (ECR).
2. Request and received confirmation that an ECR was created to store your artifact (microservice container image).

## Usage
Reusable workflows are meant to be easily picked up and placed in your repositories CI/CD pipeline. To further this effort [sample_templates](./sample_templates/) are provided.
1. [Sample-call-build-and-deploy-workflow.yaml](./sample_templates/Sample-call-build-and-deploy-workflow.yaml) - this workflow is intended to be used when container images need to be built. It promotes automated deployment by modifiying a helm charts values.yaml file.
  - Note 1: This is a general template and a full list of variables can be found below.
  - Note 2: This template only references Build-other-microservice-container.yaml and the `uses` line for the call-build-microservice-container-workflow job should be changed to reflect the intended reusuable workflow.
2. [Sample-call-release-and-deploy-workflow.yaml](./sample_templates/Sample-call-release-and-deploy-workflow.yaml) - this workflow is intended to be used when container release are required either using a new container or tagging an existing container image in AWS ECR. It promotes automated deployment by modifiying a helm charts values.yaml file.
  - Note 1: This is a general template and a full list of variables can be found below.
  - Note 2: This template only references Build-other-microservice-container.yaml and the `uses` line for the call-build-microservice-container-workflow job should be changed to reflect the intended reusuable workflow.
3. [Sample-call-trivy-container-scan.yaml](./sample_templates/Sample-call-trivy-container-scan.yaml) - this workflow is intended to be used when container scans are required in addition to those run automatically in the build and release workflows.
4. [Create-github-draft-release.yaml](./workflows/Create-github-draft-release.yaml) - This workflow creates a draft release within GitHub and upload an artifact. In addition, there is an update only mode which will update the artifact in an existing draft release.



## Custom Github Actions
### [Trivy-Scanner](.github/actions/trivy-scanner/action.yaml)
This action uses Trivy to scan built container images for vulnerabilties and output results either within the runner logs or to the GitHub Security tab.

#### Input Variables
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| container-ref | string |  | 'Locally built container id which Trivy should scan' | true |
| exit-code | string | '0' | 'Exit code when specified vulnerabilities are found (0).' | false |
| ignore-unfixed | boolean | false | 'Ignore unpatched/unfixed vulnerabilities' | false |
| limit-severities-for-sarif | boolean | true | 'By default SARIF format enforces output of all vulnerabilities regardless of configured severities. To override this behavior set this parameter to true' | false |
| severity | string | 'CRITICAL,HIGH' | 'Severities of vulnerabilities to be scanned for and displayed (UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL)' | false |
| skip-dirs | string | '' | 'Comma separated list of directories where traversal is skipped' | false |
| skip-files | string | '' | 'Comma separated list of files where traversal is skipped' | false |
| timeout | string | '10m0s' | 'Scan timeout duration' | false |
| trivyignores | string |  | 'Comma-separated list of relative paths in repository to one or more .trivyignore files, for usage see https://aquasecurity.github.io/trivy/v0.19.2/vulnerability/examples/filter/' | false |
| upload-to-github-security-tab | boolean | false | 'Upload results to GitHub security tab?' | false |

#### Outputs
None

## Workflows

### [Build-gradle-microservice-container.yaml](./workflows/Build-gradle-microservice.yaml)
This workflow build a container and push it to ECR. Application versioning is obtained using `./gradlew printVersion` from the GitHub Repositories root directory and tacking on some metadata. Uses [Trivy-Scanner](.github/actions/trivy-scanner/action.yaml) for container scanning.

#### Input Variables
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| dockerfile_relative_path | string |  | 'Relative path to dockerfile being built (use '-f' docker argument if the dockerfile referenced from the root directory).' | true |
| environment_classifier | string |  | 'Metadata to append to application version. Ex. if version=1.0.0, and environment_classifier=SNAPSHOT result will be 1.0.0-SNAPSHOT.<githubsha>.' | true |
| exit-code | string | '0' | 'Exit code when specified vulnerabilities are found (0).' | false |
| ignore-unfixed | boolean | false | 'Ignore unpatched/unfixed vulnerabilities' | false |
| java_version | string | '17' | 'Version of java which you are using to build you code.' | false |
| limit-severities-for-sarif | boolean | true | 'By default SARIF format enforces output of all vulnerabilities regardless of configured severities. To override this behavior set this parameter to true' | false |
| microservice_name | string |  | 'Name of microservice corresponding to a container in ECR.' | true |
| severity | string | 'CRITICAL,HIGH' | 'Severities of vulnerabilities to be scanned for and displayed (UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL)' | false |
| skip-dirs | string | '' | 'Comma separated list of directories where traversal is skipped' | false |
| skip-files | string | '' | 'Comma separated list of files where traversal is skipped' | false |
| timeout | string | '10m0s' | 'Scan timeout duration' | false |
| trivyignores | string |  | 'Comma-separated list of relative paths in repository to one or more .trivyignore files, for usage see https://aquasecurity.github.io/trivy/v0.19.2/vulnerability/examples/filter/' | false |
| upload-to-github-security-tab | boolean | true | 'Upload results to GitHub security tab?' | false |
| create_octopus_release | boolean | false | Create an Octopus Deploy release after successfully pushing to ECR. If false, it will still default to true on the main branch if the Octopus API key ARN is available. | false |
| octopus_project | string | | Octopus Deploy project name to create a release for. Defaults to `microservice_name` if not provided. | false |
| octopus_channel | string | 'alpha-main' | Octopus Deploy release channel. | false |
| octopus_space | string | 'Default' | Octopus Deploy space name. | false |
| octopus_server_url | string | 'https://octopus.skylightnbs.com' | Octopus Deploy server URL. | false |
| octopus_api_key_secret_name | string | | Name or ARN of the AWS Secrets Manager secret holding the Octopus Deploy API key. Defaults to the `AWS_OCTOPUS_API_KEY_ARN` org/repo variable if not provided. | false |

#### Input Secrets
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| NBS_ACCOUNTID | string | | 'AWS Account ID where ECR resides.' | false |
| CDC_NBS_SANDBOX_SHARED_SERVICES_ACCOUNTID | string |  | 'Backward compatible AWS Account ID where ECR resides.' | false |
| ECR_REPO_BASE_NAME | string |  | 'Secret named ECR_REPO_BASE_NAME where ECR resides.' | true |
| GIT_USER_EMAIL | string |  | 'Secret named GIT_USER_EMAIL for the CI user email.' | false |
| GIT_USER_NAME | string |  | 'Secret named ECR_REPO_BASE_NAME for the CI user name.' | false |
| HELM_TOKEN | string |  | 'Secret named HELM_TOKEN to access helm chart repository' | false |

#### Outputs
| Key | Type | Description |
| -------------- | -------------- | -------------- |
| output_image_tag | string | "Container image tag"  |

#### Octopus Deploy Release Creation
When `create_octopus_release` is enabled (or when running on the `main` branch), a release is automatically created in Octopus Deploy after a successful ECR push. The workflow uses a centralized mapping file to resolve naming discrepancies and handle complex multi-service projects.

**Features:**
- **Automatic on Main:** Releases are automatically created on push to the `main` branch if the Octopus API key ARN is available.
- **Centralized Mapping:** Uses [octopus-mapping.yaml](./octopus-mapping.yaml) to map `microservice_name` to the correct Octopus Project and Package ID. This allows for "zero-touch" integration in downstream repositories.
- **Multi-Service Support:** For projects that deploy multiple services (e.g., Modernization stack), building one service will create a release that includes the new version for that service while automatically picking the latest available versions for its siblings.
- **Flexible Secrets:** Supports both `NBS_ACCOUNTID` and `CDC_NBS_SANDBOX_SHARED_SERVICES_ACCOUNTID` for AWS authentication.

**Prerequisites:**
- The `AWS_OCTOPUS_API_KEY_ARN` variable must be set at the org or repo level (or passed via `octopus_api_key_secret_name`).
- The `github-actions-nbs-deployment-role` IAM role must have `secretsmanager:GetSecretValue` permission on the API key secret.

**Example — calling workflow in a downstream repo:**
```yaml
jobs:
  build:
    uses: CDCgov/NEDSS-Workflows/.github/workflows/Build-gradle-microservice-container.yaml@main
    with:
      microservice_name: my-service
      dockerfile_relative_path: Dockerfile
      environment_classifier: SNAPSHOT
    secrets:
      NBS_ACCOUNTID: ${{ secrets.NBS_ACCOUNTID }}
```
*Note: If `my-service` is defined in `octopus-mapping.yaml`, no additional Octopus configuration is needed.*

The release number in Octopus will match the ECR image tag (e.g. `1.0.0-SNAPSHOT.abc1234`), giving full traceability between the Octopus release, the container image, and the git commit.


### [Build-other-microservice-container.yaml](./workflows/Build-other-microservice-container.yaml)
This workflow build a container and push it to ECR. Application versioning is obtained using from the dockerfile after the initial `FROM` block (e.g. `FROM elasticsearch:v1.0.0` results in `v1.0.0`). Uses [Trivy-Scanner](.github/actions/trivy-scanner/action.yaml) for container scanning.

#### Input Variables
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| dockerfile_relative_path | string |  | 'Relative path to dockerfile being built (use '-f' docker argument if the dockerfile referenced from the root directory).' | true |
| environment_classifier | string |  | 'Metadata to append to application version. Ex. if version=1.0.0, and environment_classifier=SNAPSHOT result will be 1.0.0-SNAPSHOT.<githubsha>.' | true |
| exit-code | string | '0' | 'Exit code when specified vulnerabilities are found (0).' | false |
| ignore-unfixed | boolean | false | 'Ignore unpatched/unfixed vulnerabilities' | false |
| limit-severities-for-sarif | boolean | true | 'By default SARIF format enforces output of all vulnerabilities regardless of configured severities. To override this behavior set this parameter to true' | false |
| microservice_name | string |  | 'Name of microservice corresponding to a container in ECR.' | true |
| severity | string | 'CRITICAL,HIGH' | 'Severities of vulnerabilities to be scanned for and displayed (UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL)' | false |
| skip-dirs | string | '' | 'Comma separated list of directories where traversal is skipped' | false |
| skip-files | string | '' | 'Comma separated list of files where traversal is skipped' | false |
| timeout | string | '10m0s' | 'Scan timeout duration' | false |
| trivyignores | string |  | 'Comma-separated list of relative paths in repository to one or more .trivyignore files, for usage see https://aquasecurity.github.io/trivy/v0.19.2/vulnerability/examples/filter/' | false |
| upload-to-github-security-tab | boolean | true | 'Upload results to GitHub security tab?' | false |
| create_octopus_release | boolean | false | Create an Octopus Deploy release after successfully pushing to ECR. If false, it will still default to true on the main branch if the Octopus API key ARN is available. | false |
| octopus_project | string | | Octopus Deploy project name to create a release for. Defaults to `microservice_name` if not provided. | false |
| octopus_channel | string | 'alpha-main' | Octopus Deploy release channel. | false |
| octopus_space | string | 'Default' | Octopus Deploy space name. | false |
| octopus_server_url | string | 'https://octopus.skylightnbs.com' | Octopus Deploy server URL. | false |
| octopus_api_key_secret_name | string | | Name or ARN of the AWS Secrets Manager secret holding the Octopus Deploy API key. Defaults to the `AWS_OCTOPUS_API_KEY_ARN` org/repo variable if not provided. | false |

#### Input Secrets
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| NBS_ACCOUNTID | string | | 'AWS Account ID where ECR resides.' | false |
| CDC_NBS_SANDBOX_SHARED_SERVICES_ACCOUNTID | string |  | 'Backward compatible AWS Account ID where ECR resides.' | false |
| ECR_REPO_BASE_NAME | string |  | 'Secret named ECR_REPO_BASE_NAME where ECR resides.' | true |
| GIT_USER_EMAIL | string |  | 'Secret named GIT_USER_EMAIL for the CI user email.' | false |
| GIT_USER_NAME | string |  | 'Secret named ECR_REPO_BASE_NAME for the CI user name.' | false |
| HELM_TOKEN | string |  | 'Secret named HELM_TOKEN to access helm chart repository' | false |

#### Outputs
| Key | Type | Description |
| -------------- | -------------- | -------------- |
| output_image_tag | string | "Container image tag"  |

### [Release-gradle-microservice-container.yaml](./workflows/Release-gradle-microservice.yaml)
This workflow has 2 runtime options. The first option is to tag an *existing* container in AWS ECR with a new container tag. The second option is to *on-demand* build a container and push it to ECR. Application versioning is obtained using `./gradlew printVersion` from the GitHub Repositories root directory and tacking on some metadata. Uses [Trivy-Scanner](.github/actions/trivy-scanner/action.yaml) for container scanning.

#### Input Variables
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| build-new-container | boolean | false | '(true or false) Should a new container be built.'  | true |
| dockerfile_relative_path | string |  | 'Relative path to dockerfile being built (use '-f' docker argument if the dockerfile referenced from the root directory).' | true |
| environment_classifier | string |  | 'Metadata to append to application version. Ex. if version=1.0.0, and environment_classifier=SNAPSHOT result will be 1.0.0-SNAPSHOT.<githubsha>.' | true |
| existing-image-tag | string |  | 'Image tag of existing container in ECR (not used if build-new-container=true).'  | true |
| exit-code | string | '0' | 'Exit code when specified vulnerabilities are found (0).' | false |
| ignore-unfixed | boolean | false | 'Ignore unpatched/unfixed vulnerabilities' | false |
| java_version | string | '17' | 'Version of java which you are using to build you code.' | false |
| limit-severities-for-sarif | boolean | true | 'By default SARIF format enforces output of all vulnerabilities regardless of configured severities. To override this behavior set this parameter to true' | false |
| microservice_name | string |  | 'Name of microservice corresponding to a container in ECR.' | true |
| severity | string | 'CRITICAL,HIGH' | 'Severities of vulnerabilities to be scanned for and displayed (UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL)' | false |
| skip-dirs | string | '' | 'Comma separated list of directories where traversal is skipped' | false |
| skip-files | string | '' | 'Comma separated list of files where traversal is skipped' | false |
| timeout | string | '10m0s' | 'Scan timeout duration' | false |
| trivyignores | string |  | 'Comma-separated list of relative paths in repository to one or more .trivyignore files, for usage see https://aquasecurity.github.io/trivy/v0.19.2/vulnerability/examples/filter/' | false |
| upload-to-github-security-tab | boolean | true | 'Upload results to GitHub security tab?' | false |

#### Input Secrets
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| CDC_NBS_SANDBOX_SHARED_SERVICES_ACCOUNTID | string |  | 'Secret named CDC_NBS_SANDBOX_SHARED_SERVICES_ACCOUNTID where ECR resides.' | true |
| ECR_REPO_BASE_NAME | string |  | 'Secret named ECR_REPO_BASE_NAME where ECR resides.' | true |
| GIT_USER_EMAIL | string |  | 'Secret named GIT_USER_EMAIL for the CI user email.' | false |
| GIT_USER_NAME | string |  | 'Secret named ECR_REPO_BASE_NAME for the CI user name.' | false |
| HELM_TOKEN | string |  | 'Secret named HELM_TOKEN to access helm chart repository' | false |

#### Outputs
| Key | Type | Description |
| -------------- | -------------- | -------------- |
| output_image_tag | string | "Container image tag"  |

### [Release-other-microservice-container.yaml](./workflows/Release-other-microservice-container.yaml)
This workflow has 2 runtime options. The first option is to tag an *existing* container in AWS ECR with a new container tag. The second option is to *on-demand* build a container and push it to ECR. Application versioning is obtained using from the dockerfile after the initial `FROM` block (e.g. `FROM elasticsearch:v1.0.0` results in `v1.0.0`). Uses [Trivy-Scanner](.github/actions/trivy-scanner/action.yaml) for container scanning.

#### Input Variables
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| build-new-container | boolean | false | '(true or false) Should a new container be built.'  | true |
| dockerfile_relative_path | string |  | 'Relative path to dockerfile being built (use '-f' docker argument if the dockerfile referenced from the root directory).' | true |
| environment_classifier | string |  | 'Metadata to append to application version. Ex. if version=1.0.0, and environment_classifier=SNAPSHOT result will be 1.0.0-SNAPSHOT.<githubsha>.' | true |
| existing-image-tag | string |  | 'Image tag of existing container in ECR (not used if build-new-container=true).'  | true |
| exit-code | string | '0' | 'Exit code when specified vulnerabilities are found (0).' | false |
| ignore-unfixed | boolean | false | 'Ignore unpatched/unfixed vulnerabilities' | false |
| limit-severities-for-sarif | boolean | true | 'By default SARIF format enforces output of all vulnerabilities regardless of configured severities. To override this behavior set this parameter to true' | false |
| microservice_name | string |  | 'Name of microservice corresponding to a container in ECR.' | true |
| severity | string | 'CRITICAL,HIGH' | 'Severities of vulnerabilities to be scanned for and displayed (UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL)' | false |
| skip-dirs | string | '' | 'Comma separated list of directories where traversal is skipped' | false |
| skip-files | string | '' | 'Comma separated list of files where traversal is skipped' | false |
| timeout | string | '10m0s' | 'Scan timeout duration' | false |
| trivyignores | string |  | 'Comma-separated list of relative paths in repository to one or more .trivyignore files, for usage see https://aquasecurity.github.io/trivy/v0.19.2/vulnerability/examples/filter/' | false |
| upload-to-github-security-tab | boolean | true | 'Upload results to GitHub security tab?' | false |

#### Input Secrets
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| CDC_NBS_SANDBOX_SHARED_SERVICES_ACCOUNTID | string |  | 'Secret named CDC_NBS_SANDBOX_SHARED_SERVICES_ACCOUNTID where ECR resides.' | true |
| ECR_REPO_BASE_NAME | string |  | 'Secret named ECR_REPO_BASE_NAME where ECR resides.' | true |
| GIT_USER_EMAIL | string |  | 'Secret named GIT_USER_EMAIL for the CI user email.' | false |
| GIT_USER_NAME | string |  | 'Secret named ECR_REPO_BASE_NAME for the CI user name.' | false |
| HELM_TOKEN | string |  | 'Secret named HELM_TOKEN to access helm chart repository' | false |

#### Outputs
| Key | Type | Description |
| -------------- | -------------- | -------------- |
| output_image_tag | string | "Container image tag"  |

### [Trivy-container-scan.yaml](./workflows/Trivy-container-scan.yaml)
This workflow builds a container and scans it for security vulnerabilities using [Trivy-Scanner](.github/actions/trivy-scanner/action.yaml).

#### Input Variables
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| dockerfile_relative_path | string |  | 'Relative path to dockerfile being built (use '-f' docker argument if the dockerfile referenced from the root directory).' | true |
| exit-code | string | '0' | 'Exit code when specified vulnerabilities are found (0).' | false |
| ignore-unfixed | boolean | false | 'Ignore unpatched/unfixed vulnerabilities' | false |
| limit-severities-for-sarif | boolean | true | 'By default SARIF format enforces output of all vulnerabilities regardless of configured severities. To override this behavior set this parameter to true' | false |
| microservice_name | string |  | 'Name of microservice corresponding to a container in ECR.' | true |
| severity | string | 'CRITICAL,HIGH' | 'Severities of vulnerabilities to be scanned for and displayed (UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL)' | false |
| skip-dirs | string | '' | 'Comma separated list of directories where traversal is skipped' | false |
| skip-files | string | '' | 'Comma separated list of files where traversal is skipped' | false |
| timeout | string | '10m0s' | 'Scan timeout duration' | false |
| trivyignores | string |  | 'Comma-separated list of relative paths in repository to one or more .trivyignore files, for usage see https://aquasecurity.github.io/trivy/v0.19.2/vulnerability/examples/filter/' | false |
| upload-to-github-security-tab | boolean | false | 'Upload results to GitHub security tab?' | false |

#### Input Variables
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| values_file_with_path | string |  | 'Relative path Helm chart values file, including the file itself (Ex. elasticsearch/values-dev.yaml) | true |
| new_image_tag | string |  | 'Image tag to add to helm chart' | true |
| microservice_name | string |  | 'Name of microservice corresponding to a container in ECR.' | true |

#### Input Secrets
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| GIT_USER_EMAIL | string |  | 'Secret named GIT_USER_EMAIL for the CI user email.' | true |
| GIT_USER_NAME | string |  | 'Secret named ECR_REPO_BASE_NAME for the CI user name.' | true |
| HELM_TOKEN | string |  | 'Secret named HELM_TOKEN to access helm chart repository' | true |

#### Outputs
None

### [Update-helm-chart.yaml](./workflows/Update-helm-chart.yaml)
This workflow takes in an image tag and updates a supplied helm chart values.yaml file in the [NEDSS-Helm](https://github.com/CDCgov/NEDSS-Helm) GitHub repository.

#### Input Variables
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| values_file_with_path | string |  | 'Relative path Helm chart values file, including the file itself (Ex. elasticsearch/values-dev.yaml) | true |
| new_image_tag | string |  | 'Image tag to add to helm chart' | true |
| microservice_name | string |  | 'Name of microservice corresponding to a container in ECR.' | true |

#### Input Secrets
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| GIT_USER_EMAIL | string |  | 'Secret named GIT_USER_EMAIL for the CI user email.' | true |
| GIT_USER_NAME | string |  | 'Secret named ECR_REPO_BASE_NAME for the CI user name.' | true |
| HELM_TOKEN | string |  | 'Secret named HELM_TOKEN to access helm chart repository' | true |

#### Outputs
None

### [Create-github-draft-release.yaml](./workflows/Create-github-draft-release.yaml)
This workflow creates a draft release and an artifact.

#### Input Variables
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| dockerfile_relative_path | string |  | 'Relative path to dockerfile being built (use '-f' docker argument if the dockerfile referenced from the root directory).' | true |


#### Input Variables
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| update_zip_only | boolean | false | 'Will delete and update the artifact from an existing draft release (contents depend on selected branch/tag).' | true |
| body | string |  | 'A description of your release in markdown format (default is to autogenerate release notes).' | false |
| release_version | string |  | 'A release version (no 'v', this is added only to the final) to be created upon publishing the draft release (tag must not already exist in repository).' | true |
| release_name | string | "NONE" | 'Provide a custom name for your release. If none is provided the release name will match the provided release_version (default=NONE).' | false |
| artifact_base_name | string |  | 'Base name of the created artifact. The artifact_release_version is appended to this name.' | true |
| artifact_release_version | string |  | 'The artifact release version (no 'v', this is added only to the final).' | true |
| paths | string |  | 'A CSV string detailing which files and directories should be included in the artifact. If not provided only the standard artifacts will be created.' | true |
| excluded_paths | string | "" | 'A CSV list detailing specific files and directories to exclude from the provided paths (this variable serves only to limit scope of the paths variable).' | false |


#### Input Secrets
| Key | Type | Default | Description | Required |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| GIT_USER_EMAIL | string |  | 'Secret named GIT_USER_EMAIL for the CI user email.' | true |
| GIT_USER_NAME | string |  | 'Secret named ECR_REPO_BASE_NAME for the CI user name.' | true |

#### Outputs
None
