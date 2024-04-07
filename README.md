# clangd-tidy: A Faster Alternative to clang-tidy

## Motivation

[clang-tidy](https://clang.llvm.org/extra/clang-tidy/) is a powerful tool for static analysis of C++ code. However, it's [widely acknowledged](https://www.google.com/search?q=clang-tidy+slow) that clang-tidy takes a significant amount of time to run on large codebases, particularly when enabling numerous checks. This often leads to the dilemma of disabling valuable checks to expedite clang-tidy execution.

In contrast, [clangd](https://clangd.llvm.org/), the language server with built-in support for clang-tidy, has been [observed](https://stackoverflow.com/questions/76531831/why-is-clang-tidy-in-clangd-so-much-faster-than-run-clang-tidy-itself) to be significantly faster than clang-tidy when running the same checks. It provides diagnostics almost instantly upon opening a file in your editor. The key distinction lies in the fact that clang-tidy checks the codes from all headers (although it suppresses the warnings from them by default), whereas clangd only builds AST from these headers.

Unfortunately, there seems to be no plan within LLVM to accelerate the standalone version of clang-tidy. This project addresses this by offering a faster alternative to clang-tidy, leveraging the speed of clangd. It acts as a wrapper for clangd, running it in the background and collecting diagnostics. Designed as a drop-in replacement for clang-tidy, it seamlessly integrates with existing build systems and CI scripts.

## Comparison with clang-tidy

**Pros:**
- clangd-tidy is significantly faster than clang-tidy (over 10x in my experience).
- clangd-tidy can check header files individually, even if they are not included in the compilation database.
- clangd-tidy groups diagnostics by files -- no more duplicated diagnostics from the same header!
- clangd-tidy supports [`.clangd` configuration files](https://clangd.llvm.org/config), offering features not supported by clang-tidy.
    - Example: Removing unknown compiler flags from the compilation database.
        ```yaml
        CompileFlags:
          Remove: -fabi*
        ```
    - Example: Adding IWYU include checks.
        ```yaml
        Diagnostics:
          # Available in clangd-14
          UnusedIncludes: Strict
          # Require clangd-17
          MissingIncludes: Strict
        ```
- Refer to [Usage](#usage) for more features.

**Cons:**
- clangd-tidy lacks support for the `--fix` option. (Consider using code actions provided by your editor if you have clangd properly configured, as clangd-tidy is primarily designed for speeding up CI checks.)
- clangd-tidy silently disables [several](https://searchfox.org/llvm/rev/cb7bda2ace81226c5b33165411dd0316f93fa57e/clang-tools-extra/clangd/TidyProvider.cpp#199-227) checks not supported by clangd.

## Prerequisites

- [clangd](https://clangd.llvm.org/)
- Python 3.8+ (may work on older versions, but not tested)
- [tqdm](https://github.com/tqdm/tqdm) (optional)

## Usage

```
usage: clangd-tidy [-h] [-p COMPILE_COMMANDS_DIR] [-j JOBS] [-o OUTPUT]
                   [--clangd-executable CLANGD_EXECUTABLE]
                   [--allow-extensions ALLOW_EXTENSIONS]
                   [--fail-on-severity SEVERITY] [--tqdm] [--github]
                   [--git-root GIT_ROOT] [-v] [-c CONTEXT]
                   filename [filename ...]

Run clangd with clang-tidy and output diagnostics. This aims to serve as a faster
alternative to clang-tidy.

positional arguments:
  filename              Files to check. Files whose extensions are not in
                        ALLOW_EXTENSIONS will be ignored.

options:
  -h, --help            show this help message and exit
  -p COMPILE_COMMANDS_DIR, --compile-commands-dir COMPILE_COMMANDS_DIR
                        Specify a path to look for compile_commands.json. If the
                        path is invalid, clangd will look in the current directory
                        and parent paths of each source file. [default: build]
  -j JOBS, --jobs JOBS  Number of async workers used by clangd. Background index
                        also uses this many workers. [default: 1]
  -o OUTPUT, --output OUTPUT
                        Output file for diagnostics. [default: stdout]
  --clangd-executable CLANGD_EXECUTABLE
                        Path to clangd executable. [default: clangd]
  --allow-extensions ALLOW_EXTENSIONS
                        A comma-separated list of file extensions to allow.
                        [default: c,h,cpp,cc,cxx,hpp,hh,hxx,cu,cuh]
  --fail-on-severity SEVERITY
                        On which severity of diagnostics this program should exit
                        with a non-zero status. Candidates: error, warn, info,
                        hint. [default: hint]
  --tqdm                Show a progress bar (require tqdm).
  --github              Append workflow commands for GitHub Actions to output.
  --git-root GIT_ROOT   Root directory of the git repository. Only works with
                        --github. [default: current directory]
  -v, --verbose         Print verbose output from clangd.
  -c CONTEXT, --context CONTEXT
                        Number of lines of code shown either side of the detected
                        issue. [default: 2]

Find more information on https://github.com/lljbash/clangd-tidy.
```

## Acknowledgement

Special thanks to [@yeger00](https://github.com/yeger00) for his [pylspclient](https://github.com/yeger00/pylspclient).

A big shoutout to [clangd](https://clangd.llvm.org/) and [clang-tidy](https://clang.llvm.org/extra/clang-tidy/) for their great work!
