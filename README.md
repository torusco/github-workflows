# github-workflows

reusable github workflows for our organization

# branches are different here

this repository is different due to the nature of it needing to work with other repos as a reference

branches are named: v6, v7.1, v8, etc and they are not deleted when PRs are merged

PRs are only created when making a new major version to push the latest version to main before creating the new branch

thank you!

# usage example

## cdk-diff

### v8

* based on v7.3
* switch to github oidc provider (remove SAML and saml.to)

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