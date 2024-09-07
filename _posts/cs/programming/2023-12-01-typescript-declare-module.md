In TypeScript, the `declare module` syntax is used to tell the compiler about the shape of an external module or library for which there are no TypeScript type definitions. This is often used when working with JavaScript libraries or modules that don't provide TypeScript support out of the box.

Let's go through an example of declaring a module in a library and then show how a user can extend that functionality. Finally, I'll explain how TypeScript merges code together in the background.

### Example 1: Declaring a Module in a Library

Suppose you have a JavaScript library named `mathLibrary` with the following code:

```javascript
// mathLibrary.js
function add(a, b) {
  return a + b;
}
```

To use this in TypeScript and provide type information, you can declare a module for it:

```typescript
// mathLibrary.d.ts
declare module "mathLibrary" {
  function add(a: number, b: number): number;
}
```

### Example 2: Extending the Functionality

Now, let's say a user wants to extend the functionality of this library by adding a new function:

```typescript
// userExtensions.d.ts
declare module "mathLibrary" {
  function subtract(a: number, b: number): number;
}
```

### TypeScript Code Merging

When TypeScript encounters multiple declarations for the same module, it performs code merging. The declarations are combined to form a single merged declaration.

If a user imports the `mathLibrary` and uses both the original functionality and the user extension:

```typescript
// app.ts
import * as math from "mathLibrary";

const result1 = math.add(10, 5); // Original functionality
const result2 = math.subtract(10, 5); // User extension

console.log(result1); // 15
console.log(result2); // 5
```

In this case, TypeScript internally merges the declarations from `mathLibrary.d.ts` and `userExtensions.d.ts`, resulting in a merged module declaration that includes both the `add` and `subtract` functions.

The resulting merged declaration in the background would look something like this:

```typescript
// Merged declaration in the background
declare module "mathLibrary" {
  function add(a: number, b: number): number;
  function subtract(a: number, b: number): number;
}
```

This way, TypeScript allows users to extend the types and functionality of existing modules or libraries seamlessly through declaration merging.