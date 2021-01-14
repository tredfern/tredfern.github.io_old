---
layout: page
title: Moonpie
permalink: /moonpie/
---

![Read the Docs](https://img.shields.io/readthedocs/moonpie)
[![Build Status](https://travis-ci.org/tredfern/moonpie.svg?branch=master)](https://travis-ci.org/tredfern/moonpie)
[![Coverage Status](https://coveralls.io/repos/github/tredfern/moonpie/badge.svg?branch=master)](https://coveralls.io/github/tredfern/moonpie?branch=master)
[![LOVE](https://img.shields.io/badge/L%C3%96VE-11.2-EA316E.svg)](http://love2d.org/)


## Motivation
I love working with [Love2d](http://love2d.org). I like that there was limited structure that allows for 
different approaches to developing the code. I like that it feels focused on providing a foundation without
an overly rigid structure.

And, whenever I work in it, I wanted a structure that worked for me. And that is where Moonpie evolved from.
Primarily it is focused around providing an approach to GUI that is _responsive_ and easy to craft user experiences.
As the UI matured I found I needed to bring more and more pieces together. Those pieces have evolved into this
framework.

## Framework

The moonpie framework includes:
* A UI library that renders out controls similar to React components. This allows for a modular and responsive layout that is modular and highly testable.
* A State-Management system similar to Redux that manages state through action messages that dispatch events. Those actions can be functions that can process multiple dispatches or tables contain simple data. Reducers handle the actions that trigger state changes.
* Utility functions such as a robust tables library, collection management, keyboard events
* Color library that contains many name colors