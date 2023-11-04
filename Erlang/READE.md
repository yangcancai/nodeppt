## issue
```shell
opensslErrorStack: [ 'error:03000086:digital envelope routines::initialization error' ],
  library: 'digital envelope routines',
  reason: 'unsupported',
  code: 'ERR_OSSL_EVP_UNSUPPORTED'
```
### how to fix it?
```shell
export NODE_OPTIONS=--openssl-legacy-provider
```