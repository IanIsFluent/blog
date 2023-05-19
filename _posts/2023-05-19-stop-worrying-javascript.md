---
tags:
  - javascript
  - typescript
  - vscode
---

# How I learned to stop worrying and love javascript (again)

I love typescript. It helps you to avoid typos, and ensures you don't pass properties into the wrong functions, as well as helping expose possible issues with nulls. It's amazing, and i want to spend all my time writing it.

The only problem with typescript is that it is a _lot_. You need your tsconfig and you need the module settings right to allow top level await, as well as the step to actually convert your typescript code to javascript.

## The alternative

I had forgotten than VSCode does quite a lot for you even without typescript. It will infer types in just the same way as typescript - the difference is you can't help it with explicit type declarations.

But it turns out that isn't true! I didn't realise until [this drama](https://twitter.com/Rich_Harris/status/1350436286948122625) in the javascript framework world that with [JSDoc](https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html), it turns you you still can!

## An easier way to hack

Creating a new typescript project to hack some code and explore a problem space was such a hassle I wrote a [blog post](/blog/2023/01/28/ts-node-playground.html) about it. But now, I can just create a new javascript file, and start hacking away. If I want to add type info, I can start adding a JSDoc comments.
