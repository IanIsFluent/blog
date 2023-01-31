---
tags:
  - typescript
  - nodejs
---

## Typescript Node Playground

I want to be able to use Typescript as my shell, with the default node modules available. But with modern features like async await.

Starting a new Typescript project is a bit of a pain, so I wanted to create a playground to quickly try out new ideas.

[This repo](https://github.com/alinalihassan/top-level-await) was the best starting point i could find.

### To get started

```
git clone https://github.com/alinalihassan/top-level-await playground
cd playground
npm install
npm run dev
```

### Node modules

But even now this still won't let me use node modules! I added `const res = fetch('...')` and I got an error:

> ReferenceError: fetch is not defined

I had to run `npm i node` to fix this.

### Prettier

The only thing left now is to make sure your code looks nice - my standard .prettierrc is:

```json
{
  "tabWidth": 2,
  "useTabs": false,
  "printWidth": 120,
  "singleQuote": true,
  "semi": false,
  "trailingComma": "all"
}
```
