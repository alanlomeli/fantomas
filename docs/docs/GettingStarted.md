---
category: Documentation
categoryindex: 1
index: 1
---
# Getting Started

* [Command line](#Command-line-tool-API)
* [Fake build system](FAKE.html): Fantomas can be easily integrated with FAKE build system
* [JetBrains Rider](Rider.html)
* [VSCode](VSCode.html)
* [Visual Studio](VisualStudio.html)
* Try Fantomas [online](https://fsprojects.github.io/fantomas-tools/#/fantomas/preview)

## Command line tool / API
TODO: Style blockquotes</br>
Create a [.NET tool manifest](https://docs.microsoft.com/en-us/dotnet/core/tools/local-tools-how-to-use) to install tools locally
> dotnet new tool-manifest

Install the command line tool with:
> dotnet tool install fantomas

or install the tool globally with
> dotnet tool install -g fantomas

For the overview how to use the tool, you can type the command

	dotnet fantomas --help

```
USAGE: dotnet fantomas [--help] [--recurse] [--force] [--profile] [--fsi <string>] [--stdin] [--stdout] [--out <string>] [--check] [--daemon] [--version] [<string>...]

INPUT:

    <string>...           Input paths: can be multiple folders or files with *.fs,*.fsi,*.fsx,*.ml,*.mli extension.

OPTIONS:

    --recurse, -r         Process the input folder recursively.
    --force               Print the source unchanged if it cannot be parsed correctly.
    --profile             Print performance profiling information.
    --fsi <string>        Read F# source from stdin as F# signatures.
    --stdin               Read F# source from standard input.
    --stdout              Write the formatted source code to standard output.
    --out <string>        Give a valid path for files/folders. Files should have .fs, .fsx, .fsi, .ml or .mli extension only.
    --check               Don't format files, just check if they have changed. Exits with 0 if it's formatted correctly, with 1 if some files need formatting and 99 if there was an internal error
    --daemon              Daemon mode, launches an LSP-like server to can be used by editor tooling.
    --version, -v         Displays the version of Fantomas
    --help                display this list of options.

```

You have to specify an input path and optionally an output path. 
The output path is prompted by `--out` e.g.

	dotnet fantomas ../../../../tests/stackexchange/array.fs --out ../../../../tests/stackexchange_output/array.fs 

Both paths have to be files or folders at the same time. 
If they are folders, the structure of input folder will be reflected in the output one. 
The tool will explore the input folder recursively if you set `--recurse` option.
If you omit the output path, Fantomas will overwrite the input files.

### Check mode

*starting version 3.3*

Verify that a single file or folder was formatted correctly.

> dotnet fantomas --check Source.fs

This will verify if the file `Source.fs` still needs formatting.
If it does, the process will return exit code 99.
In the case that the file does not require any formatting, exit code 0 is returned.
Unexpected errors will return exit code 1.

This scenario is meant to be executed in a continuous integration environment, to enforce that the newly added code was formatted correctly.

### Multiple paths

*starting version 4.5*

Multiple paths can be passed as last argument, these can be both files and folders.  
This cannot be combined with the `--out` and `--stdout` flags.  
When combined with the `--recurse` flag, all passed folders will be processed recursively.

One interesting use-case of passing down multiple paths is that you can easily control the selection and filtering of paths from the current shell.

Consider the following PowerShell scripts:
```powershell
# Create an array with paths
$files =
     Get-ChildItem src/*.fs -Recurse # Find all *.fs files in src,
     | Where-Object { $_.FullName -notlike "*obj*" } # ignore files in the `obj` folder
     | ForEach-Object { $_.FullName } #  and select the full path name.

& "dotnet" "fantomas" $files
```

```powershell
# Filter all added and modified files in git
$files = git status --porcelain | Where-Object { $_ -match "^\s?A?M(.*)\.fs(x|i)?$" } | ForEach-Object { $_.TrimStart("AM").TrimStart(" ", "M") }
& "dotnet" "fantomas" $files
```

Or usage with `find` on unix:

`find my-project/ -type f -name "*.fs" -not -path "*obj*" | xargs dotnet fantomas --check`