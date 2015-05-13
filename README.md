zf_log
========

### Tiny logging library for C (and C++)

This is just a thin wrapper around sprintf() function. It provides less than 20%
of functionality found in more sophisticated libraries, but covers more than 80%
of common use cases. It could be particularly useful in mobile/embedded
applications, early stages of development or when prototyping. Focus is made on
simplicity, ease of use and performance (to be more precise - low overhead).

Features:

* Debug logging is reduced to no-op in release builds
* Arguments are not evaluated when the message is not logged
* No "unused" warning for variables used in log statements only
* Dump a memory region as HEX and ASCII
* Optional built-in support for Android log and Apple log (iOS, OSX)
* Custom output functions
* All externally visible symbols can be prefixed with provided string (useful in
  libraries that don't want to expose dependency on zf_log)

Examples
--------

Library provides a set of `ZF_LOGX` macros where `X` is an abbreviated log
level (e.g. `I` for `INFO`). This code will log an `INFO` message:

```c
ZF_LOGI("Number of arguments: %i", argc);
```

And will produce the following log line if `NDEBUG` is defined (aka release
build):

```
+- month           +- process id
|  +- day          |      +- thread id      +- message
|  |               |      |                 |
04-29 22:43:20.244 40059  1299 I hello.MAIN Number of arguments: 1
      |  |  |  |               | |     |
      |  |  |  +- millisecond  | |     +- tag (optional)
      |  |  +- second          | +- tag prefix (optional)
      |  +- minute             +- level
      +- hour
```

And if `NDEBUG` is NOT defined (aka debug build):

```
04-29 22:43:20.244 40059  1299 I hello.MAIN main@hello.c:9 Number of arguments: 1
                                            |    |       |
                                            |    |       +- line number
                                            |    +- source file name
                                            +- function name
```

It's also possible to dump a memory region. For example:

```c
const char data[] = "Lorem ipsum dolor sit amet, consectetur adipiscing elit.";
ZF_LOGI_MEM(data, sizeof(data), "Lorem ipsum at %p", data);
```

Will produce following output:

```
05-06 00:54:33.825 35864  1299 I hello.MAIN Lorem ipsum at 0x10fbc0f20:
05-06 00:54:33.825 35864  1299 I hello.MAIN 4c6f72656d20697073756d20646f6c6f  Lorem ipsum dolo
05-06 00:54:33.825 35864  1299 I hello.MAIN 722073697420616d65742c20636f6e73  r sit amet, cons
05-06 00:54:33.825 35864  1299 I hello.MAIN 65637465747572206164697069736369  ectetur adipisci
05-06 00:54:33.825 35864  1299 I hello.MAIN 6e6720656c69742e00                ng elit.?
```

Usage
--------

### Building

To build shared library:
```bash
cmake -DBUILD_SHARED_LIBS:BOOL=ON
```

Performance
--------

Performance comes with a caveat: it applies only to the case when the message
was not logged. Log statements that are below *current log level* (compile time
check) have no overhead - they are compiled out and arguments will not be
evaluated. Log statements that are below *output log level* (run time check)
have a small overhead of compare operaion and conditional jump. Arguments will
not be evaluated and no function call will be performed. But it makes little
sense to talk about performance when the log statement actually writes
something to the log. In that case the library only tries to minimize the size
of code that will be generated for each log statement (compare operation,
arguments evaluation and a function call).

Output
--------

By default log messages are written to the stderr, but it is also possible to
set custom output function. Library has an optional built-in support for the
following output facilities:
* Android Log (via android/log.h)
* Apple System Log (iOS, OSX via asl.h)


Why zf?
--------

It stands for Ze Foundation. "Ze" is like "The" but with french or german accent.
Mostly because zf_anything looks better than tf_anything.
