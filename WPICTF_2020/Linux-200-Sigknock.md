# **Linux: 200 Sigknock**

For this challenge you had to log in to ssh ctf@sigknock.wpictf.xyz with the clue that port knocking was boring and to enhance your security through sigknock.

Quick look at the file system reveals no immediate clues:

```
~ $ ls -la
total 16
drwxr-sr-x    1 wpictf   nogroup       4096 Apr 24 22:25 .
drwxr-xr-x    1 root     root          4096 Apr 14 05:16 ..
-rw-------    1 wpictf   nogroup          7 Apr 24 22:25 .ash_history
```

Look at the process listing and there is a program running irqknock

```
~ $ ps -ef 
PID   USER     TIME  COMMAND
    1 wpictf    0:00 {init_d} /bin/sh /bin/init_d
    6 wpictf    0:00 /usr/bin/irqknock
    7 wpictf    0:00 /bin/sh
    9 wpictf    0:00 ps -ef
```

You can run the program and it accepts input but only certain characters seem to trigger a change in state

```
~ $ /usr/bin/irqknock 
test
1234
a
b
^CGot signal 2
State advanced to 1
^A^\Got signal 3
State advanced to 2
^@^[^\Got signal 3
State advanced to 0
```


Sending various signal interrupts allows you to advance through the states, however there didn't seem to be enough signals mapped to physical keys to get beyond state 3, we can see from the ```man 7 signal page``` that there are a number of other signal interrupts that probably aren't mapped to physical keys.

       First the signals described in the original POSIX.1-1990 standard.
    
       Signal     Value     Action   Comment
       ──────────────────────────────────────────────────────────────────────
       SIGHUP        1       Term    Hangup detected on controlling terminal
                                     or death of controlling process
       SIGINT        2       Term    Interrupt from keyboard
       SIGQUIT       3       Core    Quit from keyboard
       SIGILL        4       Core    Illegal Instruction
       SIGABRT       6       Core    Abort signal from abort(3)
       SIGFPE        8       Core    Floating-point exception
       SIGKILL       9       Term    Kill signal
       SIGSEGV      11       Core    Invalid memory reference
       SIGPIPE      13       Term    Broken pipe: write to pipe with no
                                     readers; see pipe(7)
       SIGALRM      14       Term    Timer signal from alarm(2)
       SIGTERM      15       Term    Termination signal
       SIGUSR1   30,10,16    Term    User-defined signal 1
       SIGUSR2   31,12,17    Term    User-defined signal 2
       SIGCHLD   20,17,18    Ign     Child stopped or terminated
       SIGCONT   19,18,25    Cont    Continue if stopped
       SIGSTOP   17,19,23    Stop    Stop process
       SIGTSTP   18,20,24    Stop    Stop typed at terminal
       SIGTTIN   21,21,26    Stop    Terminal input for background process
       SIGTTOU   22,22,27    Stop    Terminal output for background process
    
       The signals SIGKILL and SIGSTOP cannot be caught, blocked, or ignored.
    
       Next  the  signals  not  in  the POSIX.1-1990 standard but described in
       SUSv2 and POSIX.1-2001.
    
       Signal       Value     Action   Comment
       ────────────────────────────────────────────────────────────────────
       SIGBUS      10,7,10     Core    Bus error (bad memory access)
       SIGPOLL                 Term    Pollable event (Sys V).
                                       Synonym for SIGIO
       SIGPROF     27,27,29    Term    Profiling timer expired
       SIGSYS      12,31,12    Core    Bad system call (SVr4);
                                       see also seccomp(2)
       SIGTRAP        5        Core    Trace/breakpoint trap
       SIGURG      16,23,21    Ign     Urgent condition on socket (4.2BSD)
       SIGVTALRM   26,26,28    Term    Virtual alarm clock (4.2BSD)
       SIGXCPU     24,24,30    Core    CPU time limit exceeded (4.2BSD);
                                       see setrlimit(2)
       SIGXFSZ     25,25,31    Core    File size limit exceeded (4.2BSD);
                                       see setrlimit(2)

So there is a need to test all of the signal interrupts to first find out which interrupts are involved and then what order they need to be in (much like port knocking). It may be possible to iterate through the interrupts in a more automated way with a suitable programming language but there were no compilers or interpreters on the server as it was a pretty basic environment with only some regular tools.

There was probably a better way to do this but I went for the manual approach using the 'kill' command which allows you to send signal interrupts using the -s switch.

```
~ $ /usr/bin/irqknock 
^Z[1]+  Stopped                    /usr/bin/irqknock

~ $ ps -ef 
PID   USER     TIME  COMMAND
    1 wpictf    0:00 {init_d} /bin/sh /bin/init_d
    6 wpictf    0:00 /usr/bin/irqknock
    7 wpictf    0:00 /bin/sh
    9 wpictf    0:00 /usr/bin/irqknock
   10 wpictf    0:00 ps -ef
```

Send the kill command with the first sig value (Hangup), reconnect and check to see if it triggered an advance.

```
~ $ kill -s 1 9
~ $ fg
/usr/bin/irqknock
Hangup
~ $ ps -ef 
PID   USER     TIME  COMMAND
    1 wpictf    0:00 {init_d} /bin/sh /bin/init_d
    6 wpictf    0:00 /usr/bin/irqknock
    7 wpictf    0:00 /bin/sh
   12 wpictf    0:00 ps -ef
```

No luck with '1' so on to the next value, and repeat this until you have what I guessed would be five values.

```
~ $ ps -ef
PID   USER     TIME  COMMAND
    1 wpictf    0:00 {init_d} /bin/sh /bin/init_d
    6 wpictf    0:00 /usr/bin/irqknock
    7 wpictf    0:00 /bin/sh
   13 wpictf    0:00 /usr/bin/irqknock
   14 wpictf    0:00 ps -ef
~ $ kill -s 2 13
~ $ fg
/usr/bin/irqknock
Got signal 2
State advanced to 1
```

So now we know SIGINT '2' is one of our triggers, just repeat this process (bit tedious) until you get the following list of values, which just happen to be prime numbers.

- 2
- 3
- 11
- 13
- 17

Now you have to figure out the order, but thankfully they weren't that evil and it was simply in ascending order, so just fire them at the process one at a time (The demonstration below detaches after each SIGINT but this is only really to check it is progressing):

```
~ $ /usr/bin/irqknock 
^Z[1]+  Stopped                    /usr/bin/irqknock
~ $ ps -ef
PID   USER     TIME  COMMAND
    1 wpictf    0:00 {init_d} /bin/sh /bin/init_d
    7 wpictf    0:00 /bin/sh
   39 wpictf    0:00 /usr/bin/irqknock
   40 wpictf    0:00 ps -ef
~ $ kill -s 2 39
~ $ fg
/usr/bin/irqknock
Got signal 2
State advanced to 1
^Z[1]+  Stopped                    /usr/bin/irqknock
~ $ kill -s 3 39
~ $ fg
/usr/bin/irqknock
Got signal 3
State advanced to 2
^Z[1]+  Stopped                    /usr/bin/irqknock
~ $ kill -s 11 39
~ $ fg
/usr/bin/irqknock
Got signal 11
State advanced to 3
^Z[1]+  Stopped                    /usr/bin/irqknock
~ $ kill -s 13 39
~ $ fg
/usr/bin/irqknock
Got signal 13
State advanced to 4
^Z[1]+  Stopped                    /usr/bin/irqknock
~ $ kill -s 17 39
~ $ fg
/usr/bin/irqknock
Got signal 17
State advanced to 5
WPI{1RQM@St3R}
```


Et voila the flag.
