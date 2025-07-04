# Github Composite Action

Global Repo for composite actions: `tf-shared-actions`

```
tf-shared-actions/
└── .github/
    └── actions/
        └── terraform-init/
        |        └── action.yaml
        └── terraform-plan/
        |        └── action.yaml
        └── terraform-apply/
        |        └── action.yaml
        └── terraform-destroy/
        |        └── action.yaml
        └── s3-deploy/
                 └── action.yaml
```

## Available Actions:

**terraform-init**

Single Step action
- sets up init with S3 backend

Required inputs:
- bucket
- key
- region

------------------------------------------------------------------------------------------

**terraform-plan**

Multi-step action
- aws-configure
- terraform fmt, init with s3 backend, validate, plan
- post PR comment

Optional inputs:
- create_oidc_role
- oidc_role
- var file
- tf version
- make pr comment (boolean)
- github token (for PR comment)

------------------------------------------------------------------------------------------

**terraform-apply**

Multi-step Action
- aws-configure
- terraform init, validate, apply
- post Commit comment

Same inputs as terraform-plan

------------------------------------------------------------------------------------------

**terraform-destroy**

Multi-step Action
- aws-configure
- terraform init, destroy
- post Commit comment

Same inputs as terraform-plan and terraform-apply

------------------------------------------------------------------------------------------

**s3-deploy**

Multi-step Action
- setup-node
- npm install && npm run build
- aws-configure
- aws s3 sync
- aws cloudfront create-invalidation

Optional inputs:
- create_oidc_role
- oidc_role
- build directory
- build command
- distribution id

Example of getting distribution id for cloudfront (assuming outputs.tf contains cloudfront_distribution_id):
```yaml
- name: Terraform Apply
  uses: mleager/tf-shared-actions/.github/actions/terraform-apply.yaml@main
    ...

- name: Get Distribution ID for Cloudfront (Optional)
  id: tf-outputs
  run: |
    DIST_ID=$(terraform output -raw cloudfront_distribution_id)
    echo "dist_id=$DIST_ID" >> $GITHUB_OUTPUT
    ...

- name: Deploy S3
  uses: mleager/tf-shared-action/.github/actions/s3-deploy.yaml@main
    ...
```

------------------------------------------------------------------------------------------

Pros:

- easily reusable if all terraform repos follw the same requirements

- workflows will be shorter and more concise

Cons:

- cannot add additional steps between the setup and plan

- not modular

------------------------------------------------------------------------------------------

## Using OIDC

This is present as a comment in each COmposite Action for reference.

These Composite Actions handle Terraform operations with AWS OIDC authentication

OIDC Role Behavior:  
- When create_oidc_role is 'true' (default): Uses a constructed role name in the format  
  'oidc-{project_name}-{environment}-plan'
- When create_oidc_role is 'false' and oidc_role is provided: Uses the specified oidc_role
- When create_oidc_role is 'false' but no oidc_role is provided: Falls back to the constructed  
  role name format as a default

The action performs the following steps:
1. Sets up AWS credentials using OIDC authentication
2. Runs terraform init with S3 backend configuration
3. Validates and plans the Terraform configuration
4. Optionally posts the plan as a PR comment#

Example Usage:  
- name: Terraform Plan  
  uses: mleager/tf-shared-actions/.github/actions/terraform-plan@main  
  with:  
    create_oidc_role: true  
    oidc_role: "github-actions-oidc-role"  
    aws_account_id: ${{ secrets.AWS_ACCOUNT_ID }}  
    project_name: "my-project"  
    environment: "plan"  
    s3_bucket: "my-terraform-state-bucket"  
    key: "tf-state/terraform.tfstate"  
    region: "us-west-2"  

