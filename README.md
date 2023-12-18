# github-workflows

!! Public Repository - reusable github workflows for our organization

# branches are different here

* this repository is different due to the nature of it needing to work with other repos as a reference by branch
* branches are named: v6, v7.1, v8, etc and they are not deleted when PRs are merged
* PRs are only created when making a new major version to push the latest version to main before creating the new branch

# v8

* based on v7.3
* switch to github oidc provider (remove SAML and saml.to)
* terraform base version 1.6.6 and now parameterized as input

## steps to migrate to this version

1. Delete saml-to
2. Switch to v8
3. Delete all refs to SAML_AWS_ROLE_ARN including delet the now extra step where SAML was assumed, only aws-actions/configure-aws-credentials is needed now
4. Update any Deployer role steps to configure aws credentials below and add permissions to any job that directly calls configure-aws-credentials
5. Update TARGET_AWS_ACCOUNT_ROLE_ARN to be account based

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
```

## cdk-deploy

```
```

## cdk-diff

```
```

## env-gate

```
```

## mobile-jira

```
```

## python-run

```
```

## slack-message

```
```

## terraform-apply

```
```

## terraform-checkov

```
```

## terraform-pull-request

```
```

## yarn-test

```
  update_sonar_main_analysis:

    if: ${{ !contains( github.event.pull_request.labels.*.name, 'nodeploy') || github.event.pull_request.merged == false }}
    uses: torusco/github-workflows/.github/workflows/yarn-test.yaml@v8
    with:
      YARN_TEST_COMMAND: "yarn test"
      USES_SONAR_CLOUD: true
      USES_SONAR_CLOUD_MAIN_ANALYSIS: true
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```




## old

env-gate example:
```
  testqa_env_gate:
    needs: [ globals ]
    uses: torusco/github-workflows/.github/workflows/env-gate.yaml@v7
    with:
      CDK_PREFIX: "dt1"
      RUNS_ON: ${{ needs.globals.outputs.RUNS_ON }}

```

yarn-test unit example for sonarcloud
```
  update_sonar_main_analysis:

    uses: torusco/github-workflows/.github/workflows/yarn-test.yaml@v8
    with:
      YARN_TEST_COMMAND: yarn pipeline-test
      USES_SONAR_CLOUD: true
      USES_SONAR_CLOUD_MAIN_ANALYSIS: true
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

terraform-checkov example:
```
    terraform_checkov_datadog:
        needs: [globals]
        uses: torusco/github-workflows/.github/workflows/terraform-checkov.yaml@v7.1
        with:
            TARGET_TERRAFORM_FOLDER_NAME: tf-datadog
            CDK_PREFIX: ${{ needs.globals.outputs.CDK_PREFIX }}
            ENVIRONMENT_LONG_NAME: ${{ needs.globals.outputs.ENVIRONMENT_LONG_NAME }}
            RUNS_ON: ${{ needs.globals.outputs.RUNS_ON_TF }}
        secrets:
            NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

```