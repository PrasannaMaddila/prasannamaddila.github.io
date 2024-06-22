---
layout: post
title: "Moving away from FreeBSD"
categories: code
---

Confession time. I've moved away from FreeBSD, since my workflow is catastrophically badly suited to it, and despite all my best efforts to port it, it requires a better developer with more time than I can spare.

## Python, Ray and Bazel

For my PhD, I've been using the [Ray(RLlib)](https://github.com/ray-project/ray/tree/master/rllib) library to work with Multi-agent Reinforcement Learning (coincidentally a huge theme in my thesis). Now, getting it to work on FreeBSD was a total nightmare, and here's the list of things I tried.

1. Launching it in a LinuxJail. While it installs, importing the library still fails because it doesn't find some low-level libraries.
2. Building it locally. GCC was the default compiler (hardcoded), so either installing it or changing the code to use `clang` wasn't a huge issue. However, the Bazel file doesn't find `python3.8` and `freebsd` as a supported platform. Not being an expert on Bazel, this quickly lead to a dead end.

I quickly dismissed using `bhyve` at this point, since if I was going to launch a VM on a 4 core machine every time I need to work, I might as well use Linux full-time.

## Pybind

I wanted to use Pybind to experiment with writing a Gymnasium environment in C++, and then exposing that to Python. Pybind, either when installed as a `pip` package in a virtual environment, or as a Git submodule, was not discovered by my programs, and I kept getting strange linker errors even when I specified the include directories. Considering that my other porting efforts were not going well, I was highly motivated to give up the chase at this moment.

## Fin de l'expérience 

For now, I've moved back to Debian where things work, but it's no longer that sweet UNIX simplicity. I might come back later when my workflow has changed, or when I upgrade my CPU so that I can throw more cores at it.
