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
The easiest way to explain how to port an existing tool to use this infrastructure is to walk through an example. The `tpm2_createprimary` and `tpm2_create` tools have already been ported and so we'll use the commits associated with these ports to demonstrate. The commits associated generally fall into 3 categories:
1. Tool code changes.
2. Build changes.
3. Refactoring code in the common module.

###Example: tpm2_createprimary
Personally I've included changes from category 3 into separate commits, and those from 1 and 2 into a single commit. Using `tpm2_createprimary` as an example, the associated commits broken down into category 1/2 is in [1627714b83e0246b6e62364094ec3168d812b21a](https://github.com/01org/tpm2.0-tools/commit/1627714b83e0246b6e62364094ec3168d812b21a), while [2916fba202b559d7ff654b73191f0cd2fc878bb1](https://github.com/01org/tpm2.0-tools/commit/2916fba202b559d7ff654b73191f0cd2fc878bb1), 
[037d4817ab100017ff34471efe0df94db669b6b3](https://github.com/01org/tpm2.0-tools/commit/037d4817ab100017ff34471efe0df94db669b6b3) and [825c156af7c906564a3925e165d9f5d5878cfafa](https://github.com/01org/tpm2.0-tools/commit/825c156af7c906564a3925e165d9f5d5878cfafa) fall into category 3. Let's go through the common module refactoring first.

###Common Module Refactoring
The `common.c/h` is exactly what the name says: common functions. 'common' is a pretty big bucket though and it's full of largely unrelated stuff. The `tpm2_createprimary` tool does 3 things:

1. Assembles command line parameters and transforms them into some form needed by the tool.
2. Executes a TPM2 command (`TPM2_CreatePrimary` to be precise).
3. Saves some data to disk (the context returned by the call to `TPM2_CreatePrimary`).

Of these functions, those that are in the `common` module generally fall into the first and third group. The global data associated with this context management in the legacy code makes moving individual tools over to use the new infrastructure without making structural changes to each tool (generally this means breaking their dependence on the global state). Instead it's easier to move code out of the common module and then have each individual tool come up with it's own scheme for breaking its dependence on the global state that's no longer available.

Using this approach the `common` module will eventually contain only the legacy context management code. When the last tool is ported this module can then just be deleted. Till then as each tool is ported, the dependencies it has in the `common` module must be factored out into separate modules. For `tpm2_createprimary` I factored these functions into two new modules in two commits:

1. [2916fba202b559d7ff654b73191f0cd2fc878bb1](https://github.com/01org/tpm2.0-tools/commit/2916fba202b559d7ff654b73191f0cd2fc878bb1) for functions manipulating files
2. [037d4817ab100017ff34471efe0df94db669b6b3](https://github.com/01org/tpm2.0-tools/commit/037d4817ab100017ff34471efe0df94db669b6b3) for functions converting strings into binary representations

Once these functions were moved into separate modules these files can be built separate from the `common` module and so they can be built regardless of which TCTI is configured. Their implementation files are then included in the `COMMON_SRC` variable outside of the `ifdef HAVE_TCTI_SOCKET` block. Look at the change to `Makefile.am` [here](https://github.com/01org/tpm2.0-tools/commit/037d4817ab100017ff34471efe0df94db669b6b3#diff-c949f93d03f44a4217d7a138f9e2e54a). This block is necessary due to some bizarre dependencies the `common` module has on symbols leaked by libtcti-socket (that I won't get into here).

It's also very possible that a function from the common module should be factored out into an *existing* module. The two examples above discuss creating new code modules but it's increasingly likely that the code you need to remove from `common` is closely related to an existing module and should be moved there. Keep in mind that this is preferable to creating / adding a new source file. A good example is [825c156af7c906564a3925e165d9f5d5878cfafa](https://github.com/01org/tpm2.0-tools/commit/825c156af7c906564a3925e165d9f5d5878cfafa).

###Build Changes
I recommend thinking of the changes to the build being our real end goal in this exercise. It sounds weird but in the end we want to successfully move the source file for our tool `$(SRCDIR)/src/tpm2_createprimary.c` and move it out of the `ifdef HAVE_TCTI_SOCKET`. The refactoring exercise in the previous section was required to break the dependency on the `common` module which cannot be built without the socket TCTI. So one way to figure out everything that will be required to port a tool would be to start with these build changes and then debug the various compiler / linker errors as they happen.

The relevant commit for our example is [here](https://github.com/01org/tpm2.0-tools/commit/1627714b83e0246b6e62364094ec3168d812b21a#diff-c949f93d03f44a4217d7a138f9e2e54a).

###Tool Code Changes
Each tool is different so there's no silver bullet here. Each however will need to at the very least remove their processing of TCTI and common options. This is stuff like `--version`, `--help`, `--port`, and `debug`. Additionally most of the tools (all?) depend on a few global symbols from the `common` module. Mostly this is stuff like the SAPI context object. The diff for this change in `tpm2_createprimary` are [here](https://github.com/01org/tpm2.0-tools/commit/1627714b83e0246b6e62364094ec3168d812b21a#diff-20e0543f918c01063f818b4692c0cda9).

##Conclusion
That's about it. If you find bugs in this doc while porting a tool then place file a bug against the documentation, and maybe the tool in question too.