<!--
Copyright 2025 The Flutter Authors
Use of this source code is governed by a BSD-style license that can be
found in the LICENSE file or at https://developers.google.com/open-source/licenses/bsd.
-->
# DevTools benchmark tests

There are two types of benchmarks that we currently support: size and performance.
1. `devtools_benchmarks_test.dart` - measures DevTools frame times.
2. `web_bundle_size_test.dart` - measures DevTools release build size.

The benchmark tests are run automatically on the CI.
See the "benchmark-performance" and "benchmark-size" jobs.

## Running benchmark tests locally

> [!NOTE] 
> The performance and size benchmarks cannot be run concurrently
> (e.g. by running `flutter test benchmark/`). See the [#caveats](#caveats)
> section below.

All of the commands below should be run from the `packages/devtools_app` directory.

### Performance benchmarks

To run the performance benchmark tests locally, run:
```sh
dart run benchmark/scripts/run_benchmarks.dart
```

Provide arguments to the `run_benchmarks.dart` script in order to:
* compute the average of multiple benchmark runs
* compute a delta against a prior benchmark run
* save the benchmark results to a file
* run the benchmarks in the browser
* run the benchmarks with the `dart2wasm` compiler
* run the benchmarks with the `skwasm` web renderer

Run `dart run benchmark/scripts/run_benchmarks.dart -h` to see details.

To run the performance benchmark test locally, run:
```sh
flutter test benchmark/devtools_benchmarks_test.dart
```

### Size benchmarks

To run the size benchmark test locally, run:
```sh
flutter test benchmark/web_bundle_size_test.dart
```

### Caveats

The size benchmark must be ran by itself because it actually modifies the
`devtools_app/build` folder to create and measure the release build web bundle size.
If this test is ran while other tests are running, it can affect the measurements
that the size benchmark test takes, and it can affect the DevTools build that
the other running tests are using.

## Adding a new benchmark test or test case

The tests are defined by "automators", which live in the `benchmark/test_infra/automators`
directory. To add a new test or test case, either modify an existing automator or add
a new one for a new screen. Follow existing examples in that directory for guidance.

## Comparing two benchmark test runs

There are two ways to calculate the delta between two benchmark test runs:

1. Compare two benchmarks from file:
    * In order to compare two different benchmark runs, you first need to run the
      benchmark tests and save the results to a file:
        ```sh
        dart run benchmark/scripts/run_benchmarks.dart --save-to-file=/Users/me/baseline.json
        dart run benchmark/scripts/run_benchmarks.dart --save-to-file=/Users/me/test.json
        ```
    * Then, to compare the benchmarks and calculate deltas, run:
        ```sh
        dart run benchmark/scripts/compare_benchmarks.dart /Users/me/baseline_file.json /Users/me/test_file.json
        ```

2. Compare a new benchmark run with a benchmark from file:
    * pass the baseline benchmark file path to the `--baseline` flag when running the
      `run_benchmarks.dart` script:
        ```sh
        dart run benchmark/scripts/run_benchmarks.dart --baseline=/Users/me/baseline_file.json``
        ```

# Compare dart2wasm performance to dart2js

Here are step-by-step instructions for comparing the performance
of `dart2wasm` with `dart2s` for DevTools benchmark tests.

1. From the `devtools_app` directory, run
    ```sh
    dart run benchmark/scripts/dart2wasm_performance_diff.dart
    ```

    Optional flags (to see the full usage description, run the above command with `-h`):
    * `--average-of`: specifies the number of times to run each benchmark.
    The returned results will be the average of all the benchmark runs when
    this value is greater than 1. If left unspecified, each benchmark will
    only be run once.
    * `--save-to-file`: the absolute file path to the location where you want the
    benchmark comparison to be saved as a CSV file. This will default to the
    `Downloads/` directory if left unspecified.
    * `--baseline`: the absolute file path to the baseline benchmark data (dart2js)
    to use for this performance diff. Only use this option when you do not
    want to run a new benchmark for the dart2js results.
    * `--test`: the absolute file path to the test benchmark data (dart2wasm)
    to use for this performance diff. Only use this option when you do not
    want to run a new benchmark for the dart2wasm results.

2. Open the CSV file in a spreadsheet application (Google Sheets, Excel, etc.) for viewing.