# Actions Build Steps

This repository contains reusable Github workflows to build and push a Docker image to ACR and create/deploy/destroy a release in an Octopus dynamic environment.

## `build-planday-service.yaml`

This reusable workflow will:

1. Build and push the docker image to ACR
2. Create a dynamic environment in Octopus
3. Create an Octopus release with the built docker image and deploy it to the dynamic environment

### Inputs

| Name | Type | Required | Description |
| :---: | :---: | :---: |  --- |
| `image_name` | `string` | `true` | Name of the Docker image to build. (ex: `myapp`) |
| `docker_context` | `string` | `false` | Docker build context. Default is the default git context (current directory `.`) |
| `dockerfile_path` | `string` | `false` | Path to the Dockerfile. Default `Dockerfile` |
| `registry` | `string` | `true` | Azure Containers Registry to push the image to |
| `registrydev` | `string` | `true` | Azure dev Containers Registry to pull base images from |
| `version` | `string` | `true` | Version of the Docker image to build. (ex: `1.2.3` or `v1.2.4`) |
| `helm_version` | `string` | `false` | Helm version. Default: `3.4.0` |
| `helm_chart` | `string` | `false` | Helm chart version used to deploy the Docker image. Default: `1.0.212` |
| `gateway` | `string` | `false` | Gateaway version. Default: `1.0.212` |
| `octopus_server_url` | `string` | `true` | URL of the Octopus server |
| `octopus_project_name` | `string` | `true` | Name of the project in Octopus |
| `dockerargs` | `string` | `false` | Additional build arguments for docker, provided as a multiline string |
| `sonarqube_skip_scan` | `boolean` | `false` | Whether to skip the Sonarqube scan. Default: false |
| `sonarqube_server_url` | `string` | `true` | URL of the SonarQube server |

### Secrets

| Name | Required | Description |
| :---: | :---: |  --- |
| `acr_username` | `true` | Service Principal ID for Azure ACR |
| `acr_password` | `true` | Service Principal password for Azure ACR |
| `octopus_api_key` | `true` | API key of the Octopus server |
| `sonarqube_token` | `true` | SonarQube server token |

### Outputs

| Name | Description |
| :---: | --- |
| `release_number` | Version number of the release deployed |


### Usage

```yaml
jobs:
  build-and-push:
    uses: planday-corp/build-steps/.github/workflows/build-planday-service.yaml@v2
    with: 
      image_name: "my-api"
      version: "2.1.${{github.run_number}}"
      octopus_project_name: "Planday.Domain.Api"
      octopus_server_url: "https://xxxxxxxxxxx"
      registry: "myregistry.azurecr.io"
      registrydev: "myregistrydev.azurecr.io"
      sonarqube_server_url: "https://xxxxxxxxxxx"
      dockerargs: |
        --force-rm
        --memory=2G
    secrets:
      acr_username: ${{secrets.ACRID}}
      acr_password: ${{secrets.ACRPASSWORD}}
      octopus_api_key: ${{secrets.OCTOPUSSERVERAPIKEY}}
      sonarqube_token: ${{secrets.SONARQUBETOKEN}}
```

## `delete-planday-service.yaml`

This reusable workflow will:

1. Delete the dynamic environment in Octopus
2. Delete the associated resources in Kubernetes

### Inputs

| Name | Type | Required | Description |
| :---: | :---: | :---: |  --- |
| `octopus_server_url` | `string` | `true` | URL of the Octopus server |
| `aks_resource_group` | `string` | `true` | Name of the AKS resource group |
| `aks_cluster_name` | `string` | `true` | Name of the AKS cluster |

### Secrets

| Name | Required | Description |
| :---: | :---: |  --- |
| `octopus_api_key` | `true` | API key of the Octopus server |
| `kubernetes_azure_credentials` | `true` | Credentials of the Azure SP in json format: `{"clientId":"xxxxxxx","clientSecret":"XXXXXXXX","subscriptionId":"xxxxxx","tenantId":"xxxxx"}` |

### Usage

```yaml
jobs:
  delete-dynamic-environment:
    uses: planday-corp/build-steps/.github/workflows/delete-planday-service.yaml@v2
    with:
      octopus_server_url: "https://xxxxxxxxxxx"
      aks_resource_group: "<aks resource group name>"
      aks_cluster_name: "<aks cluster name>"
    secrets:
      octopus_api_key: ${{secrets.OCTOPUSSERVERAPIKEY}}
      kubernetes_azure_credentials: ${{secrets.KUBERNETES_AZURE_CREDENTIALS}}
```
