# mkshz
self extracting shell gzip tarball creator  
Simple shell script to generate self extracting shell script with embedded gzipped tarball

# usage
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
