# <img src="./logo.svg" height="32px" /> Rainforest QA CircleCI Orb ![](https://img.shields.io/github/v/release/rainforestapp/rainforest-orb.svg)

**Registry homepage:** [`rainforest-qa/rainforest`](https://circleci.com/orbs/registry/orb/rainforest-qa/rainforest)

> This is the Rainforest QA [Orb](https://circleci.com/docs/2.0/orb-intro/) for CircleCI, it allows you to easily kick off a Rainforest run from your CircleCI workflows, to make sure that every release passes your Rainforest integration tests.

## Sections
* [Prerequisites](#prerequisites)
* [Base Usage](#base-usage-rainforestrun-job)
* [Optional Parameters](#optional-parameters)
* [Commands](#commands)
* [Executors](#executors)
* [Examples](#examples)
* [Orb Release Process](#orb-release-process)

## Prerequisites

### A Rainforest QA account

If you don't already have one, you can talk to us about setting up an account [here](https://www.rainforestqa.com/talk-to-sales?utm_source=github&utm_medium=readme&utm_campaign=circleci).

### A Rainforest QA API token

You can find yours on the [Integrations](https://app.rainforestqa.com/settings/integrations) setting page.
Do not expose this token in your `circleci/.config.yml` file. Instead, [use an environment variable in CircleCI](https://circleci.com/docs/2.0/env-vars/#setting-an-environment-variable-in-a-project). The default name for this variable is `RAINFOREST_API_TOKEN`. You may use a different name, and pass that in to the `rainforest/run` command (see below).

### A run group with at least one test

Run groups are a way to group tests that should be run together (for example, a smoke suite you might want to run on every deploy). For more information on run groups, see [this help article](https://help.rainforestqa.com/docs/organizing-tests-by-run-group).

Once you have a run group which contains at least one enabled test, you can run it in CircleCI builds using this orb. You will need its ID (visible at the end of the run group URL: `https://app.rainforestqa.com/run_groups/<ID>`).

## Base usage (`rainforest/run` job)

```yaml
# .circleci/config.yml

# To use orbs, your CircleCI config file must use version 2.1
version: 2.1

# If you don't have a top-level `orbs` section, add one
orbs:
# Add the Rainforest orb to that list
  - rainforest: rainforest-qa/rainforest@3

# In your workflows, add it as a job to be run
workflows:
  your_workflow:
    jobs:
      - rainforest/run:
          # Replace the value with the ID of whichever run group you want to run
          run_group_id: "123"
          pipeline_id: << pipeline.id >>
```

## Optional Parameters

### `description`
An arbitrary string to associate with the run.
#### Type
`string`
#### Default behavior
The default value defined in the orb is:
> `"$CIRCLE_PROJECT_REPONAME - $CIRCLE_BRANCH $CIRCLE_BUILD_NUM $(date -u +'%FT%TZ')"`

This means that if no `description` parameter is passed in and your repository is named `my_repo`, the Circle build is `#42` on the `my_feature_branch` branch, and the current time (in UTC) is noon on January 20th, 2021; then the created run's description will be:
> `my_repo - my_feature_branch 42 2021-01-20T12:00:00Z`

### `environment_id`
Use a specific environment for this run. _This parameter will be ignored if the `custom_url` parameter is also passed in._
#### Type
`string`
#### Default behavior
If no `environment_id` parameter is passed in, the created run will use the Run Group's default environment.

### `custom_url`
Use a specific URL (via a [temporary environment](https://github.com/rainforestapp/rainforest-cli#command-line-options)) for this run.
#### Type
`string`
#### Default behavior
If no `custom_url` parameter is passed in, the created run will use the Run Group's default environment.

### `conflict`
How we should handle currently active runs.
#### Type
`enum`
#### Allowed values
Value | Behavior
--- | ---
`cancel` | Cancel all other runs in the same environment.
`cancel-all` | Cancel all other runs, regardless of environment.
#### Default behavior
If no `conflict` parameter is passed in, then no active runs will be canceled.

### `execution_method`
The execution method to use for this run.
#### Type
`enum`
#### Allowed values
Value | Behavior | Requirements
--- | --- | ---
`automation` | Run against our automation agent. | - All tests in the run group are written with the Visual Editor.<br />- No tests use a Tester Instruction/Confirmation action.
`crowd` | Run against our global crowd of testers.
`automation_and_crowd` | Run against our automation agent where possible, fall back to the crowd of testers.
`on_premise` | Run against your internal testers. | - On-premise is enabled for your account.
#### Default behavior
If no `execution_method` parameter is passed in, the created run will run against the run group's default execution method.

### `release`
A string used to link a run to a release (for example, a `git` SHA or tag, a version number, a code name)
#### Type
`string`
#### Default behavior
If no `release` parameter is passed in, the SHA1 hash of the latest commit of the current build (obtained via the `CIRCLE_SHA1` environment variable) will be used.

### `automation_max_retries`
If set to a value > 0 and a test run using automation fails, it will be retried within the same run, up to that number of times.
#### Type
`string`
#### Default behavior
If no `automation_max_retries` parameter is passed in, the created run will use the Run Group's configured `automation_max_retries` setting.

### `branch`
Use a specific Rainforest branch for this run.
#### Type
`string`
#### Default behavior
If no branch parameter is passed in, the main branch will be used.

### `token`
This is not your Rainforest API token, but the name of the environment variable in which it is stored.
#### Type
`env_var_name`
#### Default behavior
If no `token` parameter is passed in, the `RAINFOREST_API_TOKEN` environment variable is assumed to contain your Rainforest QA API token.

### `dry_run`
Set to `true` to run parameter validations without actually starting a run in Rainforest.
#### Type
`boolean`
#### Default behavior
If no `dry_run` parameter is passed in, the run will be started in Rainforest.


### `executor`
The executor to run the `rainforest/run` CircleCI job in.
#### Type
`executor`
#### Allowed values
Any executor which has the Rainforest CLI installed will work.
#### Default behavior
A Docker image with the latest version of the Rainforest CLI installed will be used.

## Rerunning failed tests
The Rainforest Orb uses CircleCI's cache and pipeline concept to know when a build is being rerun. In order for the orb to know within which pipeline it is being executed, you must pass in the [pipeline ID](https://circleci.com/docs/2.0/configuration-reference/#using-pipeline-values) (`<< pipeline.id >>`) to the `pipeline_id` parameter (available on both the `run` job and `run_qa` command).

## Starting multiple Rainforest runs in a single workflow
If the same workflow calls the `run` job more than once, you will need to pass in the `cache_key` and `junit_path` parameters to the extra call sites. This will ensure that all runs are properly triggered and that results for each run are stored separately.

```yaml
workflows:
  your_workflow:
    jobs:
      - rainforest/run:
        # ...
      - rainforest/run:
        # ...
        cache_key: "rainforest-second-run-{{ .Revision }}"
        junit_path: "rainforest_second_run"
```

### Using the `run_qa` command
When using the command, you will need to explicitly define some of the logic [taken care of for you by the job](src/jobs/run.yml):
- restoring from CircleCI cache before the `run_qa` step
- saving the created run's ID when it fails (you can use the `save_run_id` command for this)
- saving the run ID to the CircleCI cache for it to be restored by the next workflow in the pipeline.


\ | Rerun from failed | Rerun from start
--- | --- | ---
Rainforest run failed | reruns failed tests | reruns failed tests
Rainforest run passed | N/A | reruns failed tests

If you want to always run the full Rainforest run, pass in a unique value to `pipeline_id`, for example, `"${CIRCLE_BUILD_NUM}"`.

If you want to run the full Rainforest run for a single failed build, then you will need to kick off a new pipeline.

## Commands
### `install`
Install the Rainforest CLI

#### Requirements
This command requires you to have [`jq`](https://stedolan.github.io/jq/) installed.
#### Parameters
Parameter | Type | Allowed values | Default
 --- | --- | --- | ---
`version` | `string` | `vX.Y.Z`* | `""` (latest)
`platform` | `enum` | `darwin` (MacOS), `linux` | `linux`
`architecture` | `enum` | `386` (32-bit), `amd64` (64-bit) | `amd64`
`install_path` | `string` | | `/usr/local/bin`

&ast; You can find a list of releases [here](https://github.com/rainforestapp/rainforest-cli/releases). The version must be greater than or equal to `v2.19.1`. All prior versions to not have downloadable assets available to install.

### `run_qa`
Start a new Rainforest run
#### Parameters
For more information regarding these parameters, see [Optional Parameters](##Optional%20Parameters).

Parameter | Type | Required | Allowed values | Default
 --- | --- | --- | --- | --- |
`description` | `string` | | any string | `"$CIRCLE_PROJECT_REPONAME - $CIRCLE_BRANCH $CIRCLE_BUILD_NUM $(date -u +'%FT%TZ')"`
`run_group_id` | `string` | ✓ | string evaluating to a positive integer | —
`environment_id` | `string` | | string evaluating to a positive integer | `""`
`custom_url` | `string` | | string evaluating to a URL | `""`
`conflict` | `string` | | `cancel` `cancel-all` | `""`
`execution_method` | `string` | | `automation` `crowd` `automation_and_crowd` `on_premise` | `""`
`release` | `string` | | any string | `"$CIRCLE_SHA1"`
`automation_max_retries` | `string` | | string evaluating to a positive integer | `""`
`branch` | `string` | | any string | `""`
`token` | `env_var_name` | | any environment variable name | `"RAINFOREST_API_TOKEN"`
`dry_run` | `boolean` | | `true` `false` | `false`
`pipeline_id` | `string` | ✓ | any string | —

### `save_run_id`
Save the created run ID to the CircleCI cache
#### Parameters
Parameter | Type | Required | Allowed values | Default
 --- | --- | --- | --- | --- |
`pipeline_id` | `string` | ✓ | any string | —
`when` | `enum` | | `always` `on_success` `on_fail` | `on_fail`

## Executors
### `default`
A Docker image with the latest version of the Rainforest CLI installed.
#### Parameters
Parameter | Type | Allowed values | Default Behavior
 --- | --- | --- | ---
 `tag` | `string` | See [list of tags](https://gcr.io/rf-public-images/rainforest-cli). | The `latest` tag is used.

## Examples
### Using all parameters
```yaml
workflows:
  my_workflow:
    jobs:
      - rainforest/run:
          run_group_id: "123"
          description: Smoke suite
          environment_id: "456"
          conflict: cancel-all
          execution_method: automation
          release: $CIRCLE_TAG
          token: RAINFOREST_QA_API_TOKEN
          executor: my_custom_rainforest_executor
          pipeline_id: << pipeline.id >>
```

### Running only on one branch
Like any other CircleCI job, you can use branch or tag filters:
```yaml
workflows:
  my_workflow:
    jobs:
      - rainforest/run:
          filters:
            branches:
              only:
                - master
          run_group_id: "123"
          pipeline_id: << pipeline.id >>
```

## Orb Release Process
This section describes the release process for the orb itself:
1. Create a feature branch and do your work.
1. Update the version in the repo (i.e. `rake update:<major|minor|patch>`).
1. Push the feature branch to Github to kick off the `lint-pack_validate_publish-dev` workflow in CircleCI.
1. When the `lint-pack_validate_publish-dev` workflow completes successfully, it will trigger the `integration-tests_prod-release` workflow to test the orb.
1. If the `integration-tests_prod-release` workflow passes, get review and merge to master.
1. Create a [GitHub Release](https://github.com/rainforestapp/rainforest-orb/releases/new) with the proper `v`-prefixed version tag (i.e. `v5.0.0`). List **Bugfixes**, **Breaking changes**, and **New features** (if present), with links to the PRs. See [previous releases](https://github.com/rainforestapp/rainforest-orb/releases) for an idea of the format we've been using.

If you want to run an integration test against Rainforest, create a new branch in the Rainforest repo and update the `.circleci/config.yml` to use the dev version of the orb and add a job to kick-off a Rainforest run.

If the build fails in Circle with an error like:
```
Dev versions of orbs are only valid for 90 days after publishing.
```

Then run:
```
(circleci orb pack src | circleci orb validate -) && (circleci orb pack src | circleci orb publish - rainforest-qa/rainforest@dev:alpha)
```
