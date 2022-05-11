---
layout: default
title: Linux Broken Executable
parent: Security
nav_order: 2
---

# Linux Broken Executable
{: .fs-9 .no_toc}

The ltrace/strace mini challenge from the 2021 Holiday Hack Challenge from SANS Institute 
{: .fs-6 .fw-300 }

[Solution](#solution){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 } [View CTF](https://www.sans.org/mlp/holiday-hack-challenge/){: .btn .fs-5 .mb-4 .mb-md-0 }

---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Challenge
We replaced the SD card in the Cranberry Pi that controls it and re-installed the software. Now it's complaining that it can't find a registration file!

## Solution
In this challenge, we will make use of the ltrace/strace commands to figure out why the cotton candy executable is broken. What do these commands do though?

ltrace is a program that simply runs the specified command until it exits. It intercepts and records the dynamic library calls which are called by the executed process and the signals which are received by that process. It can also intercept and print the system calls executed by the program. (https://linux.die.net/man/1/ltrace)

strace runs the specified command until it exits. It intercepts and records the system calls which are called by a process and the signals which are received by a process. The name of each system call, its arguments and its return value are printed on standard error or to the file specified with the -o option. (https://linux.die.net/man/1/ltrace)

Let's log into the terminal. In our directory, we see one executable file called make_the_candy which prints out an error message when we run it. The error describes not being able to find a configuration file. Let's find out which one using the ltrace command! 

```bash
kotton_kandy_co@e4bc5c940972:~$ ltrace ./make_the_candy 
fopen("registration.json", "r")                           = 0
puts("Unable to open configuration fil"...Unable to open configuration file.
)               = 35
+++ exited (status 1) +++
```

The program is making an fopen library call to open a file called registration.json but it does not seem to exist as denoted by the = 0 following the call. We can confirm this file does not exist by searching the current directory (and even the entire file system). Using the touch command, we will go ahead create our own registration.json file and rerun the make_the_candy command to see if it is fixed.

```bash
kotton_kandy_co@e4bc5c940972:~$ ./make_the_candy 
Unregistered - Exiting.
```

The error is now outputting that the command is Unregistered. Running the ltrace command again we have the following output:

```bash
kotton_kandy_co@e4bc5c940972:~$ ltrace ./make_the_candy 
fopen("registration.json", "r")                           = 0x558920bfa260
getline(0x7ffe5473ecb0, 0x7ffe5473ecb8, 0x558920bfa260, 0x7ffe5473ecb8) = -1
```

The program is trying to read a line in the file, but there is nothing in it. We need to determine what to put into our JSON file. JSON stands for JavaScript Object Notation and is a way to represent a map data structure using just strings. 

```bash
{"test": "this"}
```

Let's try putting this random JSON string into our registration file and re-running the ltrace command.

```bash
kotton_kandy_co@e4bc5c940972:~$ ltrace ./make_the_candy 
fopen("registration.json", "r")                           = 0x55acfe956260
getline(0x7fffae06e050, 0x7fffae06e058, 0x55acfe956260, 0x7fffae06e058) = 16
strstr("{"Test":"This"}\n", "Registration")               = nil
```

This time we see the strstr library call comparing our JSON to the word registration. Let's change our JSON key to "Registration." 

```bash
kotton_kandy_co@e4bc5c940972:~$ ltrace ./make_the_candy 
fopen("registration.json", "r")                           = 0x55ca37a4a260
getline(0x7ffdb71703c0, 0x7ffdb71703c8, 0x55ca37a4a260, 0x7ffdb71703c8) = 24
strstr("{"Registration":"This"}\n", "Registration")       = "Registration":"This"}\n"
strchr("Registration":"This"}\n", ':')                    = ":"This"}\n"
strstr(":"This"}\n", "True")                              = nil
```

We are getting closer! The library calls first looked for "Registration" and the ":" character, but this time it could not find the word "True". We can make the assumption now that the program is parsing the JSON string by first looking for the key, then the colon delimiter followed by the value. We will change the value of our JSON to "True" and run ltrace once more. Delightfully, the program now executes and makes the candy for all the children!!