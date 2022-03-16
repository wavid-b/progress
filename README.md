# progress
A utility for processing large amounts of textual output in test and build systems.

## Introduction

Testing and building software systems are notoriously verbose tasks that generate a lot of textual output indicating their progress.
This logging output is generally difficult to sift through by visual inspection alone, especially when such output consists of hundreds or thousands of lines, inducing a condition known as "scroll blindness".
Most of the time, when things go well, tests pass and builds succeed, and we are only interested in seeing success or a short summary of failure.
Yet detailed logs are key for diagnosing a failure deep in a complex build or set of tests.
We don't want to see the details every time, so how do we resolve this dillema?

## Denote Subtasks For Simplified Output

The answer is that we should *both* log the verbose details of the process, but only present a simplified output to the user who can then tell at a glance that something succeeded, or if it failed, where and how.
This repo contains a set of utilities to make this a snap.
With a small change to your process's standard output, interspersing a few text lines here and there, you can make use of the standard tools here.
Then, you don't need to reinvent the wheel making a nice UI for users waiting on your program to finish.

## Escaped Lines Denote Subtasks

We start with a process that *already* logs text.
The next step is to break down a large task into subtasks of an appropriate granularity.
For example, in testing, a test case; in a build system, a build step.
If we know how many subtasks are in a computation, and we denote where they begin and end in the log, and whether they succeeded, it's possible to use the log itself to track progress and generate a clean report to the user in terms of subtasks.

The utilities here do this by processing textual output from the `stdout` of your process, line by line.
If the line starts with a special escape sequence, then the utilities parse the line, learning how many tests are coming, where they start and end, and their outcome.
Allow other lines are simply ignored, so you change *nothing else* about how you log your output.
Once the format is agreed upon, standard utilities can generate clean reports for all kinds of different processes, in a completely language-agnostic way.

### Escape Sequences For Line Starts

To keep things simple, our utilities process text line by line, looking only for special lines that start with a delimiter.
To denote a special line, a process outputs a line that *must begin with* the two characters `##`.
Such a line will be treated specially, and all other lines will be ignored.
(If your process happens to output lines starting with `##` for some other reason, you can escape them with `###`, or another way.)

The special line command are:

* `##>`, followed immediately by a decimal integer. This declares the number of coming subtasks.
* `##+`, followed by a string. This declares the start of a subtask and gives it a name.
* `##-ok` Reports that the current subtask has completed successfully.
* `##-fail`, followed by an optional message. Reports the current subtask has failed with a message.

That's it! You only need to intersperse these lines into your standard output, and then a tool can parse it to do all the presentation magic.

## Using the Pre-built `progress` Utility

At this point, let's assume you've added these special lines in your process output.
Nothing is better just yet!
Now the main consideration is how to present a cleaned up summary of your process's execution to the user.
This what the pre-built `progress` utility in this repositories does!
We can simply *pipe* the standard output from our process through it.

```
% myprocess | progress
```

This utility is nothing more than a text processor that cleans up the process output and make it presentable in a nice way.
In particular, by default, it drops *all* of the uninteresting lines and present only a summary.
If we want to *also* keep the original log, with all of its details, we can use the UNIX way; the `tee` utility dumps a file.

```
% myprocess | tee mylog.txt | progress
```

## Output Styles of default `progress` utility

The `progress` utility is designed to work with thousands and millions of subtasks.
It processes the output at high speed; it doesn't keep any complex data structures.
This makes it suitable for almost anything at all.
But thousands and millions of subtasks, even leaving out the details of your log, can be an overwhelming amount of information.
Thus this utility has several different output modes.

### Character mode

The default mode for the `progress` utility is to output at least one character for every subtask.
A passing subtask outputs a green `o` and a failing subtask outputs a red `X`.
By default, a line break is inserted every 50 subtasks.
The names of failing subtasks are buffered and reported at the end in red.

### Lines mode

This outputs one line per subtask, with the name of the subtask.
A passing subtask outputs a green `ok` at the end of the line and a failing subtask outputs a new line with the name of the subtask in red.
Failures are reported as they happen.
This is useful, e.g. if your process *crashes*.

### Interactive mode

One of the most compact modes, this mode outputs the name of the *current* task running, as well as the number of passed/failed.
This is very useful to see the incremental progress of a process and helps to diagnose subtasks that get stuck.
This is also useful if your process crashes.

### Summary mode

The most compact mode, this mode only outputs pass/fail for the entire process execution, i.e. if *all* subtasks pass.
It outputs the pass/fail in the *same progress format*, so it could be processed by *another* instance of the utility.

## Make It Your Own!

The simplicity of this approach is that we decouple the producer of log output from the presentation to the user.
The connection between the two is simply the standardized, stylized lines above.
You can and *should* write your *own* processors that format subtask output however you like.
There is no limit to what you can do: make a GUI, plot multiple processes in parallel, use the coolest terminal tricks you can muster.
Go for it!
