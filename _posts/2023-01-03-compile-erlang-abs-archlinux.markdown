---
layout: post
title:  "Compiling Erlang with docs using ABS in Archlinux"
date:   2023-01-03 18:34:40 +0300
categories: vim erlang otp archlinux
---
Since I've started using `vim` with [elixir-ls](https://github.com/elixir-lsp/elixir-ls) language
server, I've noticed that Archlinux erlang package is built without docs, so I couldn't use my
LSP-completer in full power. That prompted me to investigate the [Arch Build System](https://wiki.archlinux.org/title/Arch_Build_System), `a ports-like
system for building and packaging software from source code`.

Here's how I managed to compile Erlang with docs using ABS.

#### Install `asp` tool

```bash
$ sudo pacman -S asp
```

`asp` is a bash script that aids in the management of ABS package sources. 

#### Setup

```bash
$ mkdir ~/src/abs
$ cd ~/src/abs
$ asp export erlang
$ cd erlang
```

`asp export` will get latest version of the package from repo into directory with the same name.

#### Make changes in `PKGBUILD` file

```diff
diff --unified
--- /path/to/old/PKGBUILD      2023-01-03 18:04:48.769418824 +0200
+++ PKGBUILD    2023-01-03 17:58:57.291392260 +0200
@@ -16,6 +16,7 @@
 license=(Apache)
 makedepends=(fop git glu java-environment libxslt lksctp-tools mesa perl unixodbc wxwidgets-gtk3)
 options=(staticlibs)
+groups=(modified)
 source=(epmd.conf
         epmd.service
         epmd.socket
@@ -44,6 +45,7 @@
     --prefix=/usr \
     --with-odbc
   make
+  make docs DOC_TARGETS=chunks
 }

 package_erlang() {
@@ -57,6 +59,7 @@

   export PATH="$srcdir/bin:$PATH"
   make -C otp DESTDIR="$pkgdir" install
+  make -C otp DESTDIR="$pkgdir" DOC_TARGETS=chunks install-docs

   # move files that belong to the erlang-unixodbc package
   mkdir -p unixodbc
```

By making those changes, we add the generated package to the 'modified' group (you'll see later why)
while also instructing `makepkg` to build and install Erlang documentation.

Next, run:

```bash
$ makepkg -si
```

`-si` flags ensure that `makepkg` retrieves needed dependencies and auto-installs the package after
compilation.

#### Add this line into `/etc/pacman.conf`

```code
IgnoreGroup = modified
```

That's why I've added the package to the 'modified' group:

> If new versions are available in the official repositories during a system update, pacman prints 
> a note that it is skipping this update because it is in the IgnoreGroup section. At this point, 
> the modified package should be rebuilt from ABS to avoid partial upgrades.
