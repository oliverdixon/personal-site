---
title: "Emscripten Callbacks"
date: 2026-08-05T12:14:18+01:00
draft: true
---

## Motivation

A core design principle of my [EchoMap](https://github.com/oliverdixon/echomap) DSP workbench tool is that it must
function identically, as much as practicably possible, on native platforms as it does on web platforms. As primarily a
C++ developer accustomed to targeting "normal" environments, such as on POSIX-compliant desktop platforms, the biggest
challenge of implementing that principle is figuring how to handle cases that only arise when the application is running
on WebAssembly.

Many actions that are straight-forward on native platforms require some non-trivial additional work on WebAssembly
platforms, and no action illustrates this more clearly than interacting with the filesystem. That's a pain for EchoMap,
because it uses files extensively.

Here's the setup:

* Users encode their work into projects. Projects store everything about the workspace: the signals, the sensors, the
  mappings between signals and sensors, all non-ephemeral metadata, cached DSP results, etc.
* Projects can be serialised (saved) and de-serialised (loaded) into a JSON file, which is ostensibly stored on the
  user's filesystem for persistent storage.
* While trivial signals can be stored as time-series data directly in the JSON file, in reality, most waveform data is
  held in an external file whose path is referenced by the project file, along with any additional metadata.

On the native platform, implementing the save-and-load workflow is easy:

1. The user creates a blank project and specifies their signals, sensors, and defines mappings between the two. They may
   then perform some analysis on the data, such as simple DSP routines (taking the discrete Fourier transform, computing
   the autocorrelation, downsampling the wave); or, they may do something more elaborate, such as using sensor
   positional data to perform sound-source localisation on a series of signals.
2. Once the user is happy with their project, they go to save the project to persistent storage. The serialisation
   driver produces the representative JSON and saves it to the filesystem.
