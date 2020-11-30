---
layout: post
aliases: [/post/114514280452/a-few-core-animation-gotchas-that-got-me-last]
vimeo: 123117677
title: Core Animation Gotchas
date: 2015-03-24T15:31:10-00:00
tags: [core-animation, ios-dev, teaching]
---

A few Core Animation Gotchas that got me last night while giving a demo to my students.

Recap:

- The top level layer in a CALayer hierarchy does not animate its property changes. This is because animation must exist in a container.
- Core Animation cannot animate a nil _from_ or _to_ value. Some CALayer properties are nil by default. (Like backgroundColor.)
