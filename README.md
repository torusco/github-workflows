# github-workflows

reusable github workflows for our organization

# branches are different here

this repository is different due to the nature of it needing to work with other repos as a reference

branches are named: v1, v2, v3, v4, etc and they are not deleted when PRs are merged

these version branches do need to be merged to main before the next version is created

thank you!

# usage example

## cdk-diff

### v1-v4

Deprecated.  Please upgrade.

### v5

Stable, up to date.

```
  integration1:
    needs: [ globals ]
    uses: torusco/github-workflows/.github/workflows/cdk-diff.yaml@v5
    with:
      AWS_REGION: ${{ needs.globals.outputs.AWS_REGION }}
      CDK_FOLDER_NAME: cdk-account
      CDK_PREFIX: ai1
      ENVIRONMENT_LONG_NAME: "integration"
      SAML_AWS_ROLE_ARN: ${{ needs.globals.outputs.SAML_AWS_ROLE_ARN }}
      TARGET_AWS_ACCOUNT_ROLE_ARN: "arn:aws:iam::${{ needs.globals.outputs.ACCOUNT_NUMBER }}:role/gha-app-deployer-1"
```
## cdk-diff

### v6

Github Large Runners Experiment (hard-coded)

```
  integration1:
    needs: [ globals ]
    uses: torusco/github-workflows/.github/workflows/cdk-diff.yaml@v5
    with:
      AWS_REGION: ${{ needs.globals.outputs.AWS_REGION }}
      CDK_FOLDER_NAME: cdk-account
      CDK_PREFIX: ai1
      ENVIRONMENT_LONG_NAME: "integration"
      SAML_AWS_ROLE_ARN: ${{ needs.globals.outputs.SAML_AWS_ROLE_ARN }}
      TARGET_AWS_ACCOUNT_ROLE_ARN: "arn:aws:iam::${{ needs.globals.outputs.ACCOUNT_NUMBER }}:role/gha-app-deployer-1"
```

## cdk-diff

### v7

* Simpler monorepo support for multiple cdk / tf 
* Pass yarn commands for a BC rather than assuming homogenous repos with same setups
* Integration Tests have their own workflow because it doesn't always make sense to run them after the cdk deploy, sometimes, more deploys are needed first - use yarn-test
* Filter by cdk path as well as tf path - only run when changes are present
* Environment gate is a separate workflow so it is only inserted once
* Runs on (runner type) is passed via input (update to v6)

env-gate example:
```
  testqa_env_gate:
    needs: [ globals ]
    uses: torusco/github-workflows/.github/workflows/env-gate.yaml@v7
    with:
      CDK_PREFIX: "dt1"
      RUNS_ON: ${{ needs.globals.outputs.RUNS_ON }}

```

yarn-test unit example:
```
  yarn_test_cdks:

    needs: [ globals ]
    uses: torusco/github-workflows/.github/workflows/yarn-test.yaml@v7
    with:
      AWS_REGION: ${{ needs.globals.outputs.AWS_REGION }}
      RUNS_ON: ${{ needs.globals.outputs.RUNS_ON }}
      SAML_AWS_ROLE_ARN: ${{ needs.globals.outputs.SAML_AWS_ROLE_ARN }}
      YARN_TEST_COMMAND: "yarn pipeline:test"
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```
