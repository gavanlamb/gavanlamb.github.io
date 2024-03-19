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
2. Try and get the jira issue details from the branch name
3. Update the Jira issue status to `Review`
4. Update the pull request title and the description with the following variables:
   * `{{ISSUE_IDENTIFIER}}`: the Jira issue identifier. This value is retrieved from the `issueIdentifier` input parameter.
   * `{{ISSUE_TITLE}}`: the Jira issue title. This value is retrieved from the `issueTitle` input parameter.
   * `{{ISSUE_URL}}`: the Jira issue url. This value is retrieved from the `issueUrl` input parameter.
   * `{{REPOSITORY_NAME}}`: name of the github repository. This value is retrieved from the context property `${{ github.repository }}`.
   * `{{ENVIRONMENT}}`: name of the preview environment the site is deployed to. This value is retrieved from the `environment` input parameter. Each pull request has a unique environment. The environment name has a template of `preview{{PR_NUMBER}}` i.e. preview52.
   * `{{BRANCH_NAME}}`: name of the branch the PR has been raised for. This value is retrieved from the context property `${{ github.head_ref }}`.
5. Send a slack message to the slack github channel

Variable requirements:
* `SLACK_WEBHOOK_URL`
* `JIRA_API_TOKEN`

### Preview
[preview.yml](.github/workflows/preview.yml)

This pipeline will build and deploy the assets to an S3 bucket for previewing changes. The pipeline will run on every push to an active pull request.

The pull request description will include 
* A link to the preview website 
* A version badge
* A pipeline badge for the latest run

This pipeline is executed on every push because....

### Create Release
[create-release.yml](.github/workflows/create-release.yml)

### Release
[release.yml](.github/workflows/release.yml)
