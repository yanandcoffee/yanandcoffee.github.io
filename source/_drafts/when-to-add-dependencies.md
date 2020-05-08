---
title: when-to-add-dependencies
tags:
- dependencies
---
If you remember I wrote about accessible smooth scrolling and focus management solutions. You can use libraries like the [jQuery's animate() function](https://css-tricks.com/snippets/jquery/smooth-scrolling/), or [react-scroll](https://www.npmjs.com/package/react-scroll) for that, but why?

You should be selective when adding additional dependencies, especially if you work on a large codebase, and it also depends on the scope of your work.

When you work at a big software company like Adobe, you have to check their licenses. Plus, you want to avoid extra dependencies whenever possible, because...
1) they mostly add syntactic sugar that your team would have to learn at some point, which adds time and effort on everyone's plate,
2) they also have dependencies that may be beyond your control (remember [left](https://www.theregister.co.uk/2016/03/23/npm_left_pad_chaos/)?),
3) you are relying on the maintainers to maintain the package for bugs and issues,
4) and lastly, you have to evaluate updates to the latest and fix any breaking changes.