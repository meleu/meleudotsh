---
title: "How to join() array elements in a bash script"
description: >
  Some languages (like JavaScript and PHP) have a function like 'join()' or 'implode()' to join the elements of an array separating them by a character or a string. Let's do the same in pure bash.
date: 2020-12-05T14:46:31-03:00
tags:
  - pure-bash
cover:
  image: "img/join-bash.png"
  alt: "joinBy() function"
draft: true
---


Some languages (like JavaScript and PHP) have a function like `join()` or `implode()` to join the elements of an array separating them by a character or a string. Let's do the same in pure bash.

As a bonus, at the end this article you'll know:
- the differences between `$*`, `$@`
- how the `IFS` variable influences the behaviour of `$*`
- a variable expansion/substring replacement technique
- a `printf` "hack"
- how to find help from your system (it can be faster than googling)
- how to search inside a huge man page (like the bash's one)
- how to quickly get help for the bash-builtin commands (rather then searching inside that huge man page)

Let's go!

## Join elements with a single character

As they say, "Talk is cheap, show me the code.".

```sh
joinByChar() {
  local IFS="$1"
  shift
  echo "$*"
}
```

Now let's see that code in action:

```txt
$ cat joinByChar.sh
#!/usr/bin/env bash

joinByChar() {
  local IFS="$1"
  shift
  echo "$*"
}

$ source joinByChar.sh

$ joinByChar , Moe Larry Curly
Moe,Larry,Curly

$ stooges=(Moe Larry Curly)

$ joinByChar , "${stooges[@]}"
Moe,Larry,Curly

$ joinByChar | "${stooges[@]}"
Moe: command not found

$ # Whoops! We need to protect the pipe with 'single quotes'

$ joinByChar '|' "${stooges[@]}"
Moe|Larry|Curly
```

And then let's understand how that works.


### What's `IFS` doing?

IFS stands for "Internal Field Separator". This is a special variable with a string of characters used to be treated as delimiters when splitting a line of input into different words.

Its default value is a string with a space followed by tabulation followed by new line.

Here's an example to show its behavior:

```
$ echo "$PATH"
/bin:/usr/bin:/usr/local/bin

$ for item in $PATH ; do echo $item; done;
/bin:/usr/bin:/usr/local/bin

$ IFS=:

$ for item in $PATH ; do echo $item; done;
/bin
/usr/bin
/usr/local/bin
```

As you may know, the `PATH` variable is a string with a collection of paths separated:by:a:colon.

In that example, the first `for` had only one iteration, because the whole content of `PATH` was considered as the only argument.

After setting the `IFS` as `:`, we're telling bash that we want it to split the content of `PATH` into distinct words by separating them where there's a `:` colon.

Therefore, with `IFS=:` the string in the `PATH` variable will be splitted exactly where there are `:` colons. As a result, that `for` clause will get the directory names present in the `PATH` individually.

(I'd like to write a specific article about IFS at some point, but if you want to go deeper right now, I recommend checking the [Greg's Wiki](https://mywiki.wooledge.org/IFS)).

In the `joinByChar()` function, we're setting the `IFS` with the char we want to use to join the elements in only one string. Keep reading and you'll see why it's needed.


### What's `shift` doing?

The `shift` builtin command is used to handle the positional parameters. When you use the `shift`, the first parameter goes away, the second becomes `$1`, the third becomes `$2` and so on.

In the `joinByChar()` function, as soon as we saved the character to be used as the separator in the `IFS` variable, we don't need that content to be in the `$1` anymore. Let's get rid of it with a `shift` and now all other arguments (the elements we want to join) so we can refer to all arguments just by using `$*`, which is explained right below.

### How does `$*` really work?

By checking the bash manpage in the "Special Parameters" section, we can see how the `*` and the `@` work when in the command line parameters context ( `$1`, `$2`, etc.).

**Pro Tip**: the manpage of bash is really really big. So, when in the `man` program, type `/` followed by the string you wanna search (case sensitive). For example `/Special Parameters`.

Here's an excerpt of what is described about the expansion with `*` from the bash manpage:

> When the expansion occurs within double quotes, it expands to a single word with the value of each parameter separated by the first character of the `IFS` special variable. That is, `"$*"` is equivalent to `"$1c$2c..."`, where `c` is the first character of the value of the `IFS` variable.

Therefore `*` expands separating with the first character in the `IFS` variable, which in the `joinByChar()` function is the first character given as the first argument.

And that's how the `joinByChar()` works. This technique serves you only if you want to join the elements with a single character. If you need to join the elements "gluing" them with a string check the technique below.


## Join elements with a string

First the code:

```sh
#!/usr/bin/env bash
joinByString() {
  local separator="$1"
  shift
  local first="$1"
  shift
  printf "%s" "$first" "${@/#/$separator}"
}
```

Then that code in action:

```
$ joinByString , Moe Larry Curly
Moe,Larry,Curly

$ joinByString ', ' Moe Larry Curly
Moe, Larry, Curly

$ joinByString ' <-> ' Moe Larry Curly
Moe <-> Larry <-> Curly

```

It can be a bit tricky to understand how exactly that function is working. It's combining some handy features of bash and it can be overwhelming to understand all at once. So let's break it into smaller pieces...

We already saw how `shift` works and how it influences the positional parameters. So the explanation here will be focused in the last line of the function:

```sh
printf "%s" "$first" "${@/#/$separator}"
```

There's a lot of bash concepts behind this single line of code, so keep calm and stay with me.

Starting with this cryptic string: `${@/#/$separator}`


### How does `$@` work?

An excerpt of what is described about the expansion with `@` from the bash manpage:

> Expands to the positional parameters, starting from one. (...) When the expansion occurs within double quotes, each parameter expands to a separate word. That is, "$@" is  equivalent  to "$1" "$2" ...

Differently than `$*`, which expands "gluing" the parameters with the first char of the `IFS`, the `$@` expands separating the parameters **with a literal space**.

That part in bold is important, then I'm gonna repeat it: the `$@` expands separating the parameters **with a literal space**.


### Substring replacement with `${variable/#pattern/replacement}`

It works like this: if `variable` starts with `pattern`, then `pattern` will be replaced with `replacement`.

Check this example:
```
$ var='one two three'

$ echo "${var/#one/1}"
1 two three

$ # as var starts with 'one' it worked as expected

$ echo "${var/#two/2}"
one two three

$ # var doesnt start with 'two', then nothing happens

```

As you can see, the replacement only works if the variable starts with that pattern.

And here's the trick to put a prefix before the variable's content: use an empty pattern.

We can say that an empty pattern means "nothing", and every string starts with "nothing", right?

Let's see it in action:

```
$ var='one two three'

$ echo "${var/#/zero }"
zero one two three
```

### Substring replacement with `${@/#pattern/replacement}`

An interesting thing happens when we combine this replacement technique with the `$@`. If you do something like `"${@/#two/2}"`, it'll be expanded to `"${0/#two/2}" "${1/#two/2}" "${2/#two/2}" ...`

Check it out:
```
$ replace() { echo "${@/#two/2}"; }                                             

$ replace one two three
one 2 three
```

The `replacement` part of that construction can also be a variable. Let's prefix all those numbers with something:
```
$ addPrefix() { echo "${@/#/$prefix}"; }

$ prefix='something->'

$ addPrefix one two three
something->one something->two something->three

$ prefix=', '

$ addPrefix one two three
, one , two , three
```

As you can see, it adds the prefix before every parameter.

Let's remember the problem we want to solve: concatenate all parameters "gluing" them with a specific string. We can achieve this by prefixing all parameters with a string, but we must not prefix the first parameter.

If you're following me, you probably realised that the "first parameter" thing is easy to solve by saving it in a variable and then using `shift`.

But there's another thing that doesn't look quite right...
```
$ addPrefix one two three
, one , two , three
```

We don't want those spaces after each element. To solve that, we're going to use the `printf`.


### How does bash's `printf` work?

I'm assuming you have some familiarity with `printf`. If you have a C language background, I'm sure you're comfortable with it, but if you never used `printf` before [here's an interesting article specific for the bash context](https://opensource.com/article/20/8/printf).

**Pro Tip**: To access the documentation of a bash-builtin command use `help`. As `printf` is a builtin, check its documentation with `help printf`.

The term `printf` stands for "print formatted" where you can tell the format you want by using format specifiers. The one we're going to use here is the `%s`, which is simply a string.

Let's check an example:
```
$ name='meleu'

$ adjective='nice guy'

$ printf "%s is a %s" "$name" "$adjective"
meleu is a nice guy
```

As you can see, each `%s` is replaced with the respective string passed as argument. The first `%s` is replaced with the contents of `$name` and the second one is replaced with `$adjective`.

The interesting `printf` feature that I want to highlight here is this single sentence buried in that huge bash manpage:

> The format is re-used as necessary to consume all of the arguments.

Uhm... Interesting, let's check that:
```
$ namesAndAdjectives=('meleu' 'nice guy' 'swyx' '#LearnInPublic evangelist')

$ printf "%s is a %s\n" "${namesAndAdjectives[@]}"
meleu is a nice guy
swyx is a #LearnInPublic evangelist
```

That's cool! Now we have everything we need to understand that cryptic line of code:

```sh
printf "%s" "$first" "${@/#/$separator}"
```

If you're following me until here you can agree with this description of that code:

Print the content of `$first` and then print each parameter prefixing them with the content of `$separator`.



## Links


- <https://stackoverflow.com/a/17841619>
- <https://mywiki.wooledge.org/IFS>
- <https://tldp.org/LDP/abs/html/parameter-substitution.html>
- `man bash` in the "Special Parameters" section
- `man bash` in the "Parameter Expansion" section
- `help printf`
