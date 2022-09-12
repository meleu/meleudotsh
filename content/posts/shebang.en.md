---
title: "What the #! shebang really does"
date: 2020-11-28T00:24:42-03:00
description: >
  What exactly happens when we run a file that starts with '#!' (aka shebang).
tags:
  - best-practices
  - fundamentals
cover:
  image: "img/shebang.png"
  alt: "shebang"
---


What exactly happens when we run a file starting with `#!` (aka shebang), and why some people use `#!/usr/bin/env bash`.


## How the `#!` works

The `#!` shebang is used to tell the kernel which interpreter should be used to run the commands present in the file.

When we run a file starting with `#!`, the kernel opens the file and takes the contents written right after the `#!` until the end of the line. For didactic purposes, let's consider it saves in a variable called `command` the string starting after the shebang and ending in the end of line.

After this the kernel tries to run a command with the contents of the `command` and giving as the first argument the filename of the file we're trying to execute.

Therefore, if you have an executable file called `myscript.sh` with some shell commands and starting with `#!/bin/bash`, when you run it, the kernel will execute `/bin/bash myscript.sh`.

In the examples below you're going to see it very clearly.

Starting with the classic `hello.sh`:

```sh
#!/bin/bash
echo "Hello World!"
```

Assuming this file has the executable permission, when you type this in the command line:

```txt
$ ./hello.sh
```

The kernel will notice the `#!` in the very first line and then will get what's after it, in this case `/bin/bash`. And then what is executed has the very same effect of this:

```txt
$ /bin/bash hello.sh
```

Let's use another example using `#!/bin/cat`. The name of the file is `shebangcat`:

```txt
#!/bin/cat
All the contents of this file will be
printed in the screen when it's executed
(including the '#!/bin/cat' in the first line).
```

Let's remember:
- What's after the shebang: `/bin/cat`
- Name of the file: `./shebangcat`

Therefore this is what's executed: `/bin/cat ./shebangcat`

See it by yourself:
```txt
$ ./shebangcat
#!/bin/cat
All the contents of this file will be
printed in the screen when it's executed
(including the '#!/bin/cat' in the first line).
```

Let's take another example to make it very clear that things are like I'm saying. The following file is called `shebangecho`:

```txt
#!/usr/bin/echo
The contents of this file will *NOT* be
printed when it's executed.
```

Let's check:
```txt
$ ./shebangecho
./shebangecho
```

The output was the name of the file because this is what was executed by the kernel `/usr/bin/echo ./shebangecho`.

Another interesting thing, is that if we pass arguments when calling our script, such arguments will also be passed to the command executed by the kernel. As we can see in the following example called `shebangls.sh`:

```txt
#!/bin/ls
The contents here doesn't matter.
```

Now, when we run it:

```txt
$ ./shebangls.sh
./shebangls.sh

$ ./shebangls.sh -l
-rwxr-xr-x 1 meleu meleu 41 Nov 28 14:42 ./shebangls.sh

$ ./shebangls.sh notfound
/bin/ls: cannot access 'notfound': No such file or directory
./shebangls.sh
```


## Why some people use `#!/usr/bin/env`?

You probably saw some scripts starting with `#!/usr/bin/env bash` where you're used to see just `#!/bin/bash`. The reason of this is to increase the portability of the script (even thought it's a debatable matter, as we're going to see below).

The `env` command, if used with no arguments, prints a (big) list with all the environment's variables. But if `env` is used followed by a command, it runs that command in another instance of the shell.

🤔 - **OK, but how does that influence portability?!**

When you use `#!/bin/bash` you're clearly saying that `bash` is in the `/bin/` directory. This seems to be the default in all Linux distributions, but there are other Unix flavors where it can possibly not happen (for example the `bash` can be placed in the `/usr/bin/`). In systems like that your script starting with `#!/bin/bash` would cause a `bad interpreter: No such file or directory`.

When you run `env bash`, the `env` will search for `bash` in your `$PATH` variable, and then run the first one it finds. Usually `bash` is in `/bin/`, but a user running your script on some other system can have it in `/usr/bin/` or even testing an alternative version in `/home/user/bin/bash`.

So, in order to make the script have a greater reach and be used in environments other than Linux, some people recommend the use of the `env` technique.

🤔 - **But wait! What guarantees that the `env` will always be in the `/usr/bin/`?**

There are no guarantees... 😇

The recomendation is based in what is commonly seen in the Unix systems. I see `/usr/bin/env` being used in some modern projects (like [RetroPie](https://github.com/RetroPie/RetroPie-Setup)), but where it's specially useful is when you need to run a python or even a NodeJS script.

Let's take this NodeJS usage as an example. I want to call a NodeJS script just by calling the script's filename. Then I could do something like this:
```js
#!/usr/bin/node
console.log('Hello World from NodeJS');
```

The problem is that I usually install node via [Node Version Manager](https://github.com/nvm-sh/nvm), instead of using the the distribution's package manager. So, my `node` is like this:

```txt
$ which node
/home/meleu/.nvm/versions/node/v14.15.1/bin/node
```

By any means I want to put `#!/home/meleu/.nvm/versions/node/v14.15.1/bin/node` in my script!

So, the solution here is to use `#!/usr/bin/env node`.


## And if I don't want to use `#!` at all?

I strongly recommend you to never write neither run a shell script without a `#!` shebang!

As we said, the shebang tells to the kernel which interpreter is to be used to run the commands present in the file. If you run a script without specifying the interpreter, the shell will spawn another instance of itself and try to run the commands in the script. Which means that it will execute whatever commands found in the file, even if it was written for zsh, ksh, dash, fish, node, python, or whatever.

**Summing up**: Always start your scripts with a `#!` shebang. Preferably with `#!/usr/bin/env`.


## Links

- http://mywiki.wooledge.org/BashProgramming#Shebang
- https://wiki.bash-hackers.org/scripting/basics#the_shebang
- [Here's an email from Dennis Ritchie](https://www.in-ulm.de/~mascheck/various/shebang/4.0BSD_newsys_sys1.c.html) in 1980, talking about these "magic characters".
- `man env`
- [Node.js shebang](https://alexewerlof.medium.com/node-shebang-e1d4b02f731d)
