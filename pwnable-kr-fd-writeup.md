# pwnable.kr — "fd" Writeup (Easy)

**Category:** Binary Exploitation / Pwn
**Difficulty:** Easy
**Challenge:** [pwnable.kr](http://pwnable.kr) — `fd`

---

## Introduction

This is an easy challenge, great for anyone just starting out with binary pwning. If you're new to this, I'd highly recommend starting with [pwnable.kr](http://pwnable.kr) — it has a lot of beginner-friendly challenges that are great for building fundamentals.

Let's walk through the `fd` challenge.

## Getting Started

After SSH-ing into the given port, we land on a remote server. Running `ls` shows us the files available in the home directory, including the challenge binary `fd` and its source, `fd.c`.

> **Tip:** Whenever you're solving a binary pwning challenge, always read the source code first (if it's provided) before diving into the binary itself.

## Reading the Source

```c
fd@ubuntu:~$ cat fd.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];

int main(int argc, char* argv[], char* envp[]){
    if(argc<2){
        printf("pass argv[1] a number\n");
        return 0;
    }
    int fd = atoi( argv[1] ) - 0x1234;
    int len = 0;
    len = read(fd, buf, 32);
    if(!strcmp("LETMEWIN\n", buf)){
        printf("good job :)\n");
        setregid(getegid(), getegid());
        system("/bin/cat flag");
        exit(0);
    }
    printf("learn about Linux file IO\n");
    return 0;
}
```

## Breaking Down the Logic

The program takes a single command-line argument and checks it against the following logic:

```c
int fd = atoi( argv[1] ) - 0x1234;
```

Here, `atoi` converts the command-line string into an integer. The program then subtracts `0x1234` (4660 in decimal) from that value to compute `fd`.

```c
int len = 0;
len = read(fd, buf, 32);
```

The program treats this computed value as a **file descriptor** and reads 32 bytes from it into `buf`.

### The Key Insight

If we pass in the value `4660` (which is `0x1234` in hex) as our argument, then:

```
fd = 4660 - 0x1234 = 0
```

File descriptor `0` is **stdin** — standard input. So the program is effectively just waiting for us to type something on the keyboard.

## Triggering the Flag

Once `fd` becomes `0`, the program reads whatever we type into `buf`. It then checks:

```c
if(!strcmp("LETMEWIN\n", buf)){
    printf("good job :)\n");
    setregid(getegid(), getegid());
    system("/bin/cat flag");
    exit(0);
}
```

`strcmp` returns `0` when two strings are equal, and the `!` negates that — so this block executes when `buf` **matches** `"LETMEWIN\n"`. All we need to do is type `LETMEWIN` when the program reads from stdin.

## Exploitation

```
fd@ubuntu:~$ ./fd 4660
LETMEWIN
good job :)
Mama! Now_I_understand_what_file_descriptors_are!
fd@ubuntu:~$
```

The flag is:

```
Mama! Now_I_understand_what_file_descriptors_are!
```

Enter it into the flag submission field on the site to get authenticated and complete the challenge.

## Summary

This challenge is a simple introduction to how file descriptors work in Linux and how careless handling of user-controlled input (here, using it directly as a file descriptor) can lead to unexpected program behavior. A great first step into the world of binary exploitation!
