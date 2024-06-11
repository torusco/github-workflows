# github-workflows

!! Public Repository - reusable github workflows for our organization

# branches are different here

* this repository is different due to the nature of it needing to work with other repos as a reference by branch
* branches are named: v6, v7.1, v8, etc and they are not deleted when PRs are merged
* PRs are only created when making a new major version to push the latest version to main before creating the new branch

# v8

* based on v7.3
* switch to github oidc provider (remove SAML and saml.to)
* TERRAFORM_VERSION terraform default is now 1.6.6 by default, is not required, and can be overridden
* PY_VERSION python default is now 3.11 by default, is not required, and can be overridden
* NODE_VERSION node default is now 20 by default, is not required, and can be overridden
* RUNS_ON for cdk defaults to 8 cores, and latest for all others, is no longer required, and can be overridden
* Support modern version of yarn using corepack

## steps to migrate to this version

1. Delete saml-to
2. Switch to v8
3. Delete all refs to SAML_AWS_ROLE_ARN including delet the now extra step where SAML was assumed, only aws-actions/configure-aws-credentials is needed now
4. Update any Deployer role steps to configure aws credentials below and add permissions to any job that directly calls configure-aws-credentials
5. Update TARGET_AWS_ACCOUNT_ROLE_ARN to be account based
6. Rename yarn-test to cdk-test
7. Remove RUNS_ON unless you want to override
8. Add any overrides for TERRAFORM_VERSION, PY_VERSION, NODE_VERSION but recommendation is to remove and not override

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
    uses: aws-actions/configure-aws-credentials@v4
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

