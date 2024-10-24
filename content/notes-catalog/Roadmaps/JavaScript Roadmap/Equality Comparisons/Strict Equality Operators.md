---
id: 01JAYEA4M76VV9Y15D5W14H5FF
title: Strict Equality Operators
modified: 2024-10-24T01:00:50-04:00
tags:
  - javascript
  - roadmaps
  - programming
---
# Strict Equality Operator (===)

In JavaScript, the strict equality operator `===` compares both the value and the type of two operands. This means that it will only return true if both the value and the type are identical.

```
"5" === "5"   // true
```

In this case, both the value and the type are the same, so the result is true.

```
"5" === 5   // false
```

Here, although the values might appear similar, the types are different (string and number), so the result is false. The strict equality operator does not perform type coercion; both the value and the type must be identical.

Learn more from the following resources:

Free Resources

---

- [ArticleStrict equality - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Strict_equality)