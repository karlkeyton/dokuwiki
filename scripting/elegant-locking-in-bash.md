
[Source](http://www.kfirlavi.com/blog/2012/11/06/elegant-locking-of-bash-program/ "Permalink to Elegant locking of BASH program")

# Elegant locking of BASH program

## Intro

I want to block program from running twice simultaneously. Why would I want such a thing? Lets say you have a program that creates the directory `/tmp/prog` and update files inside this directory. If you run the program twice at the same time, files inside this directory will get clobbered. Searching the net for solution, [`flock`][1] comes up as a good way to solve the problem. [`flock`][1] is part of the [util-linux][2] package.

## Basic solution

Here is an [example][3] posted at [stackoverflow][4]:

| ----- |
|

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12

 |

    #!/bin/bash

    # Makes sure we exit if flock fails.
    set -e

    (
        # Wait for lock on /var/lock/.myscript.exclusivelock (fd 200) for 10 seconds
        flock -n 200

        # Do stuff

    ) 200&gt;/var/lock/.myscript.exclusivelock

 |

At the heart of the locking mechanizm is the last line, which create the file `/var/lock/.myscript.exclusivelock` using file descriptor 200. The syntax `9&gt;textfile`, is the way in BASH to create textfile using file descriptor 9. Then we check if the file is locked in line 8, `flock -n 200` using the file descriptor 200 we used to create the lock file with. So while the critical code after line 8, that runs in the sub shell, is running, the file `/var/lock/.myscript.exclusivelock` stays locked, and will be released when the sub shell exits.

## Next step

My first rule of programming in BASH, is to have all the code functional and the only line that is globally executed is `main` like this:

| ----- |
|

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13
    14
    15

 |

    #!/bin/bash

    do_A() {
        ...
    }

    do_B() {
        ...
    }

    main() {
        do_A
        do_B
    }
    main

 |

So using this example, to lock the program we can do this:

| ----- |
|

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10

 |

    main() {
        do_A
        do_B
    }

    (
        flock -n 200
        main

    ) 200&gt;/var/lock/.myscript.exclusivelock

 |

or:

| ----- |
|

    1
    2
    3
    4
    5
    6
    7
    8
    9

 |

    main() {
        (
            flock -n 200
            do_A
            do_B

        ) 200&gt;/var/lock/.myscript.exclusivelock
    }
    main

 |

First example breaks my rule, and `main` is not the only global code, but is not too bad. The second example, does not look good to me. It seems that main have more syntax complexity then it should. What we really want, in order to keep the code clean, is creating a lock function that will behave like this:

| ----- |
|

    1
    2
    3
    4
    5

 |

    main() {
        lock || exit 1
        do_A
        do_B
    }

 |

## Creating a locking function

To move locking to a function, we'll need to get rid of this sub shell syntax.

| ----- |
|

    1
    2
    3
    4
    5

 |

    (
        flock -n 200
        ...

    ) 200&gt;/var/lock/.myscript.exclusivelock

 |

The answer is to be found in jdimpson's blog post [using flock to protect critical sections in shell scripts][5]. Using `exec` to create the lock file:

    exec 200&gt;/var/lock/.myscript.exclusivelock

Now we can crate the lock file, and use flock to acquire it:

| ----- |
|

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10

 |

    main() {
        exec 200&gt;/var/lock/.myscript.exclusivelock

        flock -n 200
            || exit 1

        do_A
        do_B
    }
    main

 |

Lets create the lock function:

| ----- |
|

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13
    14
    15

 |

    lock() {
        exec 200&gt;/var/lock/.myscript.exclusivelock

        flock -n 200
            &amp;&amp; return 0
            || return 1
    }

    main() {
        lock || exit 1

        do_A
        do_B
    }
    main

 |

So now we have hard coded function ;-) What can we do next? Lets get rid of the magic number 200 and the lock filename, and turn them to a variables.

| ----- |
|

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13
    14
    15
    16
    17
    18
    19
    20
    21
    22
    23
    24
    25
    26
    27
    28
    29
    30
    31
    32
    33
    34
    35

 |

    #!/bin/bash

    readonly PROGNAME=$(basename "$0")
    readonly LOCKFILE_DIR=/tmp
    readonly LOCK_FD=200

    lock() {
        local prefix=$1
        local fd=${2:-$LOCK_FD}
        local lock_file=$LOCKFILE_DIR/$prefix.lock

        # create lock file
        eval "exec $fd&gt;$lock_file"

        # acquier the lock
        flock -n $fd
            &amp;&amp; return 0
            || return 1
    }

    eexit() {
        local error_str="$@"

        echo $error_str
        exit 1
    }

    main() {
        lock $PROGNAME
            || eexit "Only one instance of $PROGNAME can run at one time."

        do_A
        do_B
    }
    main

 |

So now we have a lock function that does what we want. It gets the prefix of the lock filename, and then variable fd is 200 by default, but you can specify different file descriptor number like this `lock $PROGNAME 500`. The important change in the lock function is the turning of 200 to a variable `$fd`, and to be able to execute the lock file creation, we use `eval`. Try it without `eval` and see what happens. You will get error that there is no command 200.

## Conclusion

We arrived from script like locking mechanism to a well closed lock function, that will create exclusivity of the program running. I would go further and paste this lock command to a library of its own. Then source it. i.e. `source /usr/lib/lock.sh`. Look again at the main function and see how elegant and expressive it is. i.e. lock program or exit the program, because another instance of the program is already running.

nJoy,  
Kfir

[1]: http://linux.die.net/man/1/flock
[2]: http://www.kernel.org/pub/linux/utils/util-linux/
[3]: http://stackoverflow.com/questions/169964/how-to-prevent-a-script-from-running-simultaneously
[4]: http://stackoverflow.com
[5]: http://jdimpson.livejournal.com/5685.html
  
