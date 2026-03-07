---
title: "TIL: Just CLI supports modules"
date: 2026-03-06T20:00:00+01:00
slug: til-just-cli-supports-modules
categories:
  - programming
tags:
  - TIL
  - tooling
---

The `just` CLI is a rust-based `make` clone concentrating on
running tasks, similar to its [ruby counterpart
rake](https://ruby.github.io/rake/) and its [python counterpart
snakemake](https://snakemake.readthedocs.io/en/stable/).
For my day-to-day use, I prefer `just` because of its speed
and simplicity -- especially in comparison to `make`, setting
up a new `justfile` in a project comes easy and it's hard to
make mistakes.

Recently, I found out that [`just` supports
modules](https://just.systems/man/en/modules1190.html). My
main use-case for this is to have a "root" `justfile` in a
monorepo that contains all the tasks for the project.

For example, think about a monorepo containing a `terraform`
folder with modules for `prod` and `dev` environments and
a `services` folder which in turn contains submodules
containing code for microservices and a webserver application.
Spinning up a dev server for one of the microservices
or applying new env var values on the `prod` environment
can be done with a single `just` command from the root
directory.

The only caveat I've found so far is that to list all
recipes from included modules, you need to add the flag
`--list-submodules` _in addition to the `-l` option_.
This is true even if the default target is `just -l`!

Of course, it's easy to circumvent this by using `just -l --list-submodules`
as the default target but I'm not sure whether there
are valid cases where the default target should be something else.
