
# Continuous Integration Workflow

## General idea

* Everytime you push a new commit, a GitHub Workflow will be triggered.
* This workflow will build and push the Docker image containing the operator code and its execution environment .
* This workflow is specified in a YAML file you can find in the `/.github/workflows` directory.
* In this chapter, we will explain each section of the `/.github/workflows/ci.yaml` step-by-step to clarify its purpose and functionality.

## 1. Workflow Name and Trigger

```yaml
name: CI Workflow

on:
  push:
    branches: ['main']
```

The workflow is named **CI Workflow**, which is mainly for identification purposes within the GitHub Actions dashboard. The workflow is triggered by a `push` event, but only for the `main` branch. This means that every time code is pushed to the `main` branch of the repository, this workflow will automatically run.

## 2. Environment Variables

```yaml
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
```

Two environment variables are defined here:

- **REGISTRY**: This is set to `ghcr.io`, which stands for **GitHub Container Registry**. It is the domain where the Docker images will be stored.
  
- **IMAGE_NAME**: This variable is dynamically set to the name of the GitHub repository using `github.repository`. The `${{ github.repository }}` syntax is GitHub's way of accessing context values, such as the repository name. 

The image name will be used later when tagging the Docker image.

## 3. Job Definition: `build-and-push-image`

The core of the workflow is defined within a single job called `build-and-push-image`.

```yaml
jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      attestations: write
      id-token: write
```

- **runs-on: ubuntu-latest**: The job will run on the latest available version of Ubuntu. This provides a virtual machine with all the necessary tools to run Docker and interact with GitHub.

- **permissions**: This section grants specific permissions to the job:
  - `contents: read`: Allows reading the repository's contents (code, files).
  - `packages: write`: Allows publishing Docker images to GitHub Packages.
  - `attestations: write`: Grants access to manage attestations, which are metadata related to builds.
  - `id-token: write`: Allows the job to issue ID tokens for authentication, commonly used with OpenID Connect for secure token exchanges.

## 4. Steps in the Workflow

This job consists of several steps, each performing a crucial task in the workflow.

### Step 1: Checkout Repository

```yaml
- name: Checkout repository
  uses: actions/checkout@v4
```

The first step uses the `actions/checkout` action, version `v4`. This action checks out the repository code so that it can be accessed in the subsequent steps. Without this step, the job wouldn't be able to access the files in the repository.

### Step 2: Log in to the Container Registry

```yaml
- name: Log in to the Container registry
  uses: docker/login-action@v3.3.0
  with:
    registry: ${{ env.REGISTRY }}
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```

In this step, the workflow logs into GitHub's Container Registry (`ghcr.io`) using the `docker/login-action`. This action allows you to authenticate to a container registry, enabling the workflow to push Docker images.

- **registry**: Set to the value of `${{ env.REGISTRY }}` (`ghcr.io`).
- **username**: Set to `${{ github.actor }}`, which represents the username of the person or bot that triggered the workflow.
- **password**: Set to `${{ secrets.GITHUB_TOKEN }}`, a built-in secret provided by GitHub to authenticate the workflow. This token allows the workflow to access the repository's packages and perform the necessary actions.

### Step 3: Extract Docker Metadata (Tags and Labels)

```yaml
- name: Extract metadata (tags, labels) for Docker
  id: meta
  uses: docker/metadata-action@v5.5.1
  with:
    images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
```

This step uses the `docker/metadata-action` to extract metadata (such as tags and labels) from the repository and build context. The extracted metadata will be used later when tagging the Docker image.

- **id: meta**: This assigns an identifier `meta` to the step, making its outputs accessible in later steps.
- **images**: This specifies the base name for the Docker image, which is constructed from the container registry and the repository name.

The output from this step is crucial for correctly tagging and labeling the Docker image in the next step.

### Step 4: Build and Push Docker Image

```yaml
- name: Build and push Docker image
  id: push
  uses: docker/build-push-action@v6.7.0
  with:
    context: .
    push: true
    tags: ${{ steps.meta.outputs.tags }}
    labels: ${{ steps.meta.outputs.labels }}
```

In this final step, the `docker/build-push-action` is used to:

1. **Build** a Docker image using the repository's `Dockerfile`.
2. **Push** the built image to the GitHub Container Registry.

Key parameters in this step:

- **context**: Set to `.` (the current directory). This specifies the build context, meaning the files in the repository's root directory will be used for the Docker build.
- **push**: Set to `true`. This ensures the Docker image is automatically pushed to the registry after it is built.
- **tags**: The tags for the image, provided by the output of the `meta` step (`${{ steps.meta.outputs.tags }}`).
- **labels**: Additional labels for the image, also provided by the `meta` step (`${{ steps.meta.outputs.labels }}`).

### Optional: Personal Access Token

```yaml
# A PAT is needed for this action, GITHUB_TOKEN cannot get relevant permission 
# secrets: |
#   github_pat=${{ secrets.GH_PAT }}
```

This is an optional comment explaining that if higher-level permissions are required, a Personal Access Token (PAT) should be used instead of the default `GITHUB_TOKEN`. This would be defined as a secret (`GH_PAT`).

## Conclusion

This CI pipeline builds and pushes Docker images to GitHub Container Registry every time changes are made to the `main` branch. The workflow:

1. Checks out the repository.
2. Logs into the container registry.
3. Extracts metadata to tag and label the image.
4. Builds and pushes the Docker image to the registry.

By using GitHub Actions and Docker together, this workflow automates the process of containerizing code and storing it in a centralized registry, enabling future installation from Tercen.

# Release Workflow

Once you are satisfied with your operator, you can release your operator by following the steps described below.

## 1. Edit the operator.json file

You should first edit the __container__ field of the __operator.json__ file so that it matched the version number you would like to push (here, __0.0.1__ for example).

## 2. Push your changes and tag the repository

After pushing your latest commit, you can add the same version number as a tag from the command line as follows:

```bash
git tag 0.0.1
```

Then push it with this command:

```bash
git push --tags
```

## 3. Wait and check the results of the release workflow

The same way a workflow is triggered after each commit, tagging a repository will trigger another workflow, __release.yaml__. This workflow that will build and push the Docker image, tag it with the run the unit tests. 
Once the workflow is run, you can verify it has been successful. If that is the case, you are now ready to install your operator in a __Library__.