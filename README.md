[![bash](https://img.shields.io/badge/OS-Linux%20|%20macOS%20|%20SunOS%20...-blue.svg)]()

# mkshz
self extracting shell gzip tarball packager.  
> * Simple shell script to generate self extracting shell script with embedded gzipped tarball.  
> * Minimalist universal packager with pre / post install command without any complex package factory to setup.  
> * Simple deployment of application using unique shell script package.
> * Alternative to rpm/deb/... factory for simple projects

## prerequisites

* bash
* sed (gnu/bsd)
* tar / gzip

## usage

usage: `mkshz <file.shz> <dir> [<command> [--tmp-exec] [--pre-extract <command>] <tar options>]`

Creates a shell script `<file.shz>` with embedded tgz archive created from `<dir>` content.
* The script will extract the archive to the current directory on execution.
* A `<command>` in `<dir>` can be specified to run after extraction with arguments passed to `<file.shz>`.
* A pre-extract `<command>` in `<dir>` can be specified to be extracted and run before the whole archive is extracted.
* if `--tmp-exec` flag is set the resulting script will extract to /tmp/shz.XXXX executes the commands and clean the /tmp/shz.XXXXX
* shz_version environment variable can be set to get the version of the archive using `v-shz` parameter.

Example usage:
```
$ shz_version=v1.2 mkshz install.shz app/ bin/config.sh --pre-extract bin/pre-stop.bash --exclude=.git --exclude=*.o
```
Notes:
* The `<command>` arguments are relative to `<dir>`.
* The script generated will be created with a shebang line for bash.
* The pre-extract `<command>` will be extracted and executed before extracting whole archive
* The pre-extract `<command>` does not receive any arguments.
* If the pre-extract `<command>` exit code is not zero, the script will abort whole extraction.

## Example

```
$ shz_version=v1.2 mkshz installapp.shz app bin/setup.sh --pre-extract bin/stopapp.sh --exclude=.git
```

* Generated script `installapp.shz`:
```
#!/bin/bash                                                                                                                                                                         
# Script created with mkshz
# This script extracts embedded tgz archive 
# extract archive directory: 'app'
# pre-extract command: 'app/bin/stopapp.sh'
# post-extract command: 'app/bin/config.sh'
# to extract the archive without command execution, use:
#  ./installapp.shz x-shz
# shz_verbose=1 env variable can be set to enable tar output
# shz_target_dir can be set to specify the target directory for extraction
#
version="v1.2"
dir="app"
pre="app/bin/stopapp.sh"
cmd="app/bin/config.sh"
tmpexec="1"
shz="${0##*/}"
[ "$shz_verbose" ] && v=v || v=""
[ "$1" = "v-shz" ] && echo "$version" && exit 0
[ "$1" = "x-shz" ] && cmd="" pre=""

[ ! "$shz_target_dir" ] && [ "$tmpexec" ] && {
  shz_target_dir=$(mktemp -d /tmp/shz.XXXXXX) || exit 1
  [ "$cmd" ] && trap 'rm -rf "$shz_target_dir"' EXIT
}
: ${shz_target_dir:=$PWD}

extract() {
  sed '1,/^__tgz__/d' "$0"| (cd "$shz_target_dir" && tar xz${v}f - "$@" >&2) || exit 1
}

# Pre-extract script:
[ "$pre" ] && {
  echo "$shz: Extracting/Executing pre-extract script: $pre" >&2
  extract "$pre"
  "$shz_target_dir/$pre" || { echo "$shz: Error: Failed to execute pre-extract script ($pre)" >&2; exit 1; }
}
echo "$shz: Extracting archive directory '$dir' in '$shz_target_dir'" >&2
extract
[ "$cmd" ] || exit 0
cmd="$shz_target_dir/$cmd"
echo "$shz: Executing command: $cmd $*" >&2
[ ! -x "$cmd" ] && echo "$shz: Error: Command $cmd not found or not executable." >&2 && exit 1
"$cmd" "$@"
exit $?

__tgz__ ====== binary data starts here ==========================================```
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
