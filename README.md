# Gavan Lamb Website
![workflow](https://github.com/gavanlamb/gavanlamb.github.io/actions/workflows/release.yml/badge.svg)  
![Version](https://gavanlamb.com/badges/version.svg)  
[🌎](https://gavanlamb.com/index.html)  

## workflows
### Notify when pull request has been opened or reopened
[notify-pull-request-opened.yml](.github/workflows/notify-pull-request-opened.yml)  

This workflow will run on every `opened` and `reopened` pull request.

This workflow exists  

It will:
1. Send a message to a Google Chat Space with details of the PR

GitHub secrets requirements:
* `GOOGLE_CHAT_WEBHOOK_URL`

### Pull request pushed
[pull-request.pushed.yml](.github/workflows/pull-request.pushed.yml)  

This workflow will run with every push to a branch that currently has an active pull request.

This workflow exists because the pull request workflow is responsible for building and deploying the assets to a preview environment for every push. This environment is used to preview the changes before merging the pull request to main. This workflow will deploy items to an ephemeral environment that will be destroyed once the pull request is closed. The environment name will be `preview{{PR_NUMBER}}` i.e. preview52.

It will:
1. build
   1. Checkout the code
   2. Install gitversion
   3. Generate versiopn number
   4. Build the Hugo site and create an artifact on the workflow run - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/hugo/build/action.yml) for more information
   5. Archive the site - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/archive/action.yml) for more information
2. Preview
   1. Deploy the site to a preview environment in and S3 bucket - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/hugo/deploy-to-s3/action.yml) for more information

GitHub secrets requirements:
* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`

GitHub variable requirements:
* `AWS_S3_CICD_BUCKET`
* `AWS_DEFAULT_REGION`

### Remove environment when pull request is closed
[remove-environment.yml](.github/workflows/remove-environment.yml)  

This workflow will run on every `closed` pull request.

This workflow exists because the pull request workflow is responsible for cleaning up the preview environment that was created when the pull request was raised. The environment name will be `preview{{PR_NUMBER}}` i.e. preview52.

It will:
1. Remove the site from the S3 bucket
2. Delete GitHub environment created for the pull request

GitHub secrets requirements:
* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`
* `PAT_GITHUB_ACTIONS`

GitHub variable requirements:
* `AWS_S3_CICD_BUCKET`
* `AWS_DEFAULT_REGION`

### Release
[release.yml](.github/workflows/release.yml)

This workflow is manually triggered and is responsible for building and releasing the code when changes are pushed to main.

It will:
1. build
   1. Checkout the code
   2. Install gitversion
   3. Generate versiopn number
   4. Build the Hugo site and create an artifact on the workflow run - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/hugo/build/action.yml) for more information
   5. Archive the site - please see [action.yml](https://github.com/gavanlamb/github-actions/blob/main/.github/actions/archive/action.yml) for more information
2. Production
   1. Pull the website artifact and deploy the site to the production environment in and S3 bucket
