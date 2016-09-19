---
layout: post
title:  "OCaml Module Namespaces"
date:   2016-08-31 18:35:46 -0400
comments: true
---
Here's a fun thing I learned about OCaml today: OCaml's linker has issues with module namespaces. Below is my understanding of the issue, let me know if I'm mistaken! This is a hard-to-diagnose issue if you don't know what to look for, so hopefully this saves someone some time debugging.

In following snippet where `A` is an external module dependency,

```
...
open A
open M
...
```

it's not clear whether `M` is external or internal to the module `A` (i.e. `A.M`) if `A`'s signature has not yet been compiled. So, `ocamldep` must treat each module name as a potential extern reference. This can yield an overapproximation of dependencies in benign cases, but can even cause compilation to fail in the case of falsely-diagnosed cyclic dependencies.

At the root of the problem, compilation units in OCaml are represented by a module whose name is derived from the basename of the compunit's files. This means if two independently developed libraries each have compilation units sharing the same representative module name, the two libraries cannot be reliably used in the same program.

What's the error look like when you unknowingly come across this issue? Well, that depends on how the failure manifests itself in your program. Here's the scenario that led me down this path:

```
...
open Async.Std
...
(* Do some OCaml things... *)
...
   Monitor.try_with (fun () -> <code that forks a child process>)
   >>| function
   | Error exn ->
     <Do some things and eventually core dump>
   | Ok () -> ()
...
```

My build system links in the async package from JaneStreet, and my src directory includes a file named `monitor.ml`. My src-tree-local compunit for `monitor.ml` produces the `Monitor` module, and this wreaks subtle havoc on OCaml's linker. Compilation succeeds, but at runtime my executable spawns child processes, continually core dumping, until my machine runs out of memory and comes to a grinding halt. Renaming `monitor.ml` is all it takes to solve the issue if you are less fortunate than I was in figuring out the cause of the problem.

What can you do?
===

It turns out you see a lot of OCaml projects with long, hopefully-unique prefixes on all of their filenames. This ends up looking like

```
...
MyPkg_list.ml
MyPkg_set.ml
MyPkg_map.ml
...
```

for every variation of `MyPkg` that reimplements a similarly named module. This doesn't solve the problem, it just (hopefully) avoids it with awkward, encumbered filenames, with no guarantee that you're the only one authoring a package title `TheBestPkgEver`. You also can't link in two different versions of the same library with this approach if that's something you want to do.

Alternatively, OCaml offers 'packed' modules. Conceptually, this is like giving all your modules unique prefixes and then packaging them into one monolithic module that you link against. Different packs used in the same program can contain modules of the same name, at the cost of compilation time, binary bloat, and slow incremental builds because you are linking in/recompiling everything, regardless of whether or not it is being used or was changed.

As of OCaml 4.02, there's at least some relief to this problem. Whereas previously

```
module List = Core_kernel_list
```

copied the entire module into your compilation unit, it now aliases the module in reference instead. This is what happens when you `open` an external module, and now it comes with a lot less burden.

Most of my understanding of this issue is due to Yaron Minsky's [blogpost](https://blogs.janestreet.com/better-namespaces-through-module-aliases/) on the topic, [a proposal](http://gallium.inria.fr/~scherer/namespaces/spec.pdf) for an improved handling of namespaces in OCaml, and [this](http://lists.ocaml.org/pipermail/platform/2013-March/000213.html) thread on the OCaml mailing list. A huge thanks goes out to all of the people involved in those discussions.

+ [1] [http://lists.ocaml.org/pipermail/platform/2013-March/000213.html](http://lists.ocaml.org/pipermail/platform/2013-March/000213.html)
+ [2] [http://gallium.inria.fr/~scherer/namespaces/spec.pdf](http://gallium.inria.fr/~scherer/namespaces/spec.pdf)
+ [3] [https://blogs.janestreet.com/better-namespaces-through-module-aliases/](https://blogs.janestreet.com/better-namespaces-through-module-aliases/)