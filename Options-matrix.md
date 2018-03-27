**NOTE**: This is a work in progress, please see issue [#957](https://github.com/tpm2-software/tpm2-tools/issues/957) to track status.

***
| short form | long form | used by | argument type | canonical |
|------------|-----------|---------|---------------|-----------|
| -a | --attest-file | tpm2_certify | file path | |
| -a | --auth-policy-session | tpm2_createpolicy | TPM session policy | |
| -c | --context | tpm2_evictcontrol, tpm2_createek | file path | |
| -c | --context-parent | tpm2_create | file path | |
| -c | --key-context | tpm2_certify, tpm2_encryptdecrypt, tpm2_hmac | file path | |
| -c | --clear | tpm2_clearlock | NA | |
| -c | --clear-lockout | tpm2_dictionarylockout | NA | |
| -e | --endorse-password | tpm2_activatecredential, tpm2_changeauth | password | |
| -e | --endorse-passwd | tpm2_getmanufec, tpm2_createak, tpm2_createek | password | |
| -f | --in-file | tpm2_activatecredential, tpm2_certify | file path | |
| -f | --policy-file | tpm2_createpolicy | file path | |
| -f | --format | tpm2_certify | signature format | |
| -f | --out-file | tpm2_getmanufec | file path | |
| -g | --halg | tpm2_create, tpm2_createprimary, tpm2_hash, tpm2_hmac | hash algorithm | |
| -g | --policy-digest-alg | tpm2_createpolicy | hash algorithm | |
| -g | --algorithm | tpm2_getmanufec, tpm2_createak, tpm2_createek | algorithm specifier | |
| -k | --key-handle | tpm2_activatecredential, tpm2_encryptdecrypt, tpm2_hmac | hex handle id | |
| -k | --ak-handle | tpm2_createak | hex handle id | |
| -l | --lockout-passwd | tpm2_clear | password | |
| -l | --loaded-session | tpm2_flushcontext | N/A | |
| -l | --list | tpm2_getcap | N/A | |
| -o | --out-file | tpm2_activatecredential, tpm2_getrandom, tpm2_hash, tpm2_hmac |  file path | |
| -o | --owner-passwd | tpm2_changeauth, tpm2_getmanufec, tpm2_createak, tpm2_createek | password | |
| -p | --persistent | tpm2_evictcontrol | hex handle id | |
| -p | --platform | tpm2_clear, tpm2_clearlock | N/A | |
| -p | --file | tpm2_createek | file path | |
| -r | --privfile | tpm2_create | file path | |
| -s | --saved-session | tpm2_flushcontext | N/A | |
| -s | --sig-file | tpm2_certify | file path | |
| -s | --setup-parameters | tpm2_dictionarylockout | N/A | |
| -t | --transient-object | tpm2_flushcontext | N/A | |
| -t | --ticket | tpm2_hash | file path | N/A |
| -u | --pubfile | tpm2_create | file path | |
| -x | --pcr-index | - | - | - | goal |
| -A | --auth | tpm2_evictcontrol | character representing hierarchy (o or p) | |
| -A | --object-attributes | tpm2_create, tpm2_createprimary | object attributes | |
| -C | --key-context | tpm2_activatecredential | file path | |
| -C | --obj-context | tpm2_certify | file path | |
| -C | --context | tpm2_createprimary | file path | |
| -C | --capability | tpm2_getcap | capability name | |
| -D | --decrypt | tpm2_encryptdecrypt | NA | |
| -E | --old-endorse-passwd | tpm2_changeauth | password | |
| -E | --ec-cert | tpm2_getmanufec | file path | |
| -E | --ek-handle | tpm2_getcreateak | hex handle id | |
| -F | --pcr-input-file | tpm2_createpolicy | file path | |
| -G | --kalg | tpm2_create, tpm2_createprimary | key algorithm | |
| -H | --handle | tpm2_evictcontrol, tpm2_activatecredential, tpm2_flushcontext, tpm2_getmanufec, tpm2_createek | hexadecimal handle id | |
| -H | --obj-handle | tpm2_certify | hex handle id | |
| -H | --parent | tpm2_create | hex handle id | |
| -H | --hierarchy | tpm2_createprimary, tpm2_hash | char (hierarchy pretty name) | |
| -I | --in-file | tpm2_create, tpm2_encryptdecrypt | file path | |
| -K | --pwdk | tpm2_certify, tpm2_create, tpm2_createprimary | password | |
| -L | --lockout-passwd | tpm2_clear, tpm2_clearlock | password | |
| -L | --policy-file | tpm2_create, tpm2_createprimary | file path | |
| -L | --old-lockout-passwd | tpm2_changeauth | password | |
| -L | --set-list | tpm2_createpolicy | (string) list of PCR IDs | |
| -N | --non-persistent | tpm2_getmanufec | N/A | |
| -O | --old-owner-passwd | tpm2_changeauth | password | |
| -O | --offline | tpm2_getmanufec | file path | |
| -P | --auth-XXX | - | - | - | goal |
| -P | --pwda | tpm2_evictcontrol | password | |
| -P | --pwdo | tpm2_certify | password | |
| -P | --pwdp | tpm2_create, tpm2_createprimary | password | |
| -P | --password | tpm2_activatecredential | password | |
| -P | --pwdk | tpm2_encryptdecrypt, tpm2_hmac | password | |
| -P | --ek-passwd | tpm2_getmanufec | password | |
| -P | --eKPasswd | tpm2_createek | password | |
| -P | --ak-passwd | tpm2_createak | password | |
| -P | --policy-pcr | tpm2_createpolicy | N/A | |
| -S | --session | tpm2_evictcontrol, tpm2_create, tpm2_createprimary, tpm2_encryptdecrypt, tpm2_flushcontext, tpm2_getmanufec, tpm2_createek, tpm2_hmac | file path | |
| -U | --SSL_NO_VERIFY | tpm2_getmanufec | N/A | |
