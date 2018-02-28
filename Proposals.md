# Error Library

## Reason
Decoding and handling TSS2_RC codes is complex, especially when considering layer 0 or TPM2_RC codes.
Applications need a way to easily decode raw hex RCs in a more human digestible way.

LibC has strerror(), to make this easier. Build something like this for TSS2_RC codes, that can handle
custom layer handlers. This way applications can provide useful error messages to users.

## The major parts
There are two major parts to this:
1. Setting/clearing a new error layer handler
2. Decoding an error string

## Setting or clearing an error layer handler

Below describes the two parts of the interface for
registering a new layer handler. It consists
of a function where you register a new handler and
a protoype for that function.

```C
/**
 * Register or unregister a custom layer error handler.
 * @param layer
 *  The layer in which to register a handler for. It is an error
 *  to register for the following reserved layers:
 *    - TSS2_TPM_RC_LAYER  - layer  0
 * @param name
 *  A friendly layer name. It is an error for the name to be of
 *  length 0 or greater than 4.
 * @param handler
 *  The handler function to register or NULL to unregister.
 * @return
 *  True on success or False on error.
 */
bool tpm2_error_set_handler(UINT8 layer, const char *name,
        tpm2_error_handler handler);


/**
 * A custom error handler prototype.
 * @param rc
 *  The rc to decode with only the error bits set, ie no need to mask the
 *  layer bits out. Handlers will never be invoked with the error bits set
 *  to 0, as zero always indicates success.
 * @return
 *  An error string describing the rc. If the handler cannot determine
 *  a valid response, it can return NULL indicating that the framework
 *  should just print the raw hexidecimal value of the error field of
 *  a tpm2_err_layer_rc.
 *  Note that this WILL NOT BE FREED by the caller,
 *  i.e. static.
 */
typedef const char *(*tpm2_error_handler)(TSS2_RC rc);
```

### Error Decoding

The next portion, describes the interface for decoding a TSS2_RC rc
```C
/**
 * Given a TSS2_RC return code, provides a static error string in the format:
 * <layer-name>:<layer-specific-msg>.
 *
 * The layer-name section will either be the friendly name, or if no layer
 * handler is registered, the base10 layer number.
 *
 * The "layer-specific-msg" is layer specific and will contain details on the
 * error that occurred or the error code in base16 (hex) if it couldn't look it up.
 *
 * Unknown layer error codes will return the layer name, as it is known, followed by
 * the error code in hex. For example: rm:0xF 
 * Unknown layers will have the layer number in decimal and then a layer specific string of
 * a hex value representing the error code. For example: 9:0x3
 * 
 * Known layer specific substrings:
 * TPM - The tpm layer produces 2 distinct format codes that align with:
 *   - Section 6.6 of: https://trustedcomputinggroup.org/wp-content/uploads/TPM-Rev-2.0-Part-2-Structures-01.38.pdf
 *   - Section 39.4 of: https://trustedcomputinggroup.org/wp-content/uploads/TPM-Rev-2.0-Part-1-Architecture-01.38.pdf
 *
 *   The two formats are format 0 and format 1.
 *     Format 0 string format:
 *       - "<error|warn>(<version>): <description>
 *       - Examples:
 *         - error(1.2): bad tag
 *         - warn(2.0): the 1st handle in the handle area references a transient object or session that is not loaded
 *
 *   Format 1 string format:
 *      - <handle|session|parameter>(<index>):<description>
 *      - Examples:
 *        - handle(unk):value is out of range or is not correct for the context
 *        - tpm:handle(5):value is out of range or is not correct for the context
 *
 *     Note that passing TPM2_RC_SUCCESS results in the layer specific message of "success".
 *
 *   The System, TCTI and Marshaling (MU) layers, all define simple string
 *   returns analogous to strerror(3).
 *
 *
 * @param rc
 *  The error code to decode.
 * @return
 *  A human understandable error description string.
 */
const char *tpm2_error_str(TSS2_RC rc);
```

## Example Code
```C
#include <assert.h>
#include <stdbool.h>
#include <stdlib.h>

#include <sapi/tpm20.h>

#define ARRAY_LEN(x) (sizeof(x)/sizeof(x[0]))

static const char *
custom_err_handler(TSS2_RC rc) {

    static const char *err_map[] = {
        "error 1",
        "error 2",
        "error 3"
    };

    if (rc - 1u >= ARRAY_LEN(err_map)) {
        return NULL;
    }

    return err_map[rc - 1];
}

static int main(int argc, char *argv[]) {

    /* Decoding a known layer, like the tpm */
	const char *e = tpm2_error_str(TPM2_RC_SEQUENCE);
	
	/* prints "tpm:error(2.0): improper use of a sequence handle" */
	printf("%s\n", e);

    /*
     * Dealing with Custom Error Layers
     *
     * Register a custom handler for layer one with a name of "ctsm"
     * Layer 1 RC's will be sent to the function custom_err_handler
     * for decoding.
     */
    bool res = tpm2_error_set_handler(1, "cstm", custom_err_handler);
    assert_true(res);

	/* create a layer specific error code: */
    TSS2_RC rc = TSS2_RC_LAYER(1) | 1;

	/* prints "error 1" */
    e = tpm2_error_str(rc);
    printf("%s\n", e);
    
    /*
     * Test clearing a handler
     */
    tpm2_error_set_handler(1, NULL, NULL);

    /*
     * Test an unknown layer
     */
    e = tpm2_error_str(rc);
	/* prints "1:0x01" */

	return 0;
}
```