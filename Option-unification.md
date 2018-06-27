# Proposed option unification

**GOAL**: short options should be consistent in the _type_ of value they expect. This may not be possible for the entire tools suite, but should at least be achieved for the groups of tools mapping to the command groups in [Part 3 of the TPM spec](https://trustedcomputinggroup.org/wp-content/uploads/TPM-Rev-2.0-Part-3-Commands-01.38.pdf)
> This work is currently in progress

## Current options

The options as of ~3.0.3+ / commit [60768ba](https://github.com/tpm2-software/tpm2-tools/commit/60768ba73043bdf68311047bdfd23c9e89ba16cf) are documented in a separate wiki page: [[3.0.x-options-matrix]].

## Command groups

Short options should be common in their argument values and purpose across all tools in a group.

* Start-up: tpm2_startup
* Session: tpm2_startauthsession, tpm2_policyrestart
* Object: tpm2_create, tpm2_createprimary, tpm2_changeauth, tpm2_load, tpm2_loadexternal, tpm2_readpublic, tpm2_activatecredential, tpm2_makecredential, tpm2_unseal
* Duplication: tpm2_import
* Asymmetric primitives: tpm2_rsaencrypt, tpm2_rsadecrypt
* Symmetric primitives: tpm2_encryptdecrypt, tpm2_hash, tpm2_hmac
* Random number generator: tpm2_getrandom
* Attestation: tpm2_certify, tpm2_quote
* Signing and signature verification: tpm2_sign, tpm2_verifysignature
* Integrity: tpm2_pcrextend, tpm2_pcrevent, tpm2_pcrlist
* Enhanced authorisation (EA): tpm2_policypcr, tpm2_createpolicy
* Hierarchy commands: tpm2_clear, tpm2_clearlock
* Dictionary attack: tpm2_dictionarylockout
* Context management: tpm2_flushcontext, tpm2_evictcontrol
* Capability commands: tpm2_getcap
* Non-volatile storage: tpm2_nvdefine, tpm2_nvlist, tpm2_nvread, tpm2_nvreadlock, tpm2_nvrelease, tpm2_nvwrite

## Common tool options

### Standard options

Options which, in their short form, are common across all tools (same short option expects same argument type). These remain the same as 3.0.x versions of the tools.

| Short form | Argument type | Details | 
| :---: | --- | --- |
| -h | None | help |
| -Q | None | quiet |
| -T | TCTI options | - |
| -v | None | version |
| -V | None | verbose |
| -Z | None | enable errata fixups |

### Parent and current object

When tools take a parent and/or current object we can simplify the options by having **-C** always be the parent object and **-c** the child object.

The method `tpm2_util_object_load` is used to parse an option argument value string and populate a `tpm2_loaded_object` with a parsed path and/or handle. The argument value string will recognised as a context file when prefixed with "*file:*" or should the value not be parsable as a handle number (as understood by `strtoul()`). For example:
```
0x1234 - specifies a handle
foo.dat - specifies a context file
file:0x1234 - specifies a context file
```

| Short form | Argument type | Details | 
| :---: | --- | --- |
| -c | handle id or context file path | child/current object |
| -C | handle id or context file path | parent object |

### Object authorization

When objects require an authorization value we use **-P** to supply the authorization value for any parent object and **-p** to supply the authorization value for the current/child object.

| Short form | Argument type | Details | 
| :---: | --- | --- |
| -p | authorization value | authorization for child/current object |
| -P | authorization value | authorization for parent object |

### Algorithm

When tools take algorithm specifiers as arguments the **-g** and **-G** short options will be used.**-g** should always specify the name algorithm whilst **-G** should be object/type algorithm.

| Short form | Argument type | Details | 
| :---: | --- | --- |
| -g | algorithm type specifier (see Algorithm Specifiers) | name alg |
| -G | algorithm type specifier (see Algorithm Specifiers) | object type/alg |