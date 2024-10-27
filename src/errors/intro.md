# Error Codes

This page lists the error codes that `rustic` and `rustic_core` can return. The
error codes are grouped by the module that returns them.

## `rustic_core`

- C001 (Cryptography): Data decryption failed, MAC check failed.
- C002 (Password): The password that has been entered, seems to be incorrect. No
  suitable key found for the given password. Please check your password and try
  again.
- C003 (Verification): Verification failed: After decrypting and decompressing
