**NOTE**: This is a work in progress, please see issue [#957](https://github.com/tpm2-software/tpm2-tools/issues/957) to track status.

***
| short form | long form | used by | argument type | |
|------------|-----------|---------|---------------|-|
| -a | --attest-file | tpm2_certify | file path | |
| -c | --context | tpm2_evictcontrol | file path | |
| -c | --context-parent | tpm2_create | file path | |
| -c | --key-context | tpm2_certify | file path | |
| -c | --clear | tpm2_clearlock | NA | |
| -e | --endorse-password | tpm2_activatecredential, tpm2_changeauth | password | |
| -f | --in-file | tpm2_activatecredential, tpm2_certify | file path | |
| -f | --format | tpm2_certify | signature format | |
| -g | --halg | tpm2_create | hash algorithm | |
| -k | --key-handle | tpm2_activatecredential | hex handle id | |
| -l | --lockout-passwd | tpm2_clear | password | |
| -o | --out-file | tpm2_activatecredential |  file path | |
| -o | --owner-passwd | tpm2_changeauth | password | |
| -p | --persistent | tpm2_evictcontrol | hex handle id | |
| -p | --platform | tpm2_clear, tpm2_clearlock | N/A | |
| -r | --privfile | tpm2_create | file path | |
| -s | --sig-file | tpm2_certify | file path | |
| -u | --pubfile | tpm2_create | file path | |
| -x | --pcr-index | - | - | - | ☺ |
| -A | --auth | tpm2_evictcontrol | character representing hierarchy (o or p) | |
| -A | --object-attributes | tpm2_create | object attributes | |
| -C | --key-context | tpm2_activatecredential | file path | |
| -C | --obj-context | tpm2_certify | file path | |
| -E | --old-endorse-passwd | tpm2_changeauth | password | |
| -G | --kalg | tpm2_create | key algorithm | |
| -H | --handle | tpm2_evictcontrol, tpm2_activatecredential | hexadecimal handle id | |
| -H | --obj-handle | tpm2_certify | hex handle id | |
| -H | --parent | tpm2_create | hex handle id | |
| -I | --in-file | tpm2_create | file path | |
| -K | --pwdk | tpm2_certify, tpm2_create | password | |
| -L | --lockout-passwd | tpm2_clear, tpm2_clearlock | password | |
| -L | --policy-file | tpm2_create | file path | |
| -L | --old-lockout-passwd | tpm2_changeauth | password | |
| -O | --old-owner-passwd | tpm2_changeauth | password | |
| -P | --auth-XXX | - | - | - | ☺ |
| -P | --pwda | tpm2_evictcontrol | password | |
| -P | --pwdo | tpm2_certify | password | |
| -P | --pwdp | tpm2_create | password | |
| -P | --password | tpm2_activatecredential | password | |
| -S | --session | tpm2_evictcontrol, tpm2_create | file path | |
