---
object: "tool4d-cli"
json_type: null
requires: []
keywords: ["tool4d", "CLI", "headless", "4D", "runtime", "dataless", "startup-method", "validation"]
summary: "How to run 4D projects headlessly using tool4d for validation, code execution, and testing — without a graphical IDE or license."
---

# tool4d — Headless 4D CLI

Reference: https://developer.4d.com/docs/Admin/cli

tool4d is a free, headless, license-free 4D runtime for running project methods from the command line.

## Installation Location

macOS default: `/Applications/4D {Version}/tool4d.app/Contents/MacOS/tool4d`

Example: `/Applications/4D 21 R3/tool4d.app/Contents/MacOS/tool4d`

## Usage

```bash
tool4d --project {path/to/ProjectName.4DProject} --dataless --startup-method {MethodName}
```

**Key flags:**
- `--project`: Path to the `.4DProject` file
- `--dataless`: Run without a data file (no data access, schema-only validation)
- `--startup-method`: Name of the method to execute on startup (must exist in `Sources/Methods/`)
- `--skip-onstartup`: Skip the `On Startup` database method if it exists

## Behavior

- tool4d loads the project, opens a headless process, runs the startup method, and exits.
- `QUIT 4D` can be called to exit the runtime from code.
- The exit code is 0 on success.
- Output from `LOG EVENT` and standard error goes to stderr.
- Methods auto-discovered from `Project/Sources/Methods/` — no registration needed.

## Writing Output From a Method

To write results to a file (since tool4d has no console output by default):

```4d
// Example: write results to a JSON file
var $output : Object
$output:=New object
$output.result:="success"
$output.timestamp:=String(Current date)+" "+String(Current time)

var $path : Text
$path:=Get 4D folder(Database folder)+"output.json"
TEXT TO DOCUMENT($path; JSON Stringify($output; *))

QUIT 4D
```

- `Get 4D folder(Database folder)` returns the project root (parent of `Project/`).
- Always call `QUIT 4D` at the end to cleanly exit the runtime.

## Validating a Project

To verify that a 4D project is structurally valid:

1. Create a simple test method (e.g., `Project/Sources/Methods/validate.4dm`):

```4d
var $output : Object
$output:=New object("status"; "ok")

var $path : Text
$path:=Get 4D folder(Database folder)+"validation.json"
TEXT TO DOCUMENT($path; JSON Stringify($output; *))

QUIT 4D
```

2. Run tool4d:

```bash
/path/to/tool4d --project /path/to/Project/ProjectName.4DProject --dataless --startup-method validate
```

3. Check: exit code 0 and the output file contains `{"status":"ok"}`.

If tool4d fails to launch or the method is not found, the project has structural issues (bad XML, missing files, invalid identifiers).

## Writing 4D Code

When `tokenizedText` is `false` in the `.4DProject` file, write **plain 4D code without token suffixes**:

```4d
// CORRECT (tokenizedText: false)
var $col : Collection
$col:=New collection(1; 2; 3)
ALERT("Hello")

// WRONG — do NOT add token suffixes
var $col : Collection
$col:=New collection:C1472(1; 2; 3)
ALERT:C41("Hello")
```

## Notes

- tool4d does NOT require a 4D license.
- The `--dataless` flag is essential for schema validation (no data file needed).
- If a method name is not found, tool4d exits with a non-zero code and an error on stderr.
