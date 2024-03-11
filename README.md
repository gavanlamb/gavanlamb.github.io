# Gavan Lamb Website
![Pipeline](https://github.com/gavanlamb/gavanlamb.github.io/actions/workflows/preview.yml/badge.svg?event=pull_request&branch=main)
![Version](https://gavanlamb-github-actions-assets.s3.ap-southeast-2.amazonaws.com/gavanlamb/gavanlamb.github.io/main/site/version.svg)
[🌎](https://gavanlamb.github.io/index.html)

## Pipelines
### Update Pull Request Description
[update-pull-request-description.yml](.github/workflows/update-pull-request-description.yml)

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
