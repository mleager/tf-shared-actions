# Github Composite Action

Create a new Repo: `tf-shared-actions`

```
tf-shared-actions/
└── .github/
    └── actions/
        └── terraform-plan/
            └── action.yaml
```

## What it Does

1. Setup Terraform action

2. Configure AWS action

3. Format

4. Init with configured S3 Backend

5. Validate

6. Plan

Example workflow using the action:

```terraform
jobs:
  terraform:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./terraform
    steps:
      - name: Terraform Plan Setup
        uses: mleager/tf-shared-actions/.github/actions/terraform-plan@main
        with:
          bucket: tf-state-8864
          key: tf-s3-global-state/terraform.tfstate
          region: ${{ vars.AWS_REGION }}
          var_file: terraform.tfvars.development

      - name: Terraform Apply
        if: github.ref == 'refs/heads/main'
        run: terraform apply -var-file=terraform.tfvars.development -auto-approve
```

Pros:

- easily reusable if all terraform repos follw the same requirements

- workflows will be shorter and more concise

Cons:

- cannot add additional steps between the setup and plan

- not modular



## Potential Issues

This composite action may being doing too much.

Perhaps its better if the composite action only performs the "terraform init -backend-config=..."  
for the S3 Backend setup.


**Potential Solution**

Create another composite action that only has the "terraform init" portion

Example:
```terraform
jobs:
  terraform:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./terraform
    steps:
    - uses: actions/checkout@v4

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
      with:
        terraform_version: ${{ inputs.tf_version }}

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ inputs.region }}

    - name: Terraform Format Check
      run: terraform fmt -check

    - name: Terraform Validate
      run: terraform validate

    - name: Terraform Init
      uses: mleager/tf-shared-actions/.github/actions/terraform-plan@main
      with:
        bucket: tf-state-8864
        key: tf-s3-global-state/terraform.tfstate
        region: ${{ vars.AWS_REGION }}
        var_file: terraform.tfvars.development

    - name: Terraform Plan
      run: terraform plan -var-file=${{ inputs.var_file }}

      - name: Terraform Apply
        if: github.ref == 'refs/heads/main'
        run: terraform apply -var-file=terraform.tfvars.development -auto-approve
```

