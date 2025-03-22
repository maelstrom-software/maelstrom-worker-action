![Maelstrom Logo (Dark Compatible)](https://github.com/maelstrom-software/maelstrom/assets/146376379/7b46a1c1-e67f-412a-b618-42f7e2c25139)

# Maelstrom Worker GitHub Action

This GitHub action installs and runs the Maelstrom worker.

[Maelstrom](https://github.com/maelstrom-software/maelstrom) is a suite of
tools for running tests in isolated micro-containers locally on your machine or
distributed across arbitrarily large clusters. Maelstrom currently has test
runners for Rust, Go, and Python, with more on the way. For more information
about Maelstrom in general, please see the [main
repository](https://github.com/maelstrom-software/maelstrom), the
[web site](https://maelstrom-software.com), or the
[documentation](https://maelstrom-software.com/doc/book/latest).

# Usage

```yml
jobs:
  run-tests:
    name: Run Tests
    # Both ubuntu-22.04 and ubuntu-24.04 are supported.
    # Both x86-64 and ARM (AArch64) are supported.
    # The architecture needs to be the same as the workers (but not the broker).
    runs-on: ubuntu-24.04

    steps:

    # This action installs and configures (via environment variables)
    # cargo-maelstrom so it can be run simply with `cargo maelstrom`.
    - name: Install and Configure cargo-maelstrom
      uses: maelstrom-software/cargo-maelstrom@v1

    # This action installs and configures (via environment variables)
    # maelstrom-go-test so it can be run simply with `maelstrom-go-test`.
    - name: Install and Configure maelstrom-go-test
      uses: maelstrom-software/maelstrom-go-test-action@v1

    - name: Check Out Repository
      uses: actions/checkout@v4

      # You can now run `cargo maelstrom` however you want.
    - name: Run Tests
      run: cargo maelstrom

      # You can now run `maelstrom-go-test` however you want.
    - name: Run Tests
      run: maelstrom-go-test

  # This is the broker job. It runs in parallel to the worker jobs and the #
  # client jobs.
  maelstrom-broker:
    name: Maelstrom Broker
    runs-on: ubuntu-24.04 # Can be ARM or AMD. It doesn't matter.
    steps:
    - name: Install and Run Maelstrom Broker
      uses: maelstrom-software/maelstrom-broker-action@v1

  # These are the worker jobs. Tests will execute on one of these workers.
  maelstrom-worker:
    strategy:
      matrix:
        worker-number: [1, 2, 3, 4]
    name: Maelstrom Worker ${{ matrix.worker-number }}
    runs-on: ubuntu-24.04 # Must be the same as the test-running job(s).
    steps:
    - name: Install and Run Maelstrom Worker
      uses: maelstrom-software/maelstrom-worker-action@v1

  # This job stops the cluster. It needs to depend on the test-running job(s).
  stop-maelstrom:
    runs-on: ubuntu-24.04 # Can be ARM or AMD. It doesn't matter.
    if: ${{ always() }}
    needs: [run-tests]
    steps:
    - name: Install and Configure maelstrom-go-test
      uses: maelstrom-software/maelstrom-admin-action@v1
    - name: Stop Maelstrom Cluster
      run: maelstrom-admin stop
```

# How to Use

Run multiple Maelstrom worker jobs in parallel with Maelstrom test runners,
like `cargo-maelstrom`, `maelstrom-go-test`, or `maelstrom-pytest`, and one job
running `maelstrom-broker`. See [this example
workflow](https://github.com/maelstrom-software/maelstrom-examples/blob/main/.github/workflows/ci-base.yml).

# What it Does

This action installs
[`maelstrom-worker`](https://maelstrom-software.com/doc/book/latest/worker.html),
sets a [sysctl to allow unprivileged user
namespaces](https://ubuntu.com/blog/ubuntu-23-10-restricted-unprivileged-user-namespaces),
then runs `maelstrom-worker` until completion.

The worker will connect to the broker, execute tests as long as the broker
provides them, and then exit when the broker itself exits.

# Learn More

You can find documentation for running Maelstrom in GitHub
[here](https://maelstrom-software.com/doc/book/latest/github.html).

# Licensing

This project is available under the terms of either the Apache 2.0 license or the MIT license.
