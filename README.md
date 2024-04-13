# Gavan Lamb Website
![workflow](https://github.com/gavanlamb/gavanlamb.github.io/actions/workflows/release.yml/badge.svg)  
![Version](https://gavanlamb.com/badges/version.svg)  
[🌎](https://gavanlamb.com/index.html)  

## workflows
### Pull request raised
[pull-request.opened.yml](.github/workflows/pull-request.opened.yml)  

This workflow will run on every `opened` and `reopened` pull requests.

This workflow exists because reviewing pull requests adds an overhead to the review process because information is not readily available. This workflow aims to provide the reviewer with the necessary information to make an informed decision as well as help the developer automate work that is repetitive and time consuming.

It will:
1. Try and get the Jira issue details from the branch name - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/jira/get-issue-details/action.yml) for more information
2. Update the Jira issue status to `Review` if the task is of type `subtask` - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/jira/change-issue-status/action.yml) for more information
3. Update the pull request title and the description with the following variables: - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/pull-request/update-pr/action.yml) for more information
   
   | Variable               | Description                                                                                                                                                                                                                                        |
   |------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
   | `{{ISSUE_IDENTIFIER}}` | The Jira issue identifier. This value is retrieved from the `issueIdentifier` input parameter.                                                                                                                                                     |
   | `{{ISSUE_TITLE}}`      | The Jira issue title. This value is retrieved from the `issueTitle` input parameter.                                                                                                                                                               |
   | `{{ISSUE_URL}}`        | The Jira issue url. This value is retrieved from the `issueUrl` input parameter.                                                                                                                                                                   |
   | `{{REPOSITORY_NAME}}`  | Name of the github repository. This value is retrieved from the context property `${{ github.repository }}`.                                                                                                                                       |
   | `{{ENVIRONMENT}}`      | Name of the preview environment the site is deployed to. This value is retrieved from the `environment` input parameter. Each pull request has a unique environment. The environment name has a template of `preview{{PR_NUMBER}}` i.e. preview52. |
   | `{{BRANCH_NAME}}`      | Name of the branch the PR has been raised for. This value is retrieved from the context property `${{ github.head_ref }}`.                                                                                                                         |

4. Send a slack message to the slack github channel - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/slack/pull-request-created/action.yml) for more information

GitHub secrets requirements:
* `JIRA_API_TOKEN`
* `SLACK_WEBHOOK_URL`

### Pull request pushed
[pull-request.pushed.yml](.github/workflows/pull-request.pushed.yml)  

This workflow will run with every push to any branch that currently has an active pull request that doesn't target main, yes even the release branch.

This workflow exists because the pull request workflow is responsible for building and deploying the assets to a preview environment. This environment is used to preview the changes before merging the pull request. This workflow will deploy items to an ephemeral environment that will be destroyed once the pull request is closed. The environment name will be `preview{{PR_NUMBER}}` i.e. preview52.

It will:
1. build
   1. Checkout the code
   2. Generate a version number - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/version/generate/action.yml) for more information
   3. Build the Hugo site and create an artifact on the workflow run - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/hugo/build/action.yml) for more information
   4. Send a slack message to the slack github channel - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/slack/built/action.yml) for more information
2. Preview
   1. Deploy the site to a preview environment in and S3 bucket - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/hugo/deploy-to-s3/action.yml) for more information
   2. Send a slack message to the slack github channel - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/slack/released/action.yml) for more information

GitHub secrets requirements:
* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`
* `JIRA_API_TOKEN`
* `SLACK_WEBHOOK_URL`

GitHub variable requirements:
* `AWS_S3_CICD_BUCKET`
* `AWS_DEFAULT_REGION`

### Pull request closed
[pull-request.closed.yml](.github/workflows/pull-request.closed.yml)  

This workflow will run on every `closed` pull requests.

This workflow exists because the pull request workflow is responsible for cleaning up the preview environment that was created when the pull request was raised. This workflow will destroy the ephemeral environment that was created when the pull request was raised. The environment name will be `preview{{PR_NUMBER}}` i.e. preview52.

It will:
1. Try and get the Jira issue details from the branch name - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/jira/get-issue-details/action.yml) for more information
2. Update the Jira issue status to `Done` if the task is of type `subtask` - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/jira/change-issue-status/action.yml) for more information
3. Remove the site from the S3 bucket
4. Delete GitHub environment created for the pull request

GitHub secrets requirements:
* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`
* `JIRA_API_TOKEN`
* `SLACK_WEBHOOK_URL`

GitHub variable requirements:
* `AWS_S3_CICD_BUCKET`
* `AWS_DEFAULT_REGION`

### Release create
[release.create.yml](.github/workflows/release.create.yml)

This workflow is manually triggered and is responsible for creating a release or hotfix branch. If the source branch is develop it will create a release branch, if the branch is main then it will create a hotfix branch.

It will:
1. Checkout the code
2. Create the branch name
3. Create the branch if it doesn't exist
4. If the created branch is `release` then it will create a pull request from the main branch to the `release` branch

### Release
[release.yml](.github/workflows/release.yml)

This workflow is manually triggered and is responsible for building and release the code in the specified branch. This workflow will deploy the assets to the production environment. It should only be executed on `release/vx.x` branches.

It will:
1. build
   1. Checkout the code
   2. Generate a version number - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/version/generate/action.yml) for more information
   3. Build the Hugo site and create an artifact on the workflow run - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/hugo/build/action.yml) for more information
   4. Send a slack message to the slack github channel - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/slack/built/action.yml) for more information
2. Create PRs
   1. Checkout the code
   2. Create a pull request from the `release` branch to the `develop` branch
   3. Create a pull request from the `release` branch to the `main` branch
3. Production
   1. Approve deployment to production 
   2. Pull the website artifact and deploy the site to the production environment in and S3 bucket
   3. Send a slack message to the slack github channel - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/slack/released/action.yml) for more information
4. Close the release
   1. Checkout the code
   2. Create a tag for the release
   3. Merge the pull request from the `release` branch to the `develop` branch
   4. Merge the pull request from the `release` branch to the `main` branch
   5. Create a GitHub release

GitHub secrets requirements:
* `SLACK_WEBHOOK_URL`
