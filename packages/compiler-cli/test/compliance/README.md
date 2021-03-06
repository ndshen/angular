# Compliance test-cases

This directory contains rules, helpers and test-cases for the Angular compiler compliance tests.

There are three different types of tests that are run based on file-based "test-cases".

* **Full compile** - in this test the source files defined by the test-case are fully compiled by Angular.
  The generated files are compared to "expected files" via a matching algorithm that is tolerant to
  whitespace and variable name changes.
* **Partial compile** - in this test the source files defined by the test-case are "partially" compiled by
  Angular to produce files that can be published. These partially compiled files are compared directly
  against "golden files" to ensure that we do not inadvertently break the public API of partial
  declarations.
* **Linked** - in this test the golden files mentioned in the previous bullet point, are passed to the
  Angular linker, which generates files that are comparable to the fully compiled files. These linked
  files are compared against the "expected files" in the same way as in the "full compile" tests.

This way the compliance tests are able to check each mode and stage of compilation is accurate and does
not change unexpectedly.


## Defining a test-case

To define a test-case, create a new directory below `test_cases`. In this directory

* add a new file called `TEST_CASES.json`. The format of this file is described below.
* add an empty `GOLDEN_PARTIAL.js` file. This file will be updated by the tooling later.
* add any `inputFiles` that will be compiled as part of the test-case.
* add any `expected` files that will be compared to the files generated by compiling the source files.


### TEST_CASES.json format

The `TEST_CASES.json` defines an object with one or more test-case definitions in the `cases` property.

Each test-case can specify:

* A `description` of the test.
* The `inputFiles` that will be compiled.
* Additional `compilerOptions` and `angularCompilerOptions` that are passed to the compiler.
* A collection of `expectations` definitions that will be checked against the generated files.

Note that there is a JSON schema for the `TEST_CASES.json` file stored at `test_cases/test_case_schema.json`.
You should add a link to this schema at the top of your `TEST_CASES.json` file to provide
validation and intellisense for this file in your IDE.

For example:

```json
{
  "$schema": "../test_case_schema.json",
  "cases": [
    {
      "description": "description of the test - equivalent to an `it` clause message.",
      "inputFiles": ["abc.ts"],
      "expectations": [
        {
          "failureMessage": "message to display if this expectation fails",
          "files": [
            { "expected": "xyz.js", "generated": "abc.js" }, ...
          ]
        }, ...
      ],
      "compilerOptions": { ... },
      "angularCompilerOptions": { ... }
    }
  ]
}
```

### Input files

The input files are the source file that will be compiled as part of this test-case.
Input files should be stored in the directory next to the `TEST_CASES.json`.
The paths to the input files should be listed in the `inputFiles` property of `TEST_CASES.json`.
The paths are relative to the `TEST_CASES.json` file.

If no `inputFiles` property is provided, the default is `["test.ts"]`.


### Expectations

An expectation consists of a collection of expected `files` pairs, and a `failureMessage`, which
is displayed if the expectation check fails.

Each file-pair consists of a path to a `generated` file (relative to the build output folder),
and a path to an `expected` file (relative to the test case).

The `generated` file is checked to see if it "matches" the `expected` file. The matching is
resilient to whitespace and variable name changes.

If no `failureMessage` property is provided, the default is `"Incorrect generated output."`.

If no `files` property is provided, the default is a a collection of objects `{expected, generated}`,
where `expected` and `generated` are computed by taking each path in the `inputFiles` collection
and replacing the `.ts` extension with `.js`.


## Running tests

The simplest way to run all the compliance tests is:

```sh
yarn test-ivy-aot //packages/compiler-cli/test/compliance/...
```

If you only want to run one of the three types of test you can be more specific:

```sh
yarn test-ivy-aot //packages/compiler-cli/test/compliance/full
yarn test-ivy-aot //packages/compiler-cli/test/compliance/linked
yarn test-ivy-aot //packages/compiler-cli/test/compliance/test_cases/...
```

(The last command runs the partial compilation tests.)


## Updating a golden partial file

There is one golden partial file per `TEST_CASES.json` file. So even if this file defines multiple
test-cases, which each contain multiple input files, there will only be one golden file.

The golden file is generated by the tooling and should not be modified manually.

When you first create a test-case, with an empty `GOLDEN_PARTIAL.js` file, or a change is made to
the generated partial output, we must update the `GOLDEN_PARTIAL.js` file.

This is done by running a specific bazel rule of the form:

```sh
bazel run //packages/compiler-cli/test/compliance/test_cases:<path/to/test_case>.golden.update
```

where to replace `<path/to/test_case>` with the path (relative to `test_cases`) of the directory
that contains the `GOLDEN_PARTIAL.js` to update.


## Debugging test-cases

Compliance tests are basically `jasmine_node_test` rules. As such, they can be debugged
just like any other `jasmine_node_test`.  The standard approach is to add `--config=debug`
to the Bazel test command.

It is useful when debugging to focus on a single test-case.


### Focusing test-cases

You can focus a test case by setting `"focusTest": true` in the `TEST_CASES.json` file.
This is equivalent to using jasmine `fit()`.


### Excluding test-cases

You can exclude a test case by setting `"excludeTest": true` in the `TEST_CASES.json` file.
This is equivalent to using jasmine `xit()`.

