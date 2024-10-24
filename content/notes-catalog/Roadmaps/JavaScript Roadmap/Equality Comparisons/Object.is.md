---
id: 01JAYEA4M76VV9Y15D5W14H5FF
title: Object.is
modified: 2024-10-24T01:01:22-04:00
tags:
  - javascript
  - roadmaps
  - programming
---
# Object.is

The Object.is() static method determines whether two values are the same value.

```
console.log(Object.is('1', 1));
// Expected output: false

console.log(Object.is(NaN, NaN));
// Expected output: true

console.log(Object.is(-0, 0));
// Expected output: false

const obj = {};
console.log(Object.is(obj, {}));
// Expected output: false
```

Visit the following resources to learn more:

Free Resources

---

- [ArticleObject.is() - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)