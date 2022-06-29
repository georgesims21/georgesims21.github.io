# strace

Strace is a command used to view the system calls made by an executable program during runtime, it is especially helpful when trying to debug a systems program.

Basic strace commands:
```bash
strace <program executable> # output to stdout the full output
strace -o <file.out> <progex> # output to <file.out> instead of stdout
strace -e <command> <progex> # only output specific <command> (i.e openat/read etc)
strace -e trace=<command1, command2, ...> <progex>
sudo strace -p <pid> -o <file.out> # Can track running process (use tail -f <file> to track updates)
sudo strace -p $(pgrep <prog>) -o <file.out>;tail -f <file.out> # as example

flags:
-t : Give timestamp to each call printed from strace
-r : Same but with relative time (to first sys call)
-c : Output statistics about sys calls (gives totals, time spent calling each etc)
```

### Source
[strace examples](https://www.thegeekstuff.com/2011/11/strace-examples/)


