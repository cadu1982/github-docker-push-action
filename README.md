# Action to Push a Docker Imagem into GHCR

This **[composite action](https://docs.github.com/en/enterprise-server@3.4/actions/creating-actions/creating-a-composite-action)** is designed to be used in the **["Build and Deploy a Dotnet Application"](https://github.com/owner/github-build-deploy-dotnet-workflow)** workflow, for example. After generating a Docker image from the **Dockerfile**, and doing some testing and validations, it's time to send that image to run in a container on the **GitHub Container Registry**. So this action will take some inputs to do just that.

<br>

## Inputs

As inputs to this action, five required variables were passed:

<br>

```yaml
inputs:
  GITHUB_REPO_OWNER:
    required: true
    description: Repository owner
  GITHUB_REPO_NAME:
    required: true
    description: Repository name
  GENERAL_TAG:
    required: true
    description: Docker image tag
  GITHUB_TOKEN:
    required: true
    description: Token to access the GitHub Container Registry
  GITHUB_ACTOR:
    required: true
    description: Who starts the application
```

<br>

## What do these variables do?

- **`GITHUB_REPO_OWNER`**: this input represents the **repository owner**, which can be an organization or another one. For example, if the repository where the **Caller Workflow** that use this action is: **[owner/ms-exemplo](https://github.com/owner/ms-exemplo)**, the **`GITHUB_REPO_OWNER`** will store the value: **"owner"**;

  

- **`GITHUB_REPO_NAME`**: this input represents the main **repository name** that is calling this action to run definitely, for example, the Caller Workflow repository name;

  

- **`GENERAL_TAG`**: this input is used as a tag inserted in the docker image name together with the **`latest`** tag. **`GENERAL_TAG`** is composed of the run date following the pattern **yyyy-mm-dd** and the **`GITHUB_RUN_NUMBER`** environment variable, which represents a unique number for each run of a particular workflow in a repository;

  

- **`GITHUB_TOKEN`**: in this input, a token value was passed to access the **GitHub Container Registry**. This input has the value of the **`GITHUB_TOKEN`** secret, which is a token that can be used to authenticate on behalf of **GitHub Actions**. So, at the start of each workflow run, GitHub automatically creates a unique **`GITHUB_TOKEN`** secret to use in your workflow. You can use the **`GITHUB_TOKEN`** to authenticate in a workflow run. For more details, click [**here**](https://docs.github.com/en/actions/security-guides/automatic-token-authentication);

  

- **`GITHUB_ACTOR`**: this input stores the username of the user that initiated the workflow run. As this action depends on being called by a workflow, the value will be the username that initiated the run of the **Caller Workflow** that is using this action.

<br>

## What does this action do?

This composite action runs three other actions:



### Set up Docker Buildx

The [**docker/setup-buildx-action**](https://github.com/docker/setup-buildx-action) action will create and boot a builder using by default the **`docker-container`** [**builder driver**](https://github.com/docker/buildx/blob/master/docs/reference/buildx_create.md#driver). This is **not required but recommended** using it to be able to build multi-platform images, export cache, etc.

<br>

```yaml
runs:
  using: "composite"
  steps:
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
```

<br>

### Login to GitHub Container Registry

The [**docker/build-push-action**](https://github.com/docker/login-action) is used to store and manage Docker and OCI images in the Container Registry, which uses the package namespace **`https://ghcr.io`**. To authenticate to the **Container registry** within a GitHub Actions workflow, is recommended to use the **`GITHUB_TOKEN`** for the best security and experience:

<br>

```yaml
- name: Login to GitHub Container Registry
  uses: docker/login-action@v2
  with:
    registry: ghcr.io
    username: ${{ inputs.GITHUB_ACTOR }}
    password: ${{ inputs.GITHUB_TOKEN }}
```

<br>

### Build and Push a Docker Image

The [**docker/build-push-action**](https://github.com/docker/build-push-action) is responsible for the build and pushing of Docker images with **Buildx**. Please note that now we are uploading the image (**`push: true`**) to **GitHub Container Registry** once, probably, this image **had already passed some validations** and **tests** and is **ready** to be published.

<br>

To build images, you will need to have a **Dockerfile** configured in the main repository that will run definitely this action. So, if you have a **Called Workflow** (for example [**github-build-deploy-dotnet-workflow**](https://github.com/owner/github-build-deploy-dotnet-workflow)) that uses this action, but this workflow is beeing calling by a **Caller Workflow**, the **Dockerfile** needs to be in **Caller Workflow repository**, for example: [**owner/ms-exemplo**](https://github.com/owner/ms-exemplo).

<br>

The **`tags`** key will add the **`latest`** and **`GENERAL_TAG`** tags to the Docker images generated.

<br>

```yaml
- name: Build and Push a Docker Image
  uses: docker/build-push-action@v2
  with:
    push: true
    tags: ghcr.io/${{ inputs.GITHUB_REPO_OWNER }}/${{ inputs.GITHUB_REPO_NAME }}:latest, ghcr.io/${{ inputs.GITHUB_REPO_OWNER }}/${{ inputs.GITHUB_REPO_NAME }}:${{ inputs.GENERAL_TAG }}
```

<br>

## Dependencies and Requirements

To use this action, you can't forget to have the **Dockerfile** configured in the repository where the **Caller Workflow** is. Also, don't forget to become this repository accessible from repositories in your organization by accessing repository **Settings > Actions > General > Access > Accessible from repositories in the 'Organization_Name' organization**.