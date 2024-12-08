# Multi-step deployment with Review

## Goal

In this sample repository we want to implement a multi-step deployment to SAP BTP namely a subaccount including a Cloud Foudnry environment (step 1) and a Cloud Foundry space (step 2) via Terraform.

The multi step approach is necessary as we need the Cloud Foundry API from step 1 to configure the Terraform provider for Cloud Foundry in step 2. As the provider configuration does not allow a dynamic setup we split the setup in the mentioned two steps. This anyway makes sense concerning the split of state files and reducing the blast radius of changes

In addition, we want to mimic a request flow from a development team for a new subaccount. This request is filed via a GitHub issue and needs an approval.

When the request is fulfilled, the issue should be updated with the relevant URLs for the subaccount, the CF space and the endpoint of the CF API.

> [!NOTE]
> We do not use a state backend in this sample which would be the default in productive setups. As the focus lies on the integration of Terraform in a request we skip this.

## Setup

The setup consists of four parts:

- definition of a [GitHub environment](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-deployments/managing-environments-for-deployment) for the deployment to have a review or sign-off in place
- a GitHub issue that uses [issue forms](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-issue-forms) to capture the necessary input data
- a GiHub Action workflow that executes the provisioning
- a Terraform configuration of the setups that is located in the `infra` directory. There are two subdirectories namely `BTP` for the BTP specific setup and `CloudFoundry` for the Cloud Foundry part. The necessary input is defined in the corresponding `variables.tf` files. The output needed for the consequent steps or the issue

The issue template is defined as [`account_request.yml`](.github/ISSUE_TEMPLATE/account_request.yml) and collects the input needed for the setup.

The GitHub action workflow is defined as [`project-provisioning.yml`](.github/workflows/project-provisioning.yml). It is treggered whenever an issue with a specific content is created and contains the following flow:

- Read the information from the issue using the [issue-ops/parser](https://github.com/marketplace/actions/issueops-form-parser) Action. This transfers the information from the issue into JSON based on the issue temlplate
- Transfer the information form the issue to `GITHUB_OUTPUT` to consistently share it between the steps
- Setup and execute the setup of the subaccount via Terraform. We use the `TF_VAR_*` option to fill the variables with the information from the issue
- We extract the output of the Terraform run into `GITHUB_OUTPUT` to use it later
- Setup and execute the setup of the subaccount via Terraform. We use the `TF_VAR_*` option to fill the variables with the information from the issue and the previous setup of the BTP environment
- Add a comment to the triggering issue with the information from the Terraform setup

## Alternative

Instead of using GitHub Environments you can also add a GitHub Action [trstringer/manual-approval](https://github.com/marketplace/actions/manual-workflow-approval) which pauses the workflow until an issue is approved or declined.

If you want to use this variant you must make sure that all secrets are repository secrets and the environment specific setting in the workflow are removed.
