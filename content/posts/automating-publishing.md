---
date: 2024-05-10T11:22:59+01:00
title: Automating Publishing
---

I set some quick automation for publishing the site. I'm storing things in a GitHub Repo so I went ahead and setup a GitHub Workflow to build and publish the site.

To get things rolling I first had to setup an AWS IAM Identity Provider and an AWS IAM Role. I'm still managing all of this via the AWS Console and the creation wizards are much better then they were. I was able to step through the process and only need to make one manual change. I updated the IAM Role's Trust Policy to enable Session Tagging, the policy created by the wizard did not have the `sts:TagSession` action included. The walkthrough in the GitHub Docs is a great place to start: [Configuring OpenID Connect in Amazon Web Services](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services#adding-the-identity-provider-to-aws).

**Updated IAM Role Trust Policy**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "<iam_identity_provider_arn>"
      },
      "Action": ["sts:AssumeRoleWithWebIdentity", "sts:TagSession"],
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:<repo_owner>/<repo_name>:*"
        }
      }
    }
  ]
}
```

I created GitHub Repository Secrets for:

- The ARN of the IAM Role created via the AWS Console Wizard
- The S3 Bucket Name
- The Cloudfront Distribution ID

From there I setup a GitHub Workflow triggered by a push to my repo's `main` branch.

**GitHub Workflow**

```yaml
name: Publish Site
on:
  push:
    branches:
      - main
  workflow_dispatch:
jobs:
  build_publish:
    name: Build and Publish
    permissions:
      id-token: write
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Hugo
        shell: pwsh
        run: |
          $GithubRequestHeaders = @{
              Accept                 = "application/vnd.github.v3+json"
              "X-GitHub-Api-Version" = "2022-11-28"
          }

          Invoke-WebRequest -Uri "https://api.github.com/repos/gohugoio/hugo/releases/latest" -Headers $GithubRequestHeaders | ConvertFrom-Json `
          | Select-Object -ExpandProperty assets `
          | Where-Object { $_.name -like "hugo_extended_*_linux-amd64.tar*" } `
          | Select-Object -ExpandProperty browser_download_url | ForEach-Object {
              $DownloadPath = Join-Path -Path "./" -ChildPath $(Split-Path -Leaf $_)
              $output = "$(Get-Location)/$(Split-Path -Leaf $_)"
              $tarFile = (Split-Path -Leaf $_).Replace(".gz", "")
              
              Invoke-WebRequest -Uri $_ -OutFile $DownloadPath

              tar -xzf $DownloadPath -C $(Get-Location)

              New-Item -ItemType Directory -Path "$Env:RUNNER_TEMP/hugo_bin" -Force
              Move-Item -Path "./hugo" -Destination "$Env:RUNNER_TEMP/hugo_bin/hugo"
          }

          "$Env:RUNNER_TEMP/hugo_bin" | Out-File -FilePath $env:GITHUB_PATH -Append
      - name: Build Site
        shell: pwsh
        run: |
          npm install
          hugo --cleanDestinationDir --environment production --verbose
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-region: us-east-2
          role-to-assume: ${{ secrets.AWS_PUBLISH_SITE_ROLE }}
      - name: Publish Site
        shell: pwsh
        run: |
          cd public

          aws s3 sync ./ s3://${{ secrets.SITE_S3_BUCKET }}/${{ github.run_id }}/ --delete

          aws s3 sync ./ s3://${{ secrets.SITE_S3_BUCKET }}/live/ --delete

          aws cloudfront create-invalidation --distribution-id ${{ secrets.SITE_CLOUDFRONT_DISTRO }} --paths "/*"
```

Some next steps are to:

- Selectively push only new/updated files to the `live` prefix
- Create a `setup-hugo` GitHub Action
- Add Cache Control Headers as part of the upload to S3 to more easily manage cache settings
- Setup the ability to view versions of the site not in `live` based on subdomain or path or something
