# oak-release-actions

It houses a collection of reusable GitHub Actions for Terraform‑controlled applications. Any repository can call these actions to:
- Preview proposed infrastructure changes on `PRs`
- Deploy automatically to environments on new semantic version tags
- Promote a specific release to `production` by inputting a `tag_id`

#### Structure

```plaintext
.github/
└── actions/
    ├── terraform_deploy
    │   └── action.yml      # Deployment: on 'v*' tags or Production Promotion: manual workflow_dispatch
    └── terraform_pr_plan
        └── action.yml      # Plan Phase: Terraform plan on PR
```

#### Key Features

- **Plan Phase (Pull Request)**
 Runs terraform plan in Terraform Cloud and comments the plan link on the PR for easy review.
- **Terraform Deploy**
 Handles both automatic deployments to environments like beta on semantic tag pushes (v*) and manual promotions to production using a workflow_dispatch with a tag_id.

### How to Use

1. **Plan Phase (Pull Request)**
In your repo’s .github/workflows/pr_plan.yml:
```yaml
name: Terraform Plan Preview
on: pull_request

permissions:
  contents: read
  pull-requests: write

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Run Terraform Plan from oak-release-actions
        uses: oaknational/oak-release-actions/actions/terraform_pr_plan@main
        with:
            TF_API_TOKEN: ${{ secrets.TF_API_TOKEN }}
            TF_BASE_SUBDIRECTORY: "folder-that-contains-your-terraform-files"
            TF_CLOUD_ORGANIZATION: "your-terraform-cloud-organization"
            TF_WORKSPACE: "your-workspace"
            PR_COMMENT_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

2. **Tag-Based Deployment(e.g beta)**
In your repo’s .github/workflows/beta_deploy.yml:

```yaml
name: Terraform Deploy to Beta
on:
  push:
    tags:
      - 'v*'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Run Terraform Apply from oak-release-actions
        uses: oaknational/oak-release-actions/actions/terraform_deploy@main
        with:
            TAG_ID: ${{ github.ref_name }}
            TF_API_TOKEN: ${{ secrets.TF_API_TOKEN }}
            TF_BASE_SUBDIRECTORY: "folder-that-contains-your-terraform-files"
            TF_CLOUD_ORGANIZATION: "your-terraform-cloud-organization"
            TF_WORKSPACE: "your-workspace"
```

3. **Manual Production Promotion**
In your repo’s .github/workflows/prod-promote.yml:

```yaml
name: Promote Terraform to Production
on:
  workflow_dispatch:
    inputs:
      tag_id:
        description: 'Git tag to apply to production'
        required: true
        type: string

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          fetch-tags: true
          ref: ${{ github.event.inputs.tag_id }}

      - name: Run Terraform Deploy from oak-release-actions
        uses: oaknational/oak-release-actions/actions/terraform_deploy@main
        with:
            TAG_ID: ${{  inputs.tag_id }}
            TF_API_TOKEN: ${{ secrets.TF_API_TOKEN }}
            TF_BASE_SUBDIRECTORY: "folder-that-contains-your-terraform-files"
            TF_CLOUD_ORGANIZATION: "your-terraform-cloud-organization"
            TF_WORKSPACE: "your-workspace"
```
