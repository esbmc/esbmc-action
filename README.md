# esbmc-action

An action for verifying files using the [ESBMC](https://github.com/esbmc/esbmc). This action will run ESBMC with the given options on the given files, or all C and C++ files with a git diff between the current and previous commit if none are given. Note that the git diff functionality of this action relies on the user using appropriate naming conventions for their files: ESBMC will only run on files which have a git diff and end with the '.c' or '.cpp' extension. To allow for using git diff with previous commits, this action will also automatically checkout your repository on to the runner with a fetch depth of 2.

This action is supported on Linux, Windows and macOS runners. At the time of creation, ESBMC version 7.6.1 is being used for Linux and Windows, and ESBMC version 7.6 is being used for macOS.

Note that only certain SMT solvers are supported on these pre-built releases. On Linux, the smtlib, z3, boolector, cvc5 and bitwuzla solvers are supported. On macOS, only smtlib is supported. On Windows, only Z3 is supported.

## Usage

```
- name: Run ESBMC
  uses: Goblin57/esbmc-action@vx
```
(Replace 'x' with the desired version of this action).

## Inputs

**esbmc-options:** Command line options to be used with ESBMC. Not required and has a default value of '--incremental-bmc --quiet --color --verbosity 4'.

<details>
<summary>A note on using quotes with ESBMC options</summary>
<br>
You may want to surround certain options with quotes so that you may use spaces in them, for example. Please note that you should use single quotes and not double quotes when doing this: using double quotes will break the action's shell scripts.

Additionally, when using quotes (single only) to surround the file names used with the output options --output-goto, --file-output and --witness-output, note that while on macOS and Linux runners, you can use single quotes and whitespace in them as you please, but for Windows runners, you can not use whitespace after the end of the file name.
</details>

**create-artifact:** Set this option to 'y' to upload the output files generated by the --output-goto, --file-output and --witness-output ESBMC options as an artifact. Any value other than 'y' or 'Y' will be interpreted as negative, and artifacts will not be uploaded. This input is not required and has a default value of 'n' (will not upload artifacts by default).

When one of the output options above are used in the 'esbmc-options' input, an output file will be generated for each run of ESBMC. The name of these output files will be '__x_y', where 'x' is the name of the file that ESBMC ran on and 'y' is the original file name given by the output option, with whitespace removed. If this input is enabled, but no valid output files have been created, a warning will appear in the console stating that no files have been found.

**artifact-retention-days:** This input is used with the 'upload artifact' action. This is the number of days after which any produced artifacts will expire. Minimum 1 day. Maximum 90 days unless changed from the repository settings page. This input is not required and is '0' by default, which sets this to default number as described in the repository settings.

**artifact-compression-level:** A number from 0 to 9 which describes the compression level of the artifact zip file. This input is not required and is '6' by default.

