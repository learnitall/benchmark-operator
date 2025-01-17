# E2E CI

## Introduction

Benchmark-operator ships an end to end CI mechanism that runs by default in the events described in the [e2e workflow](../.github/workflows/e2e.yml).
The main goal of this workflow is to test breaking/dangerous changes pushed into benchmark-operator, this means that it's not needed to run all tests before merging every pull request.

## Workflow

### Bats

[Bats](https://github.com/bats-core/bats-core) is a TAP-compliant testing framework for Bash. It provides a simple way to verify that the UNIX programs you write behave as expected.

A Bats test file is a Bash script with special syntax for defining test cases. Under the hood, each test case is just a function with a description.

```bash
#!/usr/bin/env bats

@test "addition using bc" {
  result="$(echo 2+2 | bc)"
  [ "$result" -eq 4 ]
}

@test "addition using dc" {
  result="$(echo 2 2+p | dc)"
  [ "$result" -eq 4 ]
}
```

If every command in the test case exits with a 0 status code (success), the test passes. This workflow uses bats framework to execute end to end tests, these tests are defined in files with the termination *.bats under the [e2e directory](./e2e)

### Events

This workflow is at the moment of writing this document triggered by the following events:

- A commit is pushed to the master branch: This allows to detect issues when we want a commit is merged to this branch

- Workflow dispatch: This event allows a repository administrator to **manually trigger the worklflow**. The interesting part of this event is that allows to run specific tests using an input text dialog, this input dialog expectes a regular expression which is passed to the `-f` flag of the bats command. (`-f, --filter <regex>      Only run tests that match the regular expression`). For example if you fill out this this dialog with uperf|fio, the workflow will trigger only the tests for FIO and smallfile, test names are defined with the special @test suffix in each bats file. `grep -h @test e2e/*.bats`

- pull_request_target: This event is generated by pull requests, **the PRs must be labeled with "ok to test" to trigger this workflow.**

## Running in local

It's possible to run e2e tests in local, you can trigger them with executing the command: `make e2e-tests`
> Benchmark-operator pod must be running before triggering the tests.

In case you want to test your own code you can use the following command to build, push and deploy your benchmark-operator bits, and eventually trigger e2e test suite.

```shell
$ IMG=<your image location> make image-build image-push deploy e2e-tests
```

It's also possible to trigger specific tests by setting the BATS_TESTS env var with a regex that will be passed to the `-f` flag of bats.

```shell
$ IMG=container-registry.io/org/benchmark-operator:mybranch BATS_TESTS="uperf|fio" make image-build image-push deploy e2e-tests
```

A preconfigured Elasticsearch endpoint is set by default in the file [helpers.bash](../e2e/helpers.bash), however it's possible to set a custom one by exporting the variable `ES_SERVER`.

### Software requirements

Running tests in local has several requirements:

- podman
- parallel
- bats
- make
- kubectl
- oc
