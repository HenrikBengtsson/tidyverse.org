---
title: processx 3.2.0
date: 2018-09-06
slug: processx-3.2.0
author: Gábor Csárdi
categories: [package]
tags:
  - processx
  - r-lib
description: >
    Run external processes with processx
photo:
  url: https://unsplash.com/photos/PtgLGdMzi-Y
  author: Adam Sherez
---



## Introduction

We're delighted to announce that
[processx](https://cran.r-project.org/package=processx) in now on CRAN.
processx makes it easy to run external processes from R. It's an extended
version of `system()` and `system2()`, that gives you greater control,
and more visibility into the running process.

It's hard to make processx examples work across platforms because system
utilities vary from OS to OS. To work around this problem, processx bundles
a small program, `px`, which can perform some basic tasks, like printing
to the standard output and error, and waiting for a given amount of time.


```r
px <- processx:::get_tool("px")
px
#> [1] "/Users/gaborcsardi/r_pkgs/processx/bin//px"
```

processx deals with two kinds of external processes: foreground and
background. Foreground processes are synchronous, R waits until they
finish, and collects the output and the exit code of the process.
Background processes are asynchronous, processx does not wait for them
to finish, they run concurrently and can communicate with the R process.


## Foreground processes

`processx::run()` runs a foreground external process. It is somewhat
similar to the `system2()` base R function. Its basic usage is:

```r
processx::run(command, args)
```

`command` is a string (length 1 character vector), and `args` should be a
character vector of arguments. `command` can be an absolute file name, a
relative file name, or a command name. For the latter, the current `PATH`
is used to find the command. For example these both work on Unix systems:

```r
run("/bin/ls")
run("ls")
```

Here is the output of `px --help`:


```r
pxhelp <- run(px, "--help")
cat(pxhelp$stderr)
#> Usage: px [command arg] [command arg] ...
#> 
#> Commands:
#>   sleep  <seconds>           -- sleep for a number os seconds
#>   out    <string>            -- print string to stdout
#>   err    <string>            -- print string to stderr
#>   outln  <string>            -- print string to stdout, add newline
#>   errln  <string>            -- print string to stderr, add newline
#>   cat    <filename>          -- print file to stdout
#>   return <exitcode>          -- return with exitcode
#>   write <fd> <string>        -- write to file descriptor
#>   echo <fd1> <fd2> <nbytes>  -- echo from fd to another fd
#>   getenv <var>               -- environment variable to stdout
```

### Quoting

processx does not use a shell to start up the external process, so special
characters in `command` and `args` need _not_ be shell quoted. This makes it
much easier to support arbitrary file names (that may contain spaces or
special characters) in calls to external programs.


```r
run(px, c("outln", "arg -   with spaces", "outln", "'arg with quote'"))
#> $status
#> [1] 0
#> 
#> $stdout
#> [1] "arg -   with spaces\n'arg with quote'\n"
#> 
#> $stderr
#> [1] ""
#> 
#> $timeout
#> [1] FALSE
```

### Interruption

Unlike `system()` and `system2()`, `processx::run()` is always
interruptible, you can use the usual interruption key, e.g. ESC in RStudio,
or CTRL+C in a terminal. On interruption, the external process is
terminated.

### Spinner

`run()` can show a friendly spinner while the external process is running.
If the process takes longer then a few second, it is a good idea to use it.
The spinner is automatically hidden if R is non-interactive:

```r
run(px, c("sleep", "5"), spinner = TRUE)
```

### Time limit

You can specify a time limit in `run()`, in seconds, or as a `difftime`
object:


```r
run(px, c("sleep", "5"), timeout = 1)
#> Error in run(px, c("sleep", "5"), timeout = 1): System command timeout
```

`run()` throws an error of class `system_command_timeout_error`, so you
can easily catch timeouts using `tryCatch()`, if you wish so. By default
`run()` also throws an error if the system command fails, as indicated by
its exit status.


```r
run(px, c("return", "10"))
#> Error in run(px, c("return", "10")): System command error
```

The `error_on_status` argument can be set to `FALSE` to avoid errors
for non-zero exit statuses. This can be useful if you anticipate a failure
and want to handle it without throwing an R error, or if a non-zero exit
status does not indicate an error for the given program.

### Standard output and error

By default, `run()` collects all standard output and error of the process
and retuns them in two strings. If desired, it can also echo them to
the screen while the external process is running. (They are still collected
and returned, so you can still compute on them.)


```r
outp <- run("ls", "..", echo = TRUE)
#> _redirects
#> articles
#> contribute.md
#> help-is-on-the-way.jpg
#> help.md
#> learn.md
#> lifecycle.md
#> packages.md
#> reprex-addin.png
#> reprex-addins-menu.png
#> rstudio-logo.svg
#> test-ggplot2-1.png
#> test.md
```

### Setting environment variables

You can set environment variables for the external process via the `env`
argument. Usually you want to add these variables to those already set in
the current process, otherwise the external process might fail if some
essential environment variables (like `PATH`) are not set:


```r
run(px, c("getenv", "FOO"), env = c(Sys.getenv(), FOO = "bar"))
#> $status
#> [1] 0
#> 
#> $stdout
#> [1] "bar\n"
#> 
#> $stderr
#> [1] ""
#> 
#> $timeout
#> [1] FALSE
```

## Advanced usage: background processes

processx really shines when it comes to controlling background processes.
To start a backgound process, you create an R6 object of class `process`.
The arguments of `process$new()` mostly correspond to the arguments of
`run()`.


```r
proc <- process$new(px, c("sleep", "10"))
proc
#> PROCESS 'px', running, pid 61100.
```

`process` objects have methods to query process information and to
manipulate the subprocess. See `?process` for a complete list of methods.


```r
proc$get_name()
#> [1] "px"
proc$get_cmdline()
#> [1] "/Users/gaborcsardi/r_pkgs/processx/bin//px"
#> [2] "sleep"                                     
#> [3] "10"
proc$get_exe()
#> [1] "/Users/gaborcsardi/r_pkgs/processx/bin/px"
proc$is_alive()
#> [1] TRUE
proc$suspend()
#> NULL
proc$get_status()
#> [1] "stopped"
proc$resume()
#> NULL
proc$get_status()
#> [1] "running"
proc$kill()
#> [1] TRUE
proc$is_alive()
#> [1] FALSE
proc$get_exit_status()
#> [1] -9
```

### Output and polling

The standard output and standard error of a background process are ignored
by default. To write them to files, set the `stdout` and/or `stderr`
arguments to the paths of the files. Alternatively, processx can create
connections for standard output and error, and R can read from these
connections or poll them. Polling a set of connections or processes means
that R waits until data is available on any of the connections, or a
timeout expires. This is useful if the R process is waiting on one or more
processes.


```r
proc <- process$new(px, c("sleep", "1", "outln", "foo", "sleep", "1",
     "errln", "bar", "sleep", "1"), stdout = "|", "stderr" = "|")
proc$poll_io(-1)
#>   output    error  process 
#>  "ready" "silent" "nopipe"
proc$read_output_lines()
#> [1] "foo"
proc$poll_io(-1)
#>   output    error  process 
#> "silent"  "ready" "nopipe"
proc$read_error_lines()
#> [1] "bar"
proc$poll_io(-1)
#>   output    error  process 
#> "silent"  "ready" "nopipe"
proc$is_alive()
#> [1] FALSE
```

`$poll_io()` also returns when the process terminates.

To poll multiple processes, the non-member `poll()` function can be used,
this takes a list of processes:


```r
proc1 <- process$new(px, c("sleep", "0.5", "outln", "foo1", "sleep", "1"),
     stdout = "|", "stderr" = "|")
proc2 <- process$new(px, c("sleep", "1", "outln", "foo2", "sleep", "1"),
     stdout = "|", "stderr" = "|")
poll(list(proc1, proc2), -1)
#> [[1]]
#>   output    error  process 
#>  "ready" "silent" "nopipe" 
#> 
#> [[2]]
#>   output    error  process 
#> "silent" "silent" "nopipe"
proc1$read_output_lines()
#> [1] "foo1"
poll(list(proc1, proc2), -1)
#> [[1]]
#>   output    error  process 
#> "silent" "silent" "nopipe" 
#> 
#> [[2]]
#>   output    error  process 
#>  "ready" "silent" "nopipe"
proc2$read_output_lines()
#> [1] "foo2"
```

### Process tree cleanup

In addition to terminating the subprocess, processx supports terminating
all child processes that were started by the subprocess, and the child
processes of those, etc.

To request process tree cleanup, set the `cleanup_tree` argument of `run()`
or the `cleanup` argument of `process$new()` to  `TRUE`. (It is the
default for `process$new()`.) To clean up manually, use the `$kill_tree()`
method.

### Use case: wait for an external process to be ready

When starting up an external process, sometimes you need to wait until
the process is ready to receive input. E.g. PhantomJS is a headless
browser, used for testing web applications. The headless browser is
queried and controlled via an HTTP socket. PhantomJS has some startup time,
and to make sure that it is ready for input, you need need to wait until
it logs an INFO line to its standard output:
```
❯ phantomjs  -w
[INFO  - 2018-08-21T19:57:53.957Z] GhostDriver - Main - running on port 8910
^C
```

So processx must capture the standard output and wait until the message
is printed. If the message is not printed within a timeout we throw an
error. On success the function returns the PhantomJS `process` object:


```r
start_program <- function(command, args, message, timeout = 5, ...) {
  timeout <- as.difftime(timeout, units = "secs")
  deadline <- Sys.time() + timeout
  px <- process$new(command, args, stdout = "|", ...)
  while (px$is_alive() && (now <- Sys.time()) < deadline) {
    poll_time <- as.double(deadline - now, units = "secs") * 1000
    px$poll_io(as.integer(poll_time))
    lines <- px$read_output_lines()
    if (any(grepl(message, lines))) return(px)
  }

  px$kill()
  stop("Cannot start ", command)
}
```

Use `start_program` like this:
```r
start_program("phantomjs", "-w", "running on port")
```

Some comments about `start_program()`:

* It waits for `message` to show up in the standard output of the process.
* If this does not happen within 5 seconds, it throws an error.
* On success, it returns the process object.
* The returned process object still has a connection to the standard output
  of the process. This needs to be read out regularly, otherwise its buffer
  fills up, and the subprocess stops, until the buffer it freed.
  Alternatively, one can close it with `close(px$get_output_connection())`.
* If an error happens, the subprocess is terminated when the process object,
  referred to by `px` within the function, is garbage collected.

## Related tools

### The ps package

The [ps](https://ps.r-lib.org) package deals with system processes in
general. processx and ps methods overlap, in fact processx uses ps to
implement some of its methods. It is also possible to create a `ps_handle`
object from a processx object, with the `$as_ps_handle()` method.
This can then be used with the ps functions directly:


```r
proc <- process$new(px, c("sleep", "3"))
ps <- proc$as_ps_handle()
ps::ps_memory_info(ps)
#>        rss        vms    pfaults    pageins 
#>     663552 2491170816        323          0
```

The ps package includes a testthat reporter that can be used to check
that testthat test cases clean up all their child processes and close
their connections and open files. See `?ps::CleanupReporter` for details.

Here is a simple example on how to use it. In the `testthat.R` file of your
package, update the `test_check()` call to use `CleanupReporter`. Since
ps is not supported on all platforms (only Windows, macOS and Linux
currently), we also need to check for ps support:

```
if (ps::ps_is_supported()) {
  reporter <- ps::CleanupReporter(testthat::SummaryReporter)$new()
} else {
  ## ps does not support this platform
  reporter <- "progress"
}

test_check("<package-name>", reporter = reporter)
```

`CleanupReporter` will check for leftover child processes, R connections
and open files at the end of each `test_that()` block. If a check fails,
it generates a regular testthat test failure.

### The callr package

The callr package uses processx to start another R process, and run R
code in it. It can start R processes synchonously or asynchronously,
and the R processes can be either state-less or stateful. See `?callr::r`,
`?callr::r_process` and `?callr::r_session` for details.

## Links:

* [processx on GitHub](https://github.com/r-lib/processx)
* [processx documentation](https://processx.r-lib.org)
* [ps on CRAN](https://cran.r-project.org/package=ps)
* [callr on CRAN](https://cran.r-project.org/package=callr)

## Acknowledgements

We’re grateful to the 8 people who contributed issues, code and comments
since the last processx release:

  [&#xFF20;breichholf](https://github.com/breichholf), [&#xFF20;dchiu911](https://github.com/dchiu911), [&#xFF20;gaborcsardi](https://github.com/gaborcsardi), [&#xFF20;hadley](https://github.com/hadley), [&#xFF20;joelnitta](https://github.com/joelnitta), [&#xFF20;matthijsvanderloos](https://github.com/matthijsvanderloos), [&#xFF20;maxheld83](https://github.com/maxheld83), and [&#xFF20;wlandau](https://github.com/wlandau)
  
