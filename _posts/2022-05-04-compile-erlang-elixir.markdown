---
layout: post
title:  "Compiling Erlang/OTP and Elixir from sources"
date:   2022-05-04 18:30:40 +0300
categories: elixir erlang otp archlinux
---
It's risky to be on the edge, but it's also breathtaking to test out latest features of your
beloved programming language/environment. Despite being rolling release Archlinux still doesn't
provide you with the latest Erlang/OTP and Elixir versions in it's default repositories. Besides,
Erlang/OTP in Archlinux is compiled without documentation, which give you no chance to use awesome
new Elixir feature in `iex`, when you can read docs on Erlang functions with `h/1` helper. So why
won't we compile our own Erlang/OTP and Elixir;)

### Installing dependencies
First of all, we need to install some dependencies:
```bash
$ pacman -S fop libxslt openssl jdk-openjdk flex wxgtk2
```

Explanation of dependencies:
- `libxsls` and `fop` are required to compile documentation
- `openssl` is needed by _**crypto**_ application
- `jdk-openjdk` is needed to compile _**jinterface**_, which is

> [The Jinterface package provides a set of tools for communication with Erlang
> processes](https://www.erlang.org/doc/apps/jinterface/jinterface_users_guide.html)

- `flex` is needed by _**megaco**_ application, which is

> [Megaco/H.248 is a protocol for control of elements in a physically decomposed multimedia
> gateway, enablingeparation of call control from media
> conversion.](https://www.erlang.org/doc/apps/megaco/megaco_intro.html)

- and `wxgtk2` needed to compile _**wx**_ application, which is used, for example, by `observer`

### Installing latest Erlang/OTP from github
    NOTE: I'll be downloading everything we need into `~/src` directory.

Now, let's clone Erlang/OTP repository and setup some environment variables:
```bash
~/src $ git clone 'git@github.com:erlang/otp.git'
~/src $ cd otp
~/src/otp $ export ERL_TOP=`pwd`
~/src/otp $ ./configure --prefix=${XDG_DATA_HOME:-$HOME/.local/share}/erlang
~/src/otp $ export PATH=$ERL_TOP/bin:$PATH
~/src/otp $ export FOP_HOME=/usr/bin
~/src/otp $ export MAKEFLAGS=-j8
```

`ERL_TOP` is used in compilation time, and in our case should be set to current working directory,
which is repo's root. Passing `--prefix` option to the `./configure` script, we tell compiler,
where do we want to install Erlang/OTP, once it compiled. In my case, the path is set to
`$HOME/.local/share/erlang`. _**I suggest**_ you to do the same, to ease the way you can
remove Erlang from your system by simply deleting this directory.

Updating `PATH` variable is needed for documentation to compile
successfully.
Setting up `FOP_HOME` is for docs compilation too.  `MAKEFLAGS` will be passed to the `make`
commands, `-j8` means I want to compile in parallel using 8 cores. You can adjust it as you want,
by changing number after `-j`.

Next, we are compiling and installing Erlang/OTP and documentation:
```bash
~/src/otp $ make
~/src/otp $ make docs DOC_TARGETS=chunks
~/src/otp $ make install
~/src/otp $ make DOC_TARGETS=chunks install-docs
```

`DOC_TARGETS` is need to be set to one of the following type:
- pdf
- chunks
- man
- html

I've set it to `chunks`, so it won't compile docs in all available formats, to save time and space.

And finally, update your `PATH` environment variable, so we would use newly installed Erlang while
building Elixir from sources:
```bash
~/src/otp $ export PATH=${XDG_DATA_HOME:-$HOME/.local/share}/erlang/bin:$PATH
```

### Installing latest Elixir from github
    NOTE: run this steps in the same shell session with updated `PATH` environment variable.

Let's clone Elixir repo, and compile it with tests afterwards, to be sure everything works
properly:
```bash
~/src/otp $ cd ..
~/src $ git clone 'git@github.com:elixir-lang/elixir.git'
~/src $ cd elixir
~/src/elixir $ make clean test
```

The case when you got some errors in the last step above is beyond the scope of this post, sorry:).

So, finally let's install our freshly compiled elixir into `$HOME/.local/share/elixir` for the
same reason we did it within Erlang/OPT installation:
```bash
$ PREFIX=${XDG_DATA_HOME:-$HOME/.local/share}/elixir make install
```

### Final steps
Don't forget to update your `PATH` environment variable by adding this lines into `.bash_profile`
(if you using bash):
```bash
PATH=${XDG_DATA_HOME:-$HOME/.local/share}/erlang/bin:$PATH
export PATH=${XDG_DATA_HOME:-$HOME/.local/share}/elixir/bin:$PATH
```

### Wrap up
You can test your installation by running `iex`:
```
$ iex
Erlang/OTP 25 [RELEASE CANDIDATE 3] [erts-12.3.2] [source-71b0d4e7cf] [64-bit] [smp:8:8] [ds:8:8:10] [async-threads:1] [jit:ns]

Interactive Elixir (1.14.0-dev) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> h :lists.reverse

                                   reverse/1

  @spec reverse(list1) :: list2 when list1: [t], list2: [t], t: term()

Returns a list with the elements in List1 in reverse order.


                                   reverse/2

  @spec reverse(list1, tail) :: list2
        when list1: [t], tail: term(), list2: [t], t: term()

Returns a list with the elements in List1 in reverse order, with tail Tail
appended.

Example:

    > lists:reverse([1, 2, 3, 4], [a, b, c]).
    [4,3,2,1,a,b,c]

iex(2)>
```
Here you go, in my desktop it took approximately 10 minutes to set all up and running.

Have a nice and productive day!
