# Gavan Lamb Website
![workflow](https://github.com/gavanlamb/gavanlamb.github.io/actions/workflows/release.yml/badge.svg)  
![Version](https://gavanlamb.com/badges/version.svg)  
[🌎](https://gavanlamb.com/index.html)  

## workflows
### Pull request raised
[pull-request.opened.yml](.github/workflows/pull-request.opened.yml)  

This workflow will run on every `opened` and `reopened` pull requests.

This workflow exists because reviewing pull requests have an overhead to the review process where information is not readily available. This workflow aims to provide the reviewer with the necessary information to make an informed decision as well as help the developer automate work that is repetitive and time consuming.

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

This workflow will run with every push to branch that currently has an active pull request that doesn't target main, yes even the release branch.

This workflow exists because the pull request workflow is responsible for building and deploying the assets to a preview environment. This environment is used to preview the changes before merging the pull request. This workflow will deploy items to an ephemeral environment that will be destroyed once the pull request is closed. The environment name will be `preview{{PR_NUMBER}}` i.e. preview52.

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

### Pull request closed
[pull-request.closed.yml](.github/workflows/pull-request.closed.yml)  

This workflow will run on every `closed` pull requests.

This workflow exists because the pull request workflow is responsible for cleaning up the preview environment that was created when the pull request was raised. This workflow will destroy the ephemeral environment that was created when the pull request was raised. The environment name will be `preview{{PR_NUMBER}}` i.e. preview52.

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

### Create release
[create-release.yml](.github/workflows/release.create.yml)

This workflow is manually triggered and is responsible for create a release branch from the changes in develop.

It will:
1. Checkout the code
2. Create a release branch from the develop branch. The release branch will be named `release/vx.x` where `x.x` is the version number. The version number is generated from the version configuration [version.json](version.json) - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/version/create-release/action.yml) for more information

### Release
[release.yml](.github/workflows/release.yml)

This workflow is manually triggered and is responsible for building and release the code in the specified branch. This workflow will deploy the assets to the production environment. It should only be executed on `release/vx.x` branches.

It will:
1. build
   1. Checkout the code
   2. Generate a version number based on the version configuration [version.json](version.json) - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/version/generate/action.yml) for more information
   3. Build the Hugo site and create an artifact on the workflow run - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/hugo/build/action.yml) for more information
   4. Send a slack message to the slack github channel - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/slack/built/action.yml) for more information
2. Production
   1. Pull the website artifact and deploy the site to the production environment in and S3 bucket
   2. Send a slack message to the slack github channel - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/slack/released/action.yml) for more information

Variable requirements:
* `SLACK_WEBHOOK_URL`
