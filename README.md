# Gavan Lamb Website
![Pipeline](https://github.com/gavanlamb/gavanlamb.github.io/actions/workflows/release.yml/badge.svg)  
![Version](https://gavanlamb.com/badges/version.svg)  
[🌎](https://gavanlamb.com/index.html)  

## Pipelines
### When a pull request is raised
[pull-request.opened.yml](.github/workflows/pull-request.opened.yml)  
This pipeline will run on every `opened` and `reopened` pull requests.

This pipeline exists because reviewing pull requests have an overhead to the review process where information is not readily available. This pipeline aims to provide the reviewer with the necessary information to make an informed decision as well as help the developer automate work that is repetitive and time consuming.

It will:
1. Checkout the code
2. Try and get the jira issue details from the branch name - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/jira/get-issue-details/action.yml) for more information
3. Update the Jira issue status to `Review` - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/jira/change-issue-status/action.yml) for more information
4. Update the pull request title and the description with the following variables: - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/pull-request/update-pr/action.yml) for more information
   * `{{ISSUE_IDENTIFIER}}`: the Jira issue identifier. This value is retrieved from the `issueIdentifier` input parameter.
   * `{{ISSUE_TITLE}}`: the Jira issue title. This value is retrieved from the `issueTitle` input parameter.
   * `{{ISSUE_URL}}`: the Jira issue url. This value is retrieved from the `issueUrl` input parameter.
   * `{{REPOSITORY_NAME}}`: name of the github repository. This value is retrieved from the context property `${{ github.repository }}`.
   * `{{ENVIRONMENT}}`: name of the preview environment the site is deployed to. This value is retrieved from the `environment` input parameter. Each pull request has a unique environment. The environment name has a template of `preview{{PR_NUMBER}}` i.e. preview52.
   * `{{BRANCH_NAME}}`: name of the branch the PR has been raised for. This value is retrieved from the context property `${{ github.head_ref }}`.
5. Send a slack message to the slack github channel - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/slack/pull-request-created/action.yml) for more information

Variable requirements:
* `JIRA_API_TOKEN`
* `SLACK_WEBHOOK_URL`

### Pull request
[pull-request.yml](.github/workflows/pull-request.yml)  
This pipeline will run with every push to branch that currently has an active pull request that doesn't target main, yes even the release branch.

This pipeline exists because the pull request pipeline is responsible for building and deploying the assets to a preview environment. This environment is used to preview the changes before merging the pull request. This pipeline will deploy items to an ephemeral environment that will be destroyed once the pull request is closed. The environment name will be `preview{{PR_NUMBER}}` i.e. preview52.

It will:
1. build
   1. Checkout the code
   2. Generate a version number based on the version configuration [version.json](version.json) - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/version/generate/action.yml) for more information
   3. Build the Hugo site and create an artifact on the workflow run - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/hugo/build/action.yml) for more information
   4. Send a slack message to the slack github channel - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/slack/built/action.yml) for more information
2. Preview
   1. Deploy the site to a preview environment in and S3 bucket - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/hugo/deploy-to-s3/action.yml) for more information
   2. Send a slack message to the slack github channel - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/slack/released/action.yml) for more information

Variable requirements:
* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`
* `AWS_S3_CICD_BUCKET`
* `AWS_DEFAULT_REGION`
* `JIRA_API_TOKEN`
* `SLACK_WEBHOOK_URL`

### When a pull request is closed
[pull-request.closed.yml](.github/workflows/pull-request.closed.yml)
This pipeline will run on every `closed` pull requests.

This pipeline exists because the pull request pipeline is responsible for cleaning up the preview environment that was created when the pull request was raised. This pipeline will destroy the ephemeral environment that was created when the pull request was raised. The environment name will be `preview{{PR_NUMBER}}` i.e. preview52.

It will:
1. Checkout code
2. Try and get the jira issue details from the branch name - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/jira/get-issue-details/action.yml) for more information
3. Update the Jira issue status to `Done` - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/jira/change-issue-status/action.yml) for more information
4. Remove the site from the S3 bucket

Variable requirements:
* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`
* `AWS_S3_CICD_BUCKET`
* `AWS_DEFAULT_REGION`
* `JIRA_API_TOKEN`
* `SLACK_WEBHOOK_URL`

### Create Release
[create-release.yml](.github/workflows/create-release.yml)

### Release
[release.yml](.github/workflows/release.yml)
