---
title: PHP
parent: Useful Commands
nav_order: 1
---

# Useful Commands: PHP

## Logging Stacktrace
Log up to the last 20 stacktraces for review.
```php
error_log( print_r( debug_backtrace( DEBUG_BACKTRACE_IGNORE_ARGS, 20 ), true ) );
```