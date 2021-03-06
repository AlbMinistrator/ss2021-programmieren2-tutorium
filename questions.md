# Questions

This file is a collection of questions that are being asked during the tutorial and can't be answered immediatly.

## Java

### Question: Why do we use the keywords ``static final`` when creating a constant?

Let's start with the ``final`` keywoard. In Java, when we denote something (a class, a variable or a method) it can only be assigned once. After the assignment happened, it can't be reassigned.

Declaring a variable as static means, it's associated with the type (for example, our Roboter class). That means, we can reference the variable without creating an instance of the class.
The static variable get's it's memory once, when the class is loading. It's shared between
different instances of the same class.

Let's take a look at an example to make it clear:

Let's remove the ``private`` keywoard from our FlyingRobots ``CAN_FLY`` value.

```java
public final class FlyingRobot extends Roboter {
	final static boolean CAN_FLY = true;
    .
    .
    .
}
```
Now, let's try to access this value in our main function!

```java
public static void main(String[] args) {
		
		if (FlyingRobot.CAN_FLY) {
			System.out.println("It can fly!");
		} else {
			System.out.println("Sadly, the robot can't fly :(");
		}
		
	}
```
And we're successfull! The computer tells us that our robot, indeed, can fly!
```
It can fly!
```

### TL;DR

Assigning the variable as ``static final`` creates a constant that we can access without creating an instance of a type first. This is required for our ```super()``` constructor in the 
FlyingRobots class. 

## C/C++

### Question: Why does strlen(string) and sizeof(string) return different results?

First, let's write a small program to examine the question:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
    char password [] = "C-Programmierung";

    printf("sizeof(password): %lu\n", sizeof(password));
    printf("strlen(password): %lu\n", strlen(password));


    return EXIT_SUCCESS;
}
```
One would expect that both values are equal, yet the result from ``sizeof(password)`` is larger.
```bash
$./example 
sizeof(password): 17
strlen(password): 16
```

What exactly is ``sizeof(password)`` doing? To answer that, we will look it up in the [GNU C Manual](https://www.gnu.org/software/gnu-c-manual/gnu-c-manual.html#The-sizeof-Operator).

>You can use the sizeof operator to obtain the size (in bytes) of the data type of its operand. The operand may be an actual type specifier (such as int or float), as well as any valid expression. When the operand is a type name, it must be enclosed in parentheses

As we know now, ``sizeof(password)`` returns the number of bytes ``password`` takes up in memory. \
But wait, our super secure password ``C-Programmierung`` contains 16 chars. The size of 
eacht char is a byte. Where does the 17th byte come from?

To understand why our password takes up 17 bytes, we need to understand how C writes strings in memory. In C, a string is an array that is filled with chars. To know at what points in memory
the s tring ends, C uses [Null-terminated strings](https://en.wikipedia.org/wiki/Null-terminated_string).
So, at the end of a string, it adds a single byte: ``\0``.
That means, the memory where our string is saved looks like this:

>| C | - | P | r | o | g | r | a | m | m | i | e | r | u | n | g | \0 |

Well, of course it is saved in the ASCII representation, so it's more like this (hex representation):

>| 43| 2d | 50 | 72 | 6f | 67 | 72 | 61 | 6d | 6d | 69 | 65 | 72 | 75 | 6e | 67 | 00

Now that we lifted the secret of our ``sizeof`` return value, let's examine
``strlen``.

---

The C++ reference for [strlen](https://www.cplusplus.com/reference/cstring/strlen/) states:

>[strlen retruns] the number of characters between the beginning of the string and the terminating null character (without including the terminating null character itself)

This is why ``strlen`` returns the correct length of the string - it stops at the null
character.

The in memory size of a string and the length of a string should never be confused with each
other. To drive the point home, let's create a second example program:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
    char password_storage [100];
    
    printf("Please enter your password: \n");
    scanf("%99s", &password_storage[0]);

    printf("sizeof(password_storage): %lu\n", sizeof(password_storage));
    printf("strlen(password_storage): %lu\n", strlen(password_storage));

    return EXIT_SUCCESS;
}
```
As our password we choose ``Albstadt-Ebingen``. Our output is:

```bash
./example 
Please enter your password: 
Albstadt-Ebingen
sizeof(password_storage): 100
strlen(password_storage): 16
```
As we see, because we've initialized ``password_storage`` with enough "room" for 100 chars, it takes up 100 bytes in memory. The fact that our password is only 16 chars long doesn't matter, the rest of the memory is initialized with ``null`` values.

### TL;DR
To not confuse the string length with it's allocated memory, always use ``strlen``.

----

### Question: How does fork() work? Does a switch in C always run default?
At the end of the lecture we faced the following question:
>Does a switch in C always evaluates its default branch?

As we were in a hurry, I said yes - which is **wrong**.

Let's take a closer look at what happens in our small example program:

```c
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    pid_t pid;

    switch (pid = fork()) {
    case -1:
        printf("Error: failed fork()");
        exit(EXIT_FAILURE);
    case 0:
        printf("pid_t pid in child process: %i\n", pid);
        break;
    
    default:
        printf("pid_t pid in parent process: %i\n", pid);
    }

    return EXIT_SUCCESS;
}
```

Firstly, we declare our variable `pid`, short for process ID, as of type ``pid_t``. That type is new to us, so let's dive head first into the   [Linux Manual Pages](https://man7.org/linux/man-pages//man3/pid_t.3.html) to learn about it:

```
 pid_t
              Include: <sys/types.h>.  Alternatively, <fcntl.h>,
              <sched.h>, <signal.h>, <spawn.h>, <sys/msg.h>,
              <sys/sem.h>, <sys/shm.h>, <sys/wait.h>, <termios.h>,
              <time.h>, <unistd.h>, or <utmpx.h>.

              This type is used for storing process IDs, process group
              IDs, and session IDs.  According to POSIX, it shall be a
              signed integer type, and the implementation shall support
              one or more programming environments where the width of
              pid_t is no greater than the width of the type long.

              Conforming to: POSIX.1-2001 and later.
```
Okay, let's break that down.
First, the type ``pid_t`` is declared inside ``<sys/types.h>``. 
It can be used to store three different IDs:

<ul>
    <li> process IDs
    <li> process group IDs
    <li> session IDs
</ul>

We don't need process group IDs and session IDs now, so let's toss them aside.
At last, it's revealed: 
``pid_t`` is a signed integer which is smaller than a ``long`` type.

With that out of the way, let's continue in our main function. 

Here comes the ```pid = fork()``` call.

This is the point where our process clones itself. With that, we now have two **separate** processes, a child and its parent process. 

Execution continues in both processes, we're now reaching the point where the switch
evaluates ``pid``. But wait, what value do both ``pid`` values - one inside the parent process and one inside the child process - have?

To figure that out, let's call  ``man fork()`` and scroll down to the section ``return value``

```
       On success, the PID of the child process is returned in the
       parent, and 0 is returned in the child.  On failure, -1 is
       returned in the parent, no child process is created, and errno is
       set to indicate the error.
```

If the fork was successful, what we're going to assume, the ``pid`` in the parent process is the childs process id, which is **neither** a 0 or a -1. Inside the child process, the ``pid`` equals 0. 

With that out of the way, let's get back to our switch:

```c
switch (pid = fork()) {
    case -1:
        printf("Error: failed fork()");
        exit(EXIT_FAILURE);
    case 0:
        printf("pid_t pid in child process: %i\n", pid);
        break;
    
    default:
        printf("pid_t pid in parent process: %i\n", pid);
    }
```
The parent process executes the code in the default branch because, well, if it's neither a 0 nor a -1, it's the only matching branch. Our child process in the meantime
executes the 0 branch.

To drive our point home, this is the code that gets executed by the 
parent process if the fork was successful:

```c
int main() {
    pid_t pid;

    switch (pid = fork()) {
    default:
        printf("pid_t pid in parent process: %i\n", pid);
    }

    return EXIT_SUCCESS;
}

```
Our child process, with its ``pid`` being a 0, executes the following code:
```c
switch (pid = fork()) {
    case 0:
        printf("pid_t pid in child process: %i\n", pid);
        break;

    return EXIT_SUCCESS;
```
Reminder: the child process can access ``pid``, even though it has been declared
before its creation:

```
       The child process and the parent process run in separate memory
       spaces. At the time of fork() both memory spaces have the same
       content
```

Next step: let's compile and run
our example:
```
./fork_example
pid_t pid in parent process: 471
pid_t pid in child process: 0
```

That looks exactly as we suspected it to! But how can we be sure that both processes
executed successfully?

Let's see what happens under the hoop of our binary by running
``strace -f ./fork_example``. (Remember: to learn about ``strace``, check out its man page!)

To some of us, the output looks like a bunch of gibberish but if we scroll down to the end we encounter the important lines:
```bash
[pid   477] +++ exited with 0 +++
+++ exited with 0 +++
```
It worked! yay!


### TL;DR

Switches in C only evaluate the default branch, if none of the before declared branches meet the requirements. In our example it does get
evaluated because the ``pid`` of our parent process is neither a -1 nor a 0.

----