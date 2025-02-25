---
title: Isolation and concurrency
pcx_content_type: concept
weight: 6
---

# Isolation and concurrency

Review how the Workers Vitest integration runs your tests, how it isolates tests from each other, and how it imports modules.

## Run tests

When you run your tests with the Workers Vitest integration, Vitest will:

1. Read and evaluate your configuration file using Node.js.
2. Run any [`globalSetup`](https://vitest.dev/config/#globalsetup) files using Node.js.
3. Collect and sequence test files.
4. For each Vitest project, depending on its configured isolation and concurrency, start one or more [`workerd`](https://github.com/cloudflare/workerd) processes, each running one or more Workers.
5. Run [`setupFiles`](https://vitest.dev/config/#setupfiles) and test files in `workerd` using the appropriate Workers.
6. Watch for changes and re-run test files using the same Workers if the configuration has not changed.

## Isolation and concurrency models

The [`isolatedStorage` and `singleWorker`](/workers/testing/vitest-integration/configuration/#workerspooloptions-definition) configuration options both control isolation and concurrency. The Workers Vitest integration tries to minimise the number of `workerd` processes it starts, reusing Workers and their module caches between test runs where possible. The current implementation of isolated storage requires each `workerd` process to run one test file at a time, and does not support `.concurrent` tests. A copy of all auxiliary `workers` exists in each `workerd` process.

By default, the `isolatedStorage` option is enabled. We recommend you enable the `singleWorker: true` option if you have lots of small test files.

### `isolatedStorage: true, singleWorker: false` (Default)

In this model, a `workerd` process is started for each test file. Test files are executed concurrently but `.concurrent` tests are not supported. Each test will read/write from an isolated storage environment, and bind to its own set of auxiliary `workers`.

![Isolation Model: Isolated Storage & No Single Worker](/images/workers/testing/vitest/isolation-model-3-isolated-storage-no-single-worker.svg)

### `isolatedStorage: true, singleWorker: true`

In this model, a single `workerd` process is started with a single Worker for all test files. Test files are executed in serial and `.concurrent` tests are not supported. Each test will read/write from an isolated storage environment, and bind to the same auxiliary `workers`.

![Isolation Model: Isolated Storage & Single Worker](/images/workers/testing/vitest/isolation-model-4-isolated-storage-single-worker.svg)

### `isolatedStorage: false, singleWorker: false`

In this model, a single `workerd` process is started with a Worker for each test file. Tests files are executed concurrently and `.concurrent` tests are supported. Every test will read/write from the same shared storage, and bind to the same auxiliary `workers`.

![Isolation Model: No Isolated Storage & No Single Worker](/images/workers/testing/vitest/isolation-model-1-no-isolated-storage-no-single-worker.svg)

### `isolatedStorage: false, singleWorker: true`

In this model, a single `workerd` process is started with a single Worker for all test files. Test files are executed in serial but `.concurrent` tests are supported. Every test will read/write from the same shared storage, and bind to the same auxiliary `workers`.

![Isolation Model: No Isolated Storage & Single Worker](/images/workers/testing/vitest/isolation-model-2-no-isolated-storage-single-worker.svg)

## Modules

Each Worker has its own module cache. As Workers are reused between test runs, their module caches are also reused. Vitest invalidates parts of the module cache at the start of each test run based on changed files.

The Workers Vitest pool works by running code inside a Cloudflare Worker that Vitest would usually run inside a [Node.js worker thread](https://nodejs.org/api/worker_threads.html). To make this possible, the pool requires the [`nodejs_compat`](/workers/configuration/compatibility-dates/#nodejs-compatibility-flag) and [`export_commonjs_default`](/workers/configuration/compatibility-dates/#commonjs-modules-do-not-export-a-module-namespace) compatibility flags to be enabled. The pool also configures `workerd` to use Node-style module resolution and polyfills required `node:*` modules not provided by `nodejs_compat`.

{{<Aside type="warning">}}

Using the pool may cause your Worker to behave differently when deployed than during testing, as Node-style resolution and additional polyfills will be available to your Worker's source code and dependencies too.

{{</Aside>}}
