# Proposed option unification

**GOAL**: short options should be consistent in the _type_ of value they expect. This may not be possible for the entire tools suite, but should at least be achieved for the groups of tools mapping to the command groups in [Part 3 of the TPM spec](https://trustedcomputinggroup.org/wp-content/uploads/TPM-Rev-2.0-Part-3-Commands-01.38.pdf)

## Command groups

* Start-up: tpm2_startup
* Testing: _none implemented_
* Session commands: tpm2_startauthsession, tpm2_policyrestart
* Object commands: tpm2_create, tpm2_load, tpme_loadexternal, tpm2_readpublic, tpm2_activatecredential, tpm2_makecredential, tpm2_unseal
* Duplication commands: tpm2_import
* Asymmetric primitives: tpm2_rsadecrypt, tpm2_rsadecrypt
* Symmetric primitives: tpm2_encryptdecrypt, tpm2_hash, tpm2_hmac
* Random number generator: tpm2_getrandom
* Hash/HMAC/Event sequences: _N/A_
* Attestation commands: tpm2_certify, tpm2_quote
* Ephemeral EC keys: _none implemented_
* Signing and signature verification: tpm2_sign, tpm2_verifysignature
* Command audit: _none implemented_
* Integrity collection (PCR): tpm2_pcrextend, tpm2_pcrevent, tpm2_pcrlist
* Enhanced authorisation (EA): tpm2_policypcr
* Hierarchy commands: tpm2_createprimary, tpm2_clear
* Dictionary attack: tpm2_dictionarylockout
* Misc management: _none implemented_
* Field upgrade: _none implemented_
* Context management: tpm2_flushcontext, tpm2_evictcontrol
* Clocks and timers: _none implemented_
* Capability commands: tpm2_getcap
* Non-volatile storage: tpm2_nvdefine, tpm2_nvlist, tpm2_nvread, tpm2_nvreadlock, tpm2_nvrelease, tpm2_nvwrite

## Tool option proposals

### Common short options

Options which, in their short form, are common across all tools (same short option expects same argument type)

| Short form | Argument type | Details | 
| :---: | --- | --- |
| -h | None | help |
| -Q | None | quiet |
| -T | TCTI options | - |
| -v | None | version |
| -V | None | verbose |
| -Z | None | enable errata fixups |
