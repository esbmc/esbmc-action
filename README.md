# esbmc-action

An action for verifying files using the [ESBMC](https://github.com/esbmc/esbmc). This action will run ESBMC with the given options on the given files, or all C and C++ files with a git diff between the current and previous commit if none are given. Note that the git diff functionality of this action relies on the user using appropriate naming conventions for their files: ESBMC will only run on files that have a git diff and end with the '.c' or '.cpp' extension. To allow for using git diff with previous commits, this action will also automatically checkout your repository on to the runner with a fetch depth of 2 (this can be disabled with the checkout input).

This action is supported on Linux, Windows, and macOS runners. At the time of creation, ESBMC version 7.6.1 is being used for Linux and Windows, and ESBMC version 7.6 is being used for macOS.

Note that only certain SMT solvers are supported on these pre-built releases. On Linux, the smtlib, z3, boolector, cvc5, and bitwuzla solvers are supported. On macOS, only smtlib is supported. On Windows, only Z3 is supported.

## Version 2.0.0 what's new:

Added the checkout input, which allows users to choose whether or not to checkout at the start of the action. Also added appropriate error message if correct checkout does not occur.

## Usage

```
- name: Run ESBMC
  uses: esbmc/esbmc-action@vx
```
(Replace 'x' with the desired version of this action).

## Inputs

#### esbmc-options:

Command line options to be used with ESBMC. Not required and has a default value of '--incremental-bmc --quiet --color --verbosity 4'.

<details>
<summary>A note on using quotes with ESBMC options</summary>
<br>
You may want to surround certain options with quotes so that you may use spaces in them, for example. Please note that you should use single quotes and not double quotes when doing this; using double quotes will break the action's shell scripts.

Additionally, when using quotes (single only) to surround the file names used with the output options --output-goto, --file-output, --generate-html-output and --witness-output, note that while on macOS and Linux runners, you can use single quotes and whitespace in them as you please, but for Windows runners, you cannot use whitespace after the end of the file name.
</details>

#### files:

Manually specify file paths to be used with ESBMC. If no file paths are given for this input, this action will instead run off files with a git diff. Each file path should be given on their own line, with no quotes or whitespace before or after the path. Example:

```
files: |
    file1.c
    folder/file2.c
```

Note that while you may specify any files to be verified, the builds of ESBMC used only support a limited selection of filetypes.

#### create-artifact:

Set this input to 'y' to upload the output files generated by the --output-goto, --file-output, --generate-html-output and --witness-output ESBMC options as an artifact. Any value other than 'y' or 'Y' for this input will be interpreted as negative, and artifacts will not be uploaded. This input is not required and has a default value of 'n' (will not upload artifacts by default).

When ESBMC generates output files according to its options, one is created for each run of ESBMC. The name of these output files will be 'x__y', where 'x' is the name of the file that ESBMC ran on and 'y' is the original file name given by the output option, with whitespace removed. If this input is enabled but no valid output files have been created, a warning will appear in the console stating that no files have been found. Also note that when using multiple output options on the ESBMC, it is important to name each output file differently, otherwise ESBMC will overwrite the output file.

#### artifact-retention-days:

This input is used with the 'upload artifact' action. This is the number of days after which any produced artifacts will expire. Minimum 1 day. Maximum 90 days unless changed from the repository settings page. This input is not required and is '0' by default, which sets this to the default number as described in the repository settings.

#### artifact-compression-level:

A number from 0 to 9, which describes the compression level of the artifact zip file. This input is not required and is '6' by default.

#### checkout:

Whether or not to automatically checkout the repository at the start of the action, with a fetch-depth of 2. By default, this input is set to 'y', which enables automatic checkout. Any value other than 'y' or 'Y' for this input will be interpreted as negative, and the checkout will not occur.

Note that if this input is set to a negative option, then it will still be necessary to checkout the repository with a fetch depth of 2 for this action to work. This option exists so that the user may install any dependencies for the workflow without them being overwritten by the checkout in this action.

## Outputs

#### artifact-id:

The GitHub ID of the created Artifact, can be used by the REST API. If no artifact is uploaded, this is the empty string.

#### artifact-url:

The URL to download the artifact. Can be used in many scenarios, such as linking to artifacts in issues or pull requests. Users must be logged in in order for this URL to work. This URL is valid as long as the artifact has not expired or the artifact, run, or repository have not been deleted.

## Usage Example

```
name: ESBMC Action Demo

on: push

jobs:
    run-esbmc-action:
        runs-on: ubuntu-latest
        steps:
            - name: Run ESBMC Action
              id: esbmc-action
              uses: esbmc/esbmc-action@v1
              with:
                esbmc-options: "--incremental-bmc --quiet --color --file-output out.txt --verbosity 4"
                create-artifact: y
                artifact-compression-level: 8
           
            - name: Test outputs
              run: |
                echo "${{ steps.esbmc-action.outputs.artifact-id }}"
                echo "${{ steps.esbmc-action.outputs.artifact-url }}"
```

## Environment Variables

This action may create the FILE_DIFF, GOTO_FILENAME, NOSPACE_GOTO_FILENAME, OUTPUT_FILENAME, NOSPACE_OUTPUT_FILENAME, WITNESS_FILENAME, and NOSPACE_WITNESS_FILENAME environment variables.

The FILE_DIFF variable is created if the git diff is used. Each "FILENAME" variable is created if its corresponding output file-creating option is used in the "esbmc-options" input.

This action also uses the GITHUB_ENV and GITHUB_WORKSPACE environment variables.
