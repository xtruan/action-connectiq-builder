# ConnectIQ Builder GitHub Action

ConnectIQ Builder is a GitHub Action (`adamjakab/action-connectiq-builder@v1`) that can be used to test and build ConnectIQ applications.

## Important notice

The `PACKAGE` operation builds the packaged app inside the docker container but it does not yet have a way to upload this package and attach it to a release of the repository. This is the goal of this operation but it requires more work.

## Usage

USe it in your github workflow as `uses: adamjakab/action-connectiq-builder@v1` with the inputs described below.

### Inputs

The following inputs are available under the `with` keyword:

- `operation`: Required. [Default: TEST]. The main two operations you can chose between are:

  - `TEST` - compiles your application with the `--unit-test` flag and will run run those tests for the specified device. If any of the tests fail will cause the workflow to fail.
  - `PACKAGE` - packages your application by running `monkeyc` with `--package-app` and `--release` flags.
  - `INFO` - displays some information. Used for debugging purposes.

- `path`: Required. [Default: .] The path of the application to test. By default, the root of the repository will be used.

- `device`: Required. [Default: fr235] The id of the device used to run the operation. By default, a Forerunner 235 device is be used. The full list of devices can be found [here](https://developer.garmin.com/connect-iq/reference-guides/devices-reference/#devicereference).

- `devices`: Optional. [Default: empty] The comma separated list of device IDs to on which the opration will be run (test only). If this input is used, the `device` option will be ignored. This input was created so that instead of running tests on multiple separate containers, you can pass the list into one container and use that to run all tests.

- `type_check_level`: Required. [Default: 2] The type level check used when compiling the application with `monkeyc`. The available options (0-3) are described [here](https://developer.garmin.com/connect-iq/monkey-c/monkey-types/).

- `certificate`: Optional. The base64 encoded version of the certificate to be used to compile the application. If not specified, a temporary certificate will be used for the test operation. For `PACKAGE` operation you must supply your own. Make sure to store it as a repository secret and add the variable name to your workflow. To convert your existing developer key into a base64 encoded version run `cat my_developer_key | base64 -w0`. The `-w0` argument is important because it will not wrap the lines (no line breaks).

- `package_name`: Optional. The file name of the package that is built when operation mode is PACKAGE.

- `verbose`: Optional. [Default: 0] Set it to 1 to get more verbose output from the scripts for debugging purposes.

## Outputs

- `status`: If the opearion ran successfully, the value of this output will be `success`. Otherwise `failure`.

- `package_path`: The path of the generated package (when operation = `PACKAGE`) relative to the input path. This can be useful if you want to add the package to a release. you can reference the package in the workflow as `${{ github.workspace }}/${{ steps.package_app.outputs.package_path }}` where `package_app` is the id of this step.

### Examples

A simple usage for `TEST` operation will look something like this:

```yml
steps:
  - name: Test my garmin app
    id: run_tests
    uses: adamjakab/action-connectiq-builder@v1
    with:
      operation: TEST
      device: fr235
```

It is also very common to use a matrix of devices to `TEST` against all of them. That can be easily set up by having a setup similar to this:

```yml
jobs:
  test:
    name: Matrix Test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        device: [fenix3, fr235, vivoactive4]
    steps:
      - uses: actions/checkout@v4
      - name: Test on device ${{ matrix.device }}
        id: run_tests
        uses: adamjakab/action-connectiq-builder@v1
        with:
          operation: TEST
          device: ${{ matrix.device }}
```

However, if you want to test multiple devices without having to spin up one container for each, you can (using the `devices` option):

```yml
steps:
  - name: Multiple tests
    id: run_tests
    uses: adamjakab/action-connectiq-builder@v1
    with:
      operation: TEST
      devices: fenix3,fr235,vivoactive4
```

To use the `PACKAGE` operation the configuration will need to have your developer key (`certificate`):

```yml
steps:
  - name: Package my garmin app
    id: package_app
    uses: adamjakab/action-connectiq-builder@v1
    with:
      operation: PACKAGE
      certificate: ${{ secrets.connectiq_dev_key }}
      package_name: myApp_${{ github.ref_name }}.iq
```

## Credits

This repo is a fork of the [matco/action-connectiq-tester](https://github.com/matco/action-connectiq-tester) repo.

## Notes

- This acion relies on the docker image [ghcr.io/adamjakab/connectiq-builder](https://ghcr.io/adamjakab/connectiq-builder) built by the [adamjakab/connectiq-builder](https://github.com/adamjakab/connectiq-builder) project.

- Usign secrets in workflow: https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#using-secrets-in-a-workflow

- how to store certificates in github: https://josh-ops.com/posts/storing-certificates-as-github-secrets/

- Notes updates: After a new version tag has been pushed to GH, the v1 branch needs to be updated so that it is aligned with the master branch at the new tag. Additionally (optionally) the `v1` release can be updated to reference the new tag.
