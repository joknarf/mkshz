# mkshz
self extracting shell gzip tarball packager.  
* Simple shell script to generate self extracting shell script with embedded gzipped tarball.  
* Minimalist universal packager with pre / post install command without any complex package factory to setup.  
* Simple deployment of application using simple unique shell script package.

## prerequisites

* bash
* sed (gnu/bsd)
* tar
* gzip

## usage

usage: `mkshz <file.shz> <dir> [<command> [--pre-extract <command>] <tar options>]`

Creates a shell script `<file.shz>` with embedded tgz archive created from `<dir>` content.
* The script will extract the archive to the current directory on execution.
* A `<command>` in `<dir>` can be specified to run after extraction with arguments passed to `<file.shz>`.
* A pre-extract `<command>` in `<dir>` can be specified to be extracted and run before the whole archive is extracted.

Example usage:
```
$ mkshz install.shz app/ bin/config.sh --pre-extract bin/pre-stop.bash --exclude=.git --exclude=*.o
```
Notes:
* The `<command>` arguments are relative to `<dir>`.
* The script generated will be created with a shebang line for bash.
* The pre-extract `<command>` will be extracted and executed before extracting whole archive
* The pre-extract command does not receive any arguments.
* If the pre-extract `<command>` exit code is not zero, the script will abort whole extraction.

## Example

```
$ mkshz installapp.shz app bin/setup.sh --pre-extract bin/stopapp.sh --exclude=.git
```

* Generated script `<installapp.shz>`:
```
#!/bin/bash
# Script created with mkshz
# This script extracts embedded tgz archive
# extract archive directory: 'app'
# pre-extract command: 'app/bin/stopapp.sh'
# post-extract command: 'app/bin/setup.sh'
# to extract the archive without command execution, use:
#  ./installapp.shz x-shz
# shz_verbose=1 env variable can be set to enable tar output
#
dir="app"
pre="app/bin/stopapp.sh"
cmd="app/bin/setup.sh"
shz="${0##*/}"
[ "$1" = "x-shz" ] && cmd="" pre=""
extract() {
    [ "$shz_verbose" ] && v=v || v=""
    sed '1,/^__tgz__/d' "$0"| tar xz${v}f - "$@" >&2 || exit 1
}
# Pre-extract script:
[ "$pre" ] && {
  echo "$shz: Extracting/Executing pre-extract script ($pre)" >&2
  extract "$pre"
  "$pre" || { echo "$shz: Error: Failed to execute pre-extract script: $pre" >&2; exit 1; }
}
echo "$shz: Extracting archive directory '$dir' in '$PWD'" >&2
extract
[ "$cmd" ] || exit 0
echo "$shz: Executing command: $cmd $*" >&2
[ -x "$cmd" ] && exec "$cmd" "$@"
echo "$shz: Error: Command $cmd not found or not executable." >&2
exit 1
__tgz__ ====== binary data starts here ==========================================
<tgz binary>....
```
* script execution
```
user@host:/myapp/distrib $ ./installapp.shz --register myhost
installapp.shz: Extracting/Executing pre-extract script: app/bin/stopapp.sh
  stopapp.sh: Stopping app
  stopapp.sh: Done
installapp.shz: Extracting archive directory 'app' in '/myapp/distrib'
installapp.shz: Executing command: app/bin/setup.sh --register myhost
  config.sh: Configuring app
  config.sh: Starting app
```
