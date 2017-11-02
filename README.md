# Diff Enhancement Tools for Git

## Description

This repository houses tools designed to improve the diff output of Git for certain types of files.  Some of the tools, particularly the textconv tools, may find use outside of Git as well.

The tools are grouped into directories based on how they are meant to be used by Git:

| Category     | Operation                                                        |
| ------------ | ---------------------------------------------------------------- |
| textconv     | Convert binary files to text & compare that text in git          |
| git-ext-diff | Run the external diff program and display the raw results in git |
| merge-tool   | Resolve merges on binary files using specialized tools           |

## Setup Git

Before these tools can be used by Git, you need to tell Git:
1. what tool (aka "driver") should be used on a file, and
1. how to run the tool.

To tell Git which driver to use, we leverage the [Git attribute](https://git-scm.com/docs/gitattributes) files:
* `.gitattributes`, which should be checked into the repository, and
* `.git/info/attributes`, which is only available in your local repository (for personal overrides).

In one of these files, add a line with a fileglob to match and the diff driver to use, separated by whitespace:
```
<file*.glob>    diff=<driver>
*.tar           diff=tarball
*.patch         diff=patchfile
```

To tell Git how to run the tool/driver, see the appropriate section below:

### textconv

The textconv tools convert binary files into text output meant for human consumption.  These tools do not alter the file contents on disk, making these conversions temporary and nullipotent.

To make a textconv driver, set the `diff.<driver>.textconv` configuration value, where `<driver>` is the driver in the gitattributes file:

```sh
git config diff.tarball.textconv /path/to/textconv/tarball
git config --global diff.patchfile.textconv /path/to/textconv/filter-patch
```

When setup, `git diff`, `git show`, & `git log -p` use the textconv by default.  This behavior can be changed with `--no-textconv`.

### git-ext-diff

Instead of using Git's diff algorithm, pass the before and after files, along with metadata like the mode & the hash, to an external tool that will perform the diff.  The results from this tool are displayed verbatim.

To make a git-ext-diff driver, set the `diff.<driver>.command` configuration value, where `<driver>` is the driver in the gitattributes file:

```sh
git config diff.tarball.command /path/to/git-ext-diff/tarball
git config --global diff.patchfile.command /path/to/git-ext-diff/interdiff-patch
```

When setup, `git diff` uses the external diff by default, while `git show` and `git log -p` do not.  This behavior can be changed with the `--no-ext-diff` or `--ext-diff` options.

### merge-tool

The merge-tool directory contains scripts that make it possible to merge binary files, which are otherwise unmergeable in Git, requiring manual review and modification.  Often, this is accomplished by converting the binary contents to text with a 1-to-1 mapping, so the text can be merged and then re-converted to binary.

To add a merge-tool, set the `mergetool.<tool>.cmd` configuration value, where `<tool>` is a name for the tool:

```sh
git config mergetool.sqlite3.cmd '/path/to/merge-tool/sqlite3 $BASE $REMOTE $LOCAL $MERGED'
```

**Note:** Unlike the other tools, the argument list must be specified with merge-tools.

To use this merge tool, run `git mergetool --tool=<tool> [<file>]`.  Notice the explit inclusion of the tool to use; this is required.
