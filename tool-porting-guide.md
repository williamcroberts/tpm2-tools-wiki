This page provides useful information to those who wish to assist in porting existing tools over to use the newly added dynamic TCTI infrastructure. This new infrastructure was designed to make porting easy but there are a few things to keep in mind. We'll cover these things here step by step.

##Motivation
In the TPM2 Software Stack (TSS2), TPM2 Command Transmission Interface (TCTI) modules are libraries that provide a standard send / receive interface for transmitting TPM2 command and response buffers. It's a mechanism intended to decouple the command / response transmission mechanism from the libraries that create the command / response buffers. This tight coupling was a significant flaw in the 1.2 TSS.

Till now the tpm2.0-tools were using code copy / pasted from the TPM2.0-TSS test code that set up the socket TCTI only. This TCTI allowed the tools to communicate with the resource manager (RM) from the TPM2.0-TSS and the simulator. Supporting this single TCTI enabled most use cases, but it precludes some use cases like using the tools to send commands directly to the TPM device (`/dev/tpm0`). This may seem like an esoteric use case but many disk encryption utilities are run from the initrd, before the root file system is available and before the RM is running. 

Additionally some new TCTI may become available for a new platform (like the Windows TBS) or as part of a new RM implementation. As such, a mechanism to select which TCTI the tools should use at runtime, as well as the available TCTIs at build time is necessary.

##Requirements
This is a rough list of requirements for the dynamic TCTI infrastructure:

1. The set of supported TCTIs must be selectable at `./configure` time using autoconf `--with-*` options. The configure script must determine whether or not the requested TCTIs are available.
2. The TCTI used by each tool must be configurable using an environment variable, as well as environment variables to configure various TCTI specific parameters.
3. Each tool, at runtime, must accept command line options to allow the user to specify which TCTI the tool will use, as well as any TCTI specific options (like address / port for the socket TCTI). The command line options must override the environment variables.
4. These environment variables and command line options must be documented in a manpage for each tool.

##Infrastructure Overview
The tools currently use common SAPI and TCTI context creation code that's found in the [common.c/h files](https://github.com/01org/tpm2.0-tools/blob/master/src/common.c#L244). Ideally this code would be updated to enable support of dynamic TCTI selection however this existing code is extremely brittle and relies heavily on global data. Additionally the `common.c/h` file has almost no modular separation and so mixing new code into this mess seemed like a bad idea.

###context-util.c/h
To enable the incremental porting of each tool separately we decided to implement a completely new SAPI / TCTI creation and configuration module. As each tool is ported it will gain a dependency on this module which can be found here: https://github.com/01org/tpm2.0-tools/blob/master/src/context-util.h

###options.c/h
This new module is capable of configuring and instantiating any of the supported TCTI modules but it must be provided with a description of the TCTI before it can create it. These options, per the requirements above, must come from the environment or the command line. The `options` module does just this but it brings to light a particular issue: It must be capable of parsing command line options separate from those required by each tool.

The existing tools use `getopt_long` which is a bit unwieldy. `Argparse` would be much better suited to the task with its support for subparsers but porting each tool over to use this new infrastructure and a completely new options processing mechanism at the same time would cause a significant amount of undesirable churn. Though a bit awkward, `getopt` can be made to do what we need and with a minimal impact on the existing tools. Migrating to `argparse` can be done after this effort is complete.

###documentation
Man page fragments for the environment variables and command line options specific to the TCTIs are available in [tcti-options.troff](https://github.com/01org/tpm2.0-tools/blob/master/man/tcti-options.troff) and [tcti-environment.troff](https://github.com/01org/tpm2.0-tools/blob/master/man/tcti-environment.troff). Additionally common options like `--help` and `--verbose` are documented in the [common-options.troff](https://github.com/01org/tpm2.0-tools/blob/master/man/common-options.troff) fragment.

These fragments can be used in the man pages for each tool using the hack in the the Makefile.am that replaces `@COMMON_OPTIONS_INCLUDE@`, `@TCTI_OPTIONS_INCLUDE@` and `@TCTI_ENVIRONMENT_INCLUDE@` strings with the contents of these files. See the generic [rule to build man pages](https://github.com/01org/tpm2.0-tools/blob/master/Makefile.am#L202) in the Makefile.am for the `sed` command that does this replacement. An example man page that uses this mechanism can be found here: https://github.com/01org/tpm2.0-tools/blob/master/man/tpm2_dump_capability.8.in

As each tool is ported over to use this new infrastructure the `--help` option will launch the `man` program to display the man page for the tool executed. This is intended to remove the rather large blocks of text we're embedding in the tool code currently and to reduce duplicate text across the man pages and the tool code.

###supported TCTIs
The TCTIs supported by the tools build can be found by running `./configure --help` and looking for the lines that begin with `--with-tcti-*`. An example output looks like
```
  --with-tcti-device      Build tools with support for the device TCTI.
  --with-tcti-socket      Build tools with support for the socket TCTI.
```
These correspond to the code blocks in `configure.ac` [here](https://github.com/01org/tpm2.0-tools/blob/master/configure.ac#L9) and [here](https://github.com/01org/tpm2.0-tools/blob/master/configure.ac#L27). Enabling / disabling these TCTIs will define (or not) the appropriate HAVE_TCTI_* symbol both in the Makefile.am and in the CFLAGS passed to each compilation unit. This allows the build to selectively enable / disable tools based on the TCTIs available, to enable / disable documentation blocks in the manpages, as well as allowing the C code to include the code blocks required to configure and instantiate the TCTI context.

By default both the socket and device TCTI enabled but only tools that have been ported over to the new infrastructure will be able to use both. The remaining tools are still hard coded to the socket TCTI. Thus if the socket TCTI is *disabled* then the majority of the tools will not be built (and won't be till they're ported over to use this new infrastructure).

##Porting an Existing Tool
When porting a tool over to this new infrastructure there are several steps that should be followed:

1. Replace the tools `main` function with a function named `execute_tool`. The code common to each tool is now factored out into the [main.c/h module](https://github.com/01org/tpm2.0-tools/blob/master/src/main.c) where a common `main` function for all tools now resides. Each tool should instead implement a function called `execute_tool` that is called from `main`. This function should do whatever it is the tool does and then return. After it returns the `main` module will do all necessary tear-down.
2. Remove all duplicate option parsing from the tool code. The options that are parsed by the `options` module include `--help`, `--version` etc. The complete list can be found in the definition of the `option` structure in [options.c](https://github.com/01org/tpm2.0-tools/blob/master/src/options.c#L182).
3. Move the build rule for the tool from the `sbin_PROGRAMS` block within the `HAVE_TCTI_SOCK` ifdef to the main `sbin_PROGRAMS` block above. Also the `main.c` module must be added to the `SOURCES` build variable for the tool. A good example can be found here: https://github.com/01org/tpm2.0-tools/pull/133/commits/c585d803380946b6c32b80706d9636b382363b8f NOTE: update this reference once the PR is merged.
4. Improve the man page for the command. Each command now has a man page thanks to 56be3251567bbb1758325c2d90f07f7be14def9c but some manual intervention is required to improve formatting and content.
5. Last and likely the most difficult is decoupling the dependencies for the tool from the `common` module. Each code module that the tool being ported depends upon must be moved like the tool code itself in #3. Moving these modules requires decoupling them from the old SAPI & TCTI instantiation logic in `common.c`. Ideally once all tools have been ported common.c will have only this code in it and it will be deleted. In the meantime as each tool is ported the `common` code that it requires should be refactored and removed from the `common.c` module. There is no simple way to describe how this should be done and it will require the developers intuition and judgement.