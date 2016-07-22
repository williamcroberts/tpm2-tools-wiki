The tpm2.0-tools manual seems too confusing for people new to TPM 2.0 related areas. This HOW-TO article helps people get to know the basic usage of the tools provided by this project, and so be confident to continue using it.

First of all, let's look at some self-contained tools.

* To list values for all available pcrs:

`$ tpm2_listpcrs`

* To list info about existing NV indices:

`$ tpm2_nvlist`

* To list info about existing persistent objects:

`$ tpm2_listpersistent`

* To get a 20 bytes random number into file random.out:

`$ tpm2_getrandom -s 20 -o random.out`

* To caculate the SHA1 hash value of file data.in and output into hash.out

`$ tpm2_hash -H n -g 0x0004 -I data.in -o hash.out -t tk.out`

Then, let's understand a word 'ownership' before continuing. TPM ownership means the authorization to do things need authentication first. So taking ownership on a TPM means setting authentication secrets into a TPM so that the ownership can be authenticated later via proving the knowledge of the existing authentication secrets.

* To take ownership with "ownerpass" as owner password, "endorsepass" as endorsement password, "lockpass" as lockout password:

`$ tpm2_takeownership -o ownerpass -e endorsepass -l lockpass`

* To change ownership passwords to new ones:

`$ tpm2_takeownership -o ownerpassnew -e endorsepassnew -l lockpassnew -O ownerpass -E endorsepass -L lockpass`

* To set ownership passwords to NULL:

`$ tpm2_takeownership -O ownerpassnew -E endorsepassnew -L lockpassnew`

* To clear ownership(owner related resources will be released, but can only work while platform auth is NULL - BIOS didn't set authentication secrets for platform hierarchy):

`$ tpm2_takeownership -c`

Most tools can run with or without taking ownership, if not taking ownership, skip corresponding parameter when writing command line. Next up, command lines will be given in both cases for each tools.

* Define NV index with index number 0x1500001, size 32 bytes, attribute word 0x2000A and owner autherization:

Ownership not taken:

`$ tpm2_nvdefine -x 0x1500001 -a 0x40000001 -s 32 -t 0x2000A`

Ownership taken:

`$ tpm2_nvdefine -x 0x1500001 -a 0x40000001 -s 32 -t 0x2000A -P ownerpass`

* Write content from file nv.data into NV index 0x1500001, using owner password as authentication

Ownership not taken:

`$ tpm2_nvwrite -x 0x1500001 -a 0x40000001 -f nv.data`

Ownership taken:

`$ tpm2_nvwrite -x 0x1500001 -a 0x40000001 -f nv.data -P ownerpass`

* Read 32 bytes content from NV index 0x1500001, start from offset 0, using owner password as authentication

Ownership not taken:

`$ tpm2_nvread -x 0x1500001 -a 0x40000001 -s 32 -o 0`

Ownership taken:

`$ tpm2_nvread -x 0x1500001 -a 0x40000001 -s 32 -o 0 -P ownerpass`

* Release NV index 0x1500001, using owner password as authentication

Ownership not taken:

`$ tpm2_nvrelease -x 0x1500001 -a 0x40000001`

Ownership taken:

`$ tpm2_nvrelease -x 0x1500001 -a 0x40000001 -P ownerpass`

* Create a Primary Object in endorsement hierarchy, with objectpass as the object password, with RSA keys & SHA256 name hash algorithm, with object context saved in file po.ctx.

Ownership not taken:

`$ tpm2_createprimary -A e -K objectpass -g 0x000b -G 0x0001 -C po.ctx`

Ownership taken:

`$ tpm2_createprimary -A e -K objectpass -g 0x000b -G 0x0001 -C po.ctx -P endorsepass`

* Create a RSA key under the previous primary key, with subobjectpass as the object password, with SHA256 name hash algorithm, with public portion saved in key.pub and private portion saved in key.priv

Parent doesn't have password:

`$ tpm2_create -c po.ctx -K subobjectpass -g 0x000b -G 0x0001 -o key.pub -O key.priv`

Parent has password:

`$ tpm2_create -c po.ctx -P objectpass -K subobjectpass -g 0x000b -G 0x0001 -o key.pub -O key.priv`

* Load the created RSA key

Parent doesn't have password:

`$ tpm2_load -c po.ctx -u key.pub -O key.priv -n key.name -C obj.ctx`

Parent has password:

`$ tpm2_load -c po.ctx -P objectpass -u key.pub -O key.priv -n key.name -C obj.ctx`

* Encrypt with RSA key:

`$ tpm2_rsaencrypt -c obj.ctx -I data.in -o data.encrypted`

* Decrypt with RSA key:

Key doesn't have password:

`$ tpm2_rsadecrypt -c obj.ctx -I data.encrypted -o data.out`

Key has password:

`$ tpm2_rsadecrypt -c obj.ctx -P objectpass -I data.encrypted -o data.out`

* Sign on data with RSA key, using SHA256 as hash algorithm:

Key doesn't have password:

`$ tpm2_sign -c obj.ctx -g 0x000b -m msg.in -s sig.out -t tk.sig`

Key has password:

`$ tpm2_sign -c obj.ctx -P objectpass -g 0x000b -m msg.in -s sig.out -t tk.sig`

* Verify signature with RSA key:

`$ tpm2_verifysignature -c obj.ctx -g 0x000b -m msg.in -s sig.out -t tk.sig`

(To be continued with command lines for some real usage cases.)