# GitHub Runner

[![Docker Pull](https://img.shields.io/docker/pulls/thuong/github-runner)](https://hub.docker.com/r/thuong/github-runner)

Thanks to [tcardonne](https://github.com/tcardonne/docker-github-runner).
This is forked from his source and customized

-----------
GitHub allows developers to run GitHub Actions workflows on your own runners.
This Docker image allows you to create your own runners on Docker.

## Important notes

* GitHub [recommends](https://help.github.com/en/github/automating-your-workflow-with-github-actions/about-self-hosted-runners#self-hosted-runner-security-with-public-repositories) that you do **NOT** use self-hosted runners with public repositories, for security reasons.

## Usage

### Using docker
Use the following command to start listening for jobs:
```shell
docker run -it --name github-runner \
    -e RUNNER_NAME=github-runner \
    -e GITHUB_ACCESS_TOKEN=personal_token \
    -e RUNNER_ORGANIZATION_URL=https://github.com/[organization name] \
    -e RUNNER_REPLACE_EXISTING=True \
    -v /var/run/docker.sock:/var/run/docker.sock \
    thuong/github-runner:latest
```

### Using docker-compose.yml

In `docker-compose.yml` :
```yaml
version: "3.7"
services:
    github-runner:
      image: thuong/github-runner:latest
      environment:
        RUNNER_NAME: "github-runner"
        RUNNER_ORGANIZATION_URL: ${RUNNER_ORGANIZATION_URL}
        GITHUB_ACCESS_TOKEN: ${GITHUB_ACCESS_TOKEN}
        RUNNER_REPLACE_EXISTING: ${RUNNER_REPLACE_EXISTING}
      volumes:
        - /var/run/docker.sock:/var/run/docker.sock
```

You can create a `.env` file to provide environment variables when using docker-compose :
```
RUNNER_ORGANIZATION_URL=https://github.com/[organization name]
GITHUB_ACCESS_TOKEN=personal_token
RUNNER_REPLACE_EXISTING=True
```

## Environment variables

The following environment variables allows you to control the configuration parameters.

| Name | Description | Required/Default value |
|------|---------------|-------------|
| RUNNER_REPOSITORY_URL | The runner will be linked to this repository URL | Required if `RUNNER_ORGANIZATION_URL` is not provided |
| RUNNER_ORGANIZATION_URL | The runner will be linked to this organization URL. *(Self-hosted runners API for organizations is currently in public beta and subject to changes)* | Required if `RUNNER_REPOSITORY_URL` is not provided |
| GITHUB_ACCESS_TOKEN | Personal Access Token. Used to dynamically fetch a new runner token (recommended, see below). | Required if `RUNNER_TOKEN` is not provided.
| RUNNER_TOKEN | Runner token provided by GitHub in the Actions page. These tokens are valid for a short period. | Required if `GITHUB_ACCESS_TOKEN` is not provided
| RUNNER_WORK_DIRECTORY | Runner's work directory | `"_work"`
| RUNNER_NAME | Name of the runner displayed in the GitHub UI | Hostname of the container
| RUNNER_REPLACE_EXISTING | `"true"` will replace existing runner with the same name, `"false"` will use a random name if there is conflict | `"true"`

## Runner Token

In order to link your runner to your repository/organization, you need to provide a token. There is two way of passing the token :

* via `GITHUB_ACCESS_TOKEN` (recommended), containing a [Personnal Access Token](https://github.com/settings/tokens). This token will be used to dynamically fetch a new runner token, as runner tokens are valid for a short period of time.
  * For a single-repository runner, your PAT should have `repo` scopes.
  * For an organization runner, your PAT should have `admin:org` scopes.
* via `RUNNER_TOKEN`. This token is displayed in the Actions settings page of your organization/repository, when opening the "Add Runner" page.

## Runner auto-update behavior

The GitHub runner (the binary) will update itself when receiving a job, if a new release is available.
In order to allow the runner to exit and restart by itself, the binary is started by a supervisord process.
