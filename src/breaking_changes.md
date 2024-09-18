# Breaking Changes

This document lists all user facing breaking changes in `rustic` and provides
guidance on how to migrate from one version to another.

## 0.9.0

### Configuration File

Consistently apply singular and plural naming conventions for keys in the
configuration file that accept an array of values. This change affects the
following keys:

- glob -> globs
- iglob -> iglobs
- glob_file -> glob_files
- iglob_file -> iglob_files
- custom_ignorefile -> custom_ignorefiles
- tag -> tags

You need to update your configuration file accordingly, changing the key names
from singular to plural.
