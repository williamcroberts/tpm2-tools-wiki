This page is mostly my notes on the dependencies that the tpm2.0-tools repo has on the TPM2.0-TSS source tree. Sorting out the use of the SAPI / TCTI libraries is pretty trivial. That's under review now and will hopefully be merged soon. The hard part is addressing the bits of the build that directly build source from the TSS tree and link it into an executable. This page will track the files in the TSS tree, what we're using from these source files, and why we need that code. The end goal is sort out a path forward that will decouple these two source trees.

## debug.(c|h)
The debug source file is an interesting one. The tpm2.0-tools barely use it and most of the uses are in dead code. The unfortunate bit is that the other source files that we pull in from the TSS repo are heavily dependent on the debug file. Likely this file will remain a dependency till we kill off other dependencies.

Function | Consumer | Notes
---------|----------|------
PrintRMDebugPrefix | common.cpp, TpmClientPrintf | Function to print a debug level prefix specific to the resource manager. This further implies that we're using debug levels and the constants that define them from the resource manager code. The call to this function is in code that's removed by preprocessor macros in the #if 1 / #else style. This is a common way to switch out code blocks during development / test / debug cycles. The function where this call is made is dead code. It currently is removed by a macro, its replacement returns just 0, and it's only called from 3 places in all of the tools code.
DebugPrintBuffer | not used |
DebugPrintBufferOpen | note used |
OpenOutFile | common.cpp, TpmClientPrintf | Function to get a handle to an output file presumably for writing debug messages. It does a bit of magic to maintain a single open file handle. There are multiple definitions of this function controlled by the SHARED_OUT_FILE macro that we do not define in the build AFAIK. When this macro is not defined the function just returns stdout. This behavior seems to be what we want for command line tools so removing our dependency on this function should be simple.
CloseOutFile | common.cpp, TpmClientPrintf | Same as OpenOutfile. Used in dead code.
printfFunction | not used | This is a function pointer that's actually defined elsewhere (extern).

That's pretty much it. Everything else is function pointers extern'd.