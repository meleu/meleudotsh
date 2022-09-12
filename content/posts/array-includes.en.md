---
translationKey: elemento-presente-no-array
title: "Checking if an array contains an element in bash"
date: 2020-12-12T05:00:59-03:00
tags:
  - arrays
  - pure-bash
cover:
  image: "img/elemento-presente-no-array.png"
  alt: "code snippet"
---

When working with arrays it's quite common to face the need to check if an array includes a certain element. Although we can have arrays in bash, we don't have a specific method to check that. In this article we're going to address this problem using a not so known bash feature.

## TL;DR

If you just want the solution and don't care about how it works, here it is (the explanation comes right after):
```bash
#!/usr/bin/env bash

joinByChar() {
  local IFS="$1"
  shift
  echo "$*"
}

# if extglob is not enabled, uncomment the line below
# shopt -s extglob
# The function returns status 0 if the array contains the element
elementInArray() {
  local element="$1"
  shift
  local array=("$@")
  [[ "$element" == @($(joinByChar '|' "${array[@]//|/\\|}")) ]]
}
```


## Explanation


### `extglob`

We're going to use the shell option `extglob`, which stands for Extended Globbing. This feature was implemented in bash in version 2.02 (1998), so, unless you're using a 20 year old system, this method is pretty portable.

With `extglob` we have a richer way to specify sets of filenames (aka [globbing](https://en.wikipedia.org/wiki/Glob_(programming))). There are many cool things you can do with this feature. Here in this article we're going to use just a single part of it.

In order to use the method we're going to explain here the shell option `extglob` must be enabled. It seems to be the default nowadays, but to be sure, let's check it:
```
$ shopt extglob
extglob         on

$ # if it was off, you could turn it on with the following command

$ shopt -s extglob

$ # the -s option stands for *set* the option (turn it on)
```

Once that option is enabled, we can search for a string inside a list where each element is separated by a `|` pipe. Example:
```bash
[[ $element == @(element1|element2|elementN) ]]
```

Let's a practical example:
```
$ [[ one == @(one|two|three) ]] && echo yes || echo no
yes

$ [[ four == @(one|two|three) ]] && echo yes || echo no
no
```

Pretty simple, isn't it? In order to use this feature to check if an element is present in an array, first we need to get each element of the array and separate them with a `|` pipe. So, let's use a function to help us with that.


### `joinByChar()`

We addressed this problem in [a previous article](https://dev.to/meleu/how-to-join-array-elements-in-a-bash-script-303a) where we created the `joinByChar()` function. The function is pretty short (you can find the explanation of how it works in that article):
```bash
joinByChar() {
  local IFS="$1"
  shift
  echo "$*"
}
```

Nice, now looks like we can do this:
```bash
#!/usr/bin/env bash
# elementInArray.sh
###################

joinByChar() {
  local IFS="$1"
  shift
  echo "$*"
}

# if extglob is not enabled, uncomment the line below
# shopt -s extglob
elementInArray() {
  local element="$1"
  shift
  local array=("$@")
  [[ "$element" == @($(joinByChar '|' "${array[@]}")) ]]
}
```

Let's test it:
```
$ source elementInArray.sh 

$ array1=(one two 'three|four' 'five six')

$ elementInArray ten "${array1[@]}"

$ elementInArray one "${array1[@]}" && echo yes || echo no
yes

$ elementInArray two "${array1[@]}" && echo yes || echo no
yes

$ elementInArray three "${array1[@]}" && echo yes || echo no
yes

$ # wait! I don't have an element 'three' in that array
```

Uhm... That `|` in the element `three|four` "confused" our function. We're going to need more bash tricks here.

Let's use the variable substring replacement to escape our `|` pipes.


### Escaping the pipes

The construction we're using here is sometimes called **global replacement**, and it works like this:
```
${variable//pattern/replacement}
```

Where all matches of `pattern`, within `variable` is replaced with `replacement`.

We're going to use this: `${array[@]//|/\\|}`

Which means "replace every ocurrence of `|` found in the elements of `array` with `\|`".

So, our final solution will be this one:
```bash
#!/usr/bin/env bash
# elementInArray.sh
###################

joinByChar() {
  local IFS="$1"
  shift
  echo "$*"
}

# if extglob is not enabled, uncomment the line below
# shopt -s extglob
# The function returns status 0 if the array contains the element
elementInArray() {
  local element="$1"
  shift
  local array=("$@")
  [[ "$element" == @($(joinByChar '|' "${array[@]//|/\\|}")) ]]
}
```

Let's see if it's really safe:
```
$ source elementInArray.sh 

$ term='watermelon'

$ fruits=(grape apple orange kiwi)

$ elementInArray "$term" "${fruits[@]}"

$ elementInArray "$term" "${fruits[@]}" && echo true || echo false
false

$ # let's make it easier to check the exit status with an alias:

$ alias result='echo true || echo false'

$ elementInArray "$term" "${fruits[@]}" && result
false

$ # let's try to confuse it with   s p a c e s

$ term='passion'

$ fruits=('passion fruit' apple orange kiwi)

$ elementInArray "$term" "${fruits[@]}" && result
false

$ term='passion fruit'

$ elementInArray "$term" "${fruits[@]}" && result
true

$ # let's try to confuse it with|pipes|too

$ array1=(one two 'three|four' 'five six')

$ elementInArray one "${array1[@]}" && result
true

$ elementInArray three "${array1[@]}" && result
false

$ # nice! we, indeed, don't have a 'three' element

$ elementInArray 'three|four' "${array1[@]}" && result
true

$ elementInArray 'five' "${array1[@]}" && result
false

$ elementInArray 'five six' "${array1[@]}" && result
true
```

As you can see, this seems to be a nice method to check if an array contains an element.


## Links

- Bash Extended Globbing: <https://www.linuxjournal.com/content/bash-extended-globbing>
- How to join array elements in a bash script: <https://dev.to/meleu/how-to-join-array-elements-in-a-bash-script-303a>
- List of features added to specific releases of Bash: <http://mywiki.wooledge.org/BashFAQ/061>
- https://tldp.org/LDP/abs/html/parameter-substitution.html
