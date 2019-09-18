# Breakpad Tools

Builds Breakpad and creates artifacts on different operating systems. To use it,
clone on a compatible system, go to the folder of the respective operating
system, and run `make all` or run one of the targets below.

1. Clone and Build
```sh
git clone --recursive https://github.com/getsentry/breakpad-tools
cd breakpad-tools/linux && make all
or
cd breakpad-tools/macos && make all
```

2. Produce a crash
```
cd build
./crash
// outputs a .dmp file (minidump)
```

3. Upload Debug Files

4. Upload the Minidump

## Download

The latest builds can always be downloaded here:

* [Download linux archive](https://s3.amazonaws.com/getsentry-builds/getsentry/breakpad-tools/breakpad-tools-linux.zip)
* [Download macOS archive](https://s3.amazonaws.com/getsentry-builds/getsentry/breakpad-tools/breakpad-tools-macos.zip)
* [Download windows archive](https://s3.amazonaws.com/getsentry-builds/getsentry/breakpad-tools/windows/breakpad-tools-windows.zip)

## Contents

Each platform build folder contains the following artifacts:

**Libraries**

* `libclient.a`: Static library containing the breakpad `ExceptionHandler`<br>
  _target: `make client`_

**Tools**

* `dump_syms`: A tool to create breakpad symbols<br>
  _target: `make dump_syms`_
* `minidump_stackwalk`: A tool to process minidumps<br>
  _target: `make minidump_stackwalk`_

**Examples**

* `crash`: A program that crashes and generates a minidump<br>
  _target: `make crash`_
* `crash.sym`: Breakpad symbols for the crashing program<br>
  _target: `make symbols`_
* `mini.dmp`: A crash dump of the `crash` executable<br>
  _target: `make minidump`_
* `symbols/`: Symbol folder structure required by the processor<br>
  _target: `make dist` (requires all other targets)_

_**Please note** that the examples are always built from scratch, so UUIDs will
change!_

## Build Process

The breakpad libraries and tools are built with custom makefiles. Each OS folder
contains a slightly different version customized to the platform. Breakpad has
broken their own build files (especially on macOS) quite frequently, so this is
the most stable approach.

The Windows build uses `msbuild` instead of a makefile. It currently does not
generate the client library and symbols folders.

The breakpad submodule has been updated last on `2018-01-09`. Future updates
might require changes to the makefiles.


---------------------------------


1. download release  
go to https://github.com/getsentry/sentrypad/releases  
click `sentry-native-all.zip`, not Source code (zip) or Source code (tar.gz)

2. open downloaded release in terminal  
`cd $HOME/<path_to>/sentry-native` is wherever you unzipped the downloaded release to

3. upload dsym files (update org and project slug in CLI example)  
sentry-cli upload-dif -t dsym --no-bin .  --org testorg-az --project minidump-mac

```
# uses breakpad-app on sentry.io
sentry-cli upload-dif -t dsym --no-bin . --org testorg-az --project breakpad-app

# output is:
➜  sentry-native sentry-cli upload-dif -t dsym --no-bin . --org testorg-az --project breakpad-app
> Found 3 debug information files
> Prepared debug information files for upload
> Uploaded 3 missing debug information files
> File processing complete:

     OK fabe80cd-83a7-b83c-1a0d-5ff00aa22f22 (crashInMain; x86 executable)
     OK 047f1eb1-7c9e-d4b6-9b86-2b709398380f (crashduringload; x86 executable)
     OK 94bf873c-47a7-3bc0-7125-291390b4c5f1 (dump_syms_dwarf_data; x86 debug companion)
```

4. verify that dsym files have been uploaded (update org and project slug in URL example)  
go to https://sentry.io/settings/testorg-az/projects/breakpad-app/debug-symbols/

5. generate new minidump
```
./crash

# creates 050F6967-CA42-4473-A47E-D233742B5184.dmp in /build
```
6. upload minidump (update URL and filename in curl example)  
(curl -X POST 'https://sentry.io/api/1317411/minidump/?sentry_key=bb839727976a4b1c981f1de7a9232188'  -F upload_file_minidump=@mini.dmp)

```
cd $HOME/getsentry/breakpad-tools/macos/build

# uses breakpad-app on sentry.io
curl -X POST 'https://sentry.io/api/1730077/minidump/?sentry_key=77783407f1a341dfa70ffb8827c772e2' -F 050F6967-CA42-4473-A47E-D233742B5184.dmp=@mini.dmp
```



7. verify that minidump was received by Sentry  
?
8. verify that minidump was processed without any errors  
?



--------------------------------------------------------------


```
# IGNORE - is for attaching to an event
curl -X POST \
  'https://sentry.io/api/testorg-az/hardware-store-express/events/358b4cb012324028b8173a70848a3577/attachments/?sentry_key=772d35e856764ce9af8129e95cbd7ab2' \
  -F upload_file_minidump=@mini.dmp
```