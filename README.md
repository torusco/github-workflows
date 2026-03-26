# github-workflows

!! Public Repository - reusable github workflows for our organization

# branches are different here

* this repository is different due to the nature of it needing to work with other repos as a reference by branch
* branches are named: v6, v7.1, v8, etc and they are not deleted when PRs are merged
* PRs are only created when making a new major version to push the latest version to main before creating the new branch

# v8.1

An argument compatible Node 24 update.

* based on v8
* update github actions to newer versions
* node 24
* terraform TBD
* python 3.12
* Slack API to v3

## steps to migrate to this version

1. Look for any local github workflows and update those first
2. Switch to v8.1

# Examples

## configure aws credentials

```
  job_name:

    runs-on: ${{ inputs.RUNS_ON }}

    permissions: # github oidc
      id-token: write
      contents: read

    steps:

```

```
  - name: Deployer Role with Github OIDC Provider
    uses: aws-actions/configure-aws-credentials@8df5847569e6427dd6c4fb1cf565c83acfa8afa7 # v6.0.0 Feb2026
    with:
        role-to-assume: ${{inputs.TARGET_AWS_ACCOUNT_ROLE_ARN}}
        aws-region: ${{inputs.AWS_REGION}}

```

## airflow-waiter

```
  check_airflow_is_ready:
    if: ${{ !contains( github.event.pull_request.labels.*.name, 'data_skip_airflow') }}
    needs: [ globals, ... ]
    uses: torusco/github-workflows/.github/workflows/airflow-waiter.yaml@v8
    with:
      AIRFLOW_ENVIRONMENT_NAME: ${{ needs.globals.outputs.AIRFLOW_ENVIRONMENT_NAME }}
      AWS_REGION: ${{ needs.globals.outputs.AWS_REGION }}
      CDK_PREFIX: ${{ needs.globals.outputs.CDK_PREFIX }}
      ENVIRONMENT_LONG_NAME: ${{ needs.globals.outputs.ENVIRONMENT_LONG_NAME }}
      SLACK_CHANNEL_ID: ${{ needs.globals.outputs.SLACK_CHANNEL_ID }}
      TARGET_AWS_ACCOUNT_ROLE_ARN: ${{ needs.globals.outputs.TARGET_AWS_ACCOUNT_ROLE_ARN }}
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }} 
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

```

## cdk-deploy

```
  cdk_example:
    needs: [globals]
    uses: torusco/github-workflows/.github/workflows/cdk-deploy.yaml@v8
    with:
      CDK_FOLDER_NAME: 'cdk-folder-name'
      YARN_DEPLOY_COMMAND: 'yarn pipeline-deploy-folder-name'
      AWS_REGION: ${{ needs.globals.outputs.AWS_REGION }}
      CDK_PREFIX: ${{ needs.globals.outputs.CDK_PREFIX }}
      ENVIRONMENT_LONG_NAME: ${{ needs.globals.outputs.ENVIRONMENT_LONG_NAME }}
      SLACK_CHANNEL_ID: ${{ needs.globals.outputs.SLACK_CHANNEL_ID }}
      TARGET_AWS_ACCOUNT_ROLE_ARN: ${{ needs.globals.outputs.TARGET_AWS_ACCOUNT_ROLE_ARN }}
    secrets:
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }} 
```

## cdk-diff

```
  long_short:

    needs: [ globals ]
    uses: torusco/github-workflows/.github/workflows/cdk-diff.yaml@v8
    with:
      AWS_REGION: ${{ needs.globals.outputs.AWS_REGION }}
      CDK_FOLDER_NAME: cdk-name
      CDK_PREFIX: shortname
      ENVIRONMENT_LONG_NAME: "longname"
      TARGET_AWS_ACCOUNT_ROLE_ARN: "arn:aws:iam::ACCOUNT:role/ROLE"
      YARN_DIFF_COMMAND: "yarn pipeline:diff"
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## cdk-test

note: renamed from yarn-test

```
  update_sonar_main_analysis:

    if: ${{ !contains( github.event.pull_request.labels.*.name, 'nodeploy') || github.event.pull_request.merged == false }}
    uses: torusco/github-workflows/.github/workflows/cdk-test.yaml@v8
    with:
      YARN_TEST_COMMAND: "yarn test"
      USES_SONAR_CLOUD: true
      USES_SONAR_CLOUD_MAIN_ANALYSIS: true
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

```
    yarn_test_cdks:
        needs: [globals]
        uses: torusco/github-workflows/.github/workflows/cdk-test.yaml@v8
        with:
            YARN_TEST_COMMAND: 'yarn pipeline-test'
        secrets:
            SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
            NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## env-gate

```
  gate_check:
    needs: [globals]
    uses: torusco/github-workflows/.github/workflows/env-gate.yaml@v8
    with:
      SLACK_CHANNEL_ID: ${{ needs.globals.outputs.SLACK_CHANNEL_ID }}
      CDK_PREFIX: ${{ needs.globals.outputs.CDK_PREFIX }}
    secrets:
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

```

## python-run

```
  python_metabase_example:
    needs: [ globals, ... ]
    uses: torusco/github-workflows/.github/workflows/python-run.yaml@v8
    with:
      PY_FOLDER_NAME: 'py-metabase-config'
      PY_VERSION: '3.11'
      PY_RUN_COMMAND: 'python ga-configure-metabase-redshift.py'
      AWS_REGION: ${{ needs.globals.outputs.AWS_REGION }}
      CDK_PREFIX: ${{ needs.globals.outputs.CDK_PREFIX }}
      ENVIRONMENT_LONG_NAME: ${{ needs.globals.outputs.ENVIRONMENT_LONG_NAME }}
      SLACK_CHANNEL_ID: ${{ needs.globals.outputs.SLACK_CHANNEL_ID }}
      TARGET_AWS_ACCOUNT_ROLE_ARN: ${{ needs.globals.outputs.TARGET_AWS_ACCOUNT_ROLE_ARN }}
      USES_METABASE: true
    secrets:
      CHECKOUT_TOKEN: ${{ secrets.NPM_TOKEN }}            
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
      METABASE_USER: ${{ secrets.METABASE_USER }}
      METABASE_PASS: ${{ secrets.METABASE_PASS }}    
```

## terraform-apply

```
  tf_fivetran_with_python:
    needs: [globals, ... ]
    uses: torusco/github-workflows/.github/workflows/terraform-apply.yaml@v8
    with:
      TARGET_TERRAFORM_FOLDER_NAME: tf-fivetran-name
      TERRAFORM_VAR_FILE: ./${{ needs.globals.outputs.ENVIRONMENT_LONG_NAME }}/${{ needs.globals.outputs.CDK_PREFIX }}.tfvars
      PIP_INSTALL_REQUIREMENTS_FILE: ./tf-fivetran-name/requirements.txt
      PY_VERSION: "3.11"
      AWS_REGION: ${{ needs.globals.outputs.AWS_REGION }}
      CDK_PREFIX: ${{ needs.globals.outputs.CDK_PREFIX }}
      ENVIRONMENT_LONG_NAME: ${{ needs.globals.outputs.ENVIRONMENT_LONG_NAME }}
      SLACK_CHANNEL_ID: ${{ needs.globals.outputs.SLACK_CHANNEL_ID }}
      TARGET_AWS_ACCOUNT_ROLE_ARN: ${{ needs.globals.outputs.TARGET_AWS_ACCOUNT_ROLE_ARN }}
      USES_FIVETRAN: true
    secrets:
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
      FIVETRAN_APIKEY: ${{ secrets.FIVETRAN_APIKEY }}
      FIVETRAN_APISECRET: ${{ secrets.FIVETRAN_APISECRET }}
```

```
  tf_uses_cloudflare:
    needs: [globals, cdk_redshift]
    uses: torusco/github-workflows/.github/workflows/terraform-apply.yaml@v8
    with:
      TARGET_TERRAFORM_FOLDER_NAME: tf-cloudflare-name
      TERRAFORM_VAR_FILE: ./${{ needs.globals.outputs.ENVIRONMENT_LONG_NAME }}/.tfvars
      AWS_REGION: ${{ needs.globals.outputs.AWS_REGION }}
      CDK_PREFIX: ${{ needs.globals.outputs.CDK_PREFIX }}
      ENVIRONMENT_LONG_NAME: ${{ needs.globals.outputs.ENVIRONMENT_LONG_NAME }}
      SLACK_CHANNEL_ID: ${{ needs.globals.outputs.SLACK_CHANNEL_ID }}
      TARGET_AWS_ACCOUNT_ROLE_ARN: ${{ needs.globals.outputs.TARGET_AWS_ACCOUNT_ROLE_ARN }}
      USES_CLOUDFLARE: true
    secrets:
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
      CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

## terraform-checkov

```
    terraform_checkov_name:
        needs: [globals]
        uses: torusco/github-workflows/.github/workflows/terraform-checkov.yaml@v8
        with:
            TARGET_TERRAFORM_FOLDER_NAME: tf-name
            CDK_PREFIX: ${{ needs.globals.outputs.CDK_PREFIX }}
            ENVIRONMENT_LONG_NAME: ${{ needs.globals.outputs.ENVIRONMENT_LONG_NAME }}
        secrets:
            NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## terraform-pull-request

```
  tf_pull_request:

    needs: globals
    uses: torusco/github-workflows/.github/workflows/terraform-pull-request.yaml@v8
    with:
      AWS_REGION: ${{ needs.globals.outputs.AWS_REGION }}
      CDK_PREFIX: "shortname"
      ENVIRONMENT_LONG_NAME: "longname"
      TARGET_AWS_ACCOUNT_ROLE_ARN: ${{ needs.globals.outputs.TARGET_AWS_ACCOUNT_ROLE_ARN }}
      TARGET_TERRAFORM_FOLDER_NAME: tf-name
      TERRAFORM_VAR_FILE: ./${{ needs.globals.outputs.ENVIRONMENT_LONG_NAME }}/.tfvars
      USES_CLOUDFLARE: true
    secrets:
      CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}

```

