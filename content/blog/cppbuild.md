---
title: "Build Environments for Personal C++ Projects"
date: 2026-08-04T09:21:22+01:00
draft: true
---

## Motivation

It's not surprising that, as a language built with 40 years of hindsight, the first stable release of the Rust
programming language shipped with a default package manager from day 1. Unfortunately, C++ programmers don't have a
luxurious and universally standard ecosystem that surrounds our language; as far as standardisation is concerned, we
have [ISO/IEC 14882](https://www.iso.org/standard/83626.html), and that's about it. Everything supporting the needs of
the modern developer is a patchwork of variably documented consensus of what is considered "good practice". Even more
frustrating is that our understanding of "good practice" moves with the times, and we often repeatedly fall into traps
caused by problems that have been solved decades ago.

A common pain point for software engineers moving from building small, self-contained projects to working within larger
systems (possibly in a first internship or graduate job) is that their ability to write good code is bottlenecked by
their inability to participate in the project environment. They will probably be able to describe and use the SOLID
principles—maybe they can even debug memory errors and investigate performance regressions with Valgrind—but knowledge
of good engineering practice isn't very relevant when
[you can't even get the thing to build](https://github.com/Hello-World-EE/Java-Hello-World-Enterprise-Edition).
Universities have started to address this in their respective curricula, dedicating entire modules to
["The Real World"](https://www.york.ac.uk/students/studying/manage/programmes/module-catalogue/module/COM00019I/latest),
but most CS graduands with their vast GitHub portfolios still treat development and build infrastructure as secondary
because configuring a decent environment is overkill for a small project, right?

This post provides a blueprint of a sensible, minimal set of tools and configuration to complement SWE processes for
small, simple projects. I'll elaborate that by saying this blueprint works well when:

* You don't have a huge number of complex co-dependent build-time and runtime dependencies.
* You don't need distributed builds.
* There's no extensive polyglossia in your project.
* You're building in some reasonably standard environment: no air-gapped networks, no elaborate compliance and reporting
  requirements, no 2B SLOC mono-repos.

I wager that some number within a rounding error of 100% of early-career developers and students satisfy the above
criteria. So, if you're within that 100%: this article is for you. Try it out, pick and choose, and use some of the
hints to make your projects look more professional.

## The Build System

I used to manually write Makefiles and Ninja scripts by hand. Don't be like me.

Make, Ninja, and countless others are excellent for supporting incremental compilation, but they're just that. Unless
you have the time, expertise, and need to write a custom layer on top of the lower-level tools, such
as [Kbuild](https://docs.kernel.org/kbuild/kconfig-language.html), it's almost always better to let a more generic tool
handle the implementation details. You want your build to be tailored to the host build system, but writing a portable
build script serialiser probably requires
[more work than most developers appreciate](https://github.com/acli/trn/blob/613a7e97aca06dd807fb225990fa804e8c744574/Configure)
(bear in mind that this was required for portable tooling in 1984, so imagine what it would be like now).

### Which One?

For personal projects, my default recommendation is CMake. It is not inherently better than competitors, especially
tooling geared more towards enterprise environments such as [Blaze/Bazel](https://bazel.build/), but its support in all
major IDEs is excellent. It is infinitely configurable, has a familiar imperative-esque specification language, and can
be customised declaratively with JSON. Its integration with third-party tools beside IDEs is also first-class: Conan,
CTest, packaging tools, and CI providers all know how to talk CMake. It also has a simple command-line interface for the
occasional time you need to fire off an impromptu build over SSH.

### A Minimal Working Example

This post isn't trying to duplicate the existing excellent CMake documentation, but here's a brief primer on your first
CMake project, if you've never used it before.

## Dependencies

## Compiler Toolchains

## Formatting and Static Analysis

## Testing

## Documentation
