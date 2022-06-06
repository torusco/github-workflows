# github-workflows

reusable github workflows for our organization

# branches are different here

this repository is different due to the nature of it needing to work with other repos as a reference

branches are named: v1, v2, v3, v4, etc and they are not deleted when PRs are merged

these version branches do need to be merged to main before the next version is created

thank you!

# usage example

## cdk-diff

### v1

```
  integration-ai1:
    needs: [ globals ]
    uses: torusinc/github-workflows/.github/workflows/cdk-diff.yaml@v1
    with:
      AWS_REGION: ${{ needs.globals.outputs.AWS_REGION }}
      CDK_FOLDER_NAME: cdk-account
      CDK_PREFIX: ai1
      ENVIRONMENT_LONG_NAME: "integration"
      SAML_AWS_ROLE_ARN: ${{ needs.globals.outputs.SAML_AWS_ROLE_ARN }}
      TARGET_AWS_ACCOUNT_ROLE_ARN: "arn:aws:iam::206272298119:role/gha-app-deployer-1"
```
## cdk-test

### v1

```
  yarn-test:
    needs: globals
    uses: torusinc/github-workflows/.github/workflows/cdk-test.yaml@v1
    with:
      AWS_REGION: ${{ needs.globals.outputs.AWS_REGION }}
      CDK_FOLDER_NAME: cdk-account
      SAML_AWS_ROLE_ARN: ${{ needs.globals.outputs.SAML_AWS_ROLE_ARN }}
```
