# Rainforest QA CircleCI Orb
`v0.1.0`

## Sections
* [Prerequisites](##Prerequisites)
* [Base Usage](##Base%20Usage)
* [Optional Parameters](##Optional%20Parameters)
* [Executors](##Executors)
* [Examples](##Examples)

## Prerequisites

### A Rainforest QA account

If you don't already have one, we have a free trial that you can sign up for [here](https://app.rainforestqa.com/trial).

### A Rainforest QA API token

You can find yours on the [Integrations](https://app.rainforestqa.com/settings/integrations) setting page.
Do not expose this token in your `circleci/.config.yml` file. Instead, [use an environment variable in CircleCI](https://circleci.com/docs/2.0/env-vars/#setting-an-environment-variable-in-a-project). The default name for this token is `RAINFOREST_TOKEN`. You may use a different name, and pass that in to the `rainforest/run` command (see below).

### A run group with at least one test

Run groups are a way to group tests that should be run together (for example, a smoke suite you might want to run on every deploy). For more information on run groups, see [this help article](https://help.rainforestqa.com/en/articles/3230981-run-groups).

Once you have a run group which contains at least one test (included either directly in the run group or via a Feature added to the run group), you can run it in CircleCI builds using this orb. You will need its ID (visible at the end of the run group URL: `https://app.rainforestqa.com/run_groups/<ID>`).

## Base usage

```yaml
# .circleci/config.yml

# To use orbs, your CircleCI config file must use version 2.1
version: 2.1

# If you don't have a top-level `orbs` section, add one
orbs:
# Add the Rainforest orb to that list
  - rainforest: rainforest-qa/rainforest@0.1.0

# In your workflows, add it as a job to be run
workflows:
  your_workflow:
    jobs:
      - rainforest/run:
          # Replace the value with the ID of whichever run group you want to run
          run_group_id: "123"
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
Use a specific environment for this run.
#### Type
`string`
#### Default behavior
If no `environment_id` parameter is passed in, the created run will use the Run Group's default environment.

### `conflict`
How we should handle currently active runs.
#### Type
`enum`
#### Allowed values
Value | Behavior
--- | ---
`abort` | Abort all other runs in the same environment.
`abort-all` | Abort all other runs, regardless of environment.
#### Default behavior
If no `conflict` parameter is passed in, then no active runs will be aborted.

### `crowd`
The crowd to use for this run.
#### Type
`enum`
#### Allowed values
Value | Behavior | Requirements
--- | --- | ---
`default` | Run against our global crowd of testers.
`automation` | Run against our automation agent. | - Automation is enabled for your account.<br />- All tests in the run group are written in <abbr title="Rainforest Test Language">RTL</abbr>.<br />- No tests use a Plain Language action.
`on_premise_crowd` | Run against your internal testers. | - On-premise is enabled for your account.
#### Default behavior
If no `crowd` parameter is passed in, the created run will run against the run group's default crowd.

### `release`
A string used to link a run to a release (for example, a `git` SHA or tag, a version number, a code name)
#### Type
`string`
#### Default behavior
If no `release` parameter is passed in, the SHA1 hash of the latest commit of the current build (obtained via the `CIRCLE_SHA1` environment variable) will be used.

### `token`
This is not your Rainforest API token, but the name of the environment variable in which it is stored.
#### Type
`env_var_name`
#### Default behavior
If no `token` parameter is passed in, the `RAINFOREST_TOKEN` environment variable is assumed to contain your Rainforest QA API token.

### `executor`
The executor to run the `rainforest/run` CircleCI job in.
#### Type
`executor`
#### Allowed values
Any executor which has the Rainforest CLI installed will work.
#### Default behavior
A Docker image with the latest version of the Rainforest CLI installed will be used.

## Executors
### `default`
A Docker image with the latest version of the Rainforest CLI installed.
### Parameters
#### `tag`
##### Type
`string`
##### Allowed values
See [list of tags](https://gcr.io/rf-public-images/rainforest-cli).
##### Default behavior
The `latest` tag is used.

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
          conflict: abort-all
          crowd: automation
          release: $CIRCLE_TAG
          token: RAINFOREST_QA_API_TOKEN
          executor: my_custom_rainforest_executor
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
```
