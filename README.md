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


------------------------------------------------------------------------------------------------------
TODO: mention that you need `sentry-cli`? Show output of curl -x Post (uploading the minidump)?
TODO: `make all` is failing:
```
➜  macos git:(master) pwd
/Users/wcap/getsentry/breakpad-tools/macos
➜  macos git:(master) make all
make: *** No rule to make target `build/deps/breakpad/src/client/minidump_file_writer.o', needed by `build/libclient.a'.  Stop.
```


# Will's README
## Setup
### Compile the 'crash' executable for producing a minidump
1. 
```
git clone git@github.com:getsentry/breakpad-tools.git

# <macos> | <linux> | <windows>
cd breakpad-tools/macos
make all

# note, the above command currently is failing, see next section for fix
```

### Obtain debug symbols and upload them (a.k.a. DSYM or debug information files)
> Note - Sentry Native is a wrapper around two most popular crash-reporting frameworks: Breakpad and Crashpad

3. Go to https://github.com/getsentry/sentry-native/releases and download the latest `sentry-native-all.zip`  

4. cd into extracted zip where the DSYM files are

5. Upload DSYM files (update org and project slug in CLI example)  
`sentry-cli upload-dif -t dsym --no-bin . --org <sentry_org_name> --project <sentry_project_name>`  
example:  
```
## FUTURE 
## 09/19 IS BREAKING symbolication
## sentry-cli upload-dif -—include-sources -t dsym --no-bin . --org testorg-az --project breakpad-app 

sentry-cli upload-dif -t dsym --no-bin . --org testorg-az --project breakpad-app

> Found 3 debug information files
> Prepared debug information files for upload
> Uploaded 3 missing debug information files
> File processing complete:
     OK fabe80cd-83a7-b83c-1a0d-5ff00aa22f22 (crashInMain; x86 executable)
     OK 047f1eb1-7c9e-d4b6-9b86-2b709398380f (crashduringload; x86 executable)
     OK 94bf873c-47a7-3bc0-7125-291390b4c5f1 (dump_syms_dwarf_data; x86 debug companion)
```
6. Verify that dsym files have been uploaded (update org and project slug in URL example)  
Go to https://sentry.io/settings/testorg-az/projects/breakpad-app/debug-symbols/

### Produce & Upload a Minidump
Use this if `make all` in previous step failed:
1. Produce minidump  
- [Download macOS archive](https://s3.amazonaws.com/getsentry-builds/getsentry/breakpad-tools/breakpad-tools-macos.zip) it unzips as `breakpad-tools-macos`
```
cd breakpad-tools-macos
./crash

# Check if a file like 050F6967-CA42-4473-A47E-D233742B5184.dmp was created
ls
050F6967-CA42-4473-A47E-D233742B5184.dmp
```  


2. Upload minidump  
`curl -X POST 'https://sentry.io/api/1317411/minidump/?sentry_key=bb839727976a4b1c981f1de7a9232188'  -F upload_file_minidump=@<your_minidump_file.dmp>`  
example:  
```
cd breakpad-tools/macos/build
curl -X POST 'https://sentry.io/api/1730077/minidump/?sentry_key=77783407f1a341dfa70ffb8827c772e2' -F upload_file_minidump=@050F6967-CA42-4473-A47E-D233742B5184.dmp
```

3. Verify that minidump was received by Sentry  
Go to sentry.io....Events...(image of events stream)
4. Verify that minidump was processed without any errors  
Go to sentry.io....Event...(image of it?)



