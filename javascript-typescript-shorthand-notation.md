# Shorthand Notations in TypeScript and JavaScript

Shorthand notations provide a way to write more concise and readable code in TypeScript and JavaScript. Here are some of the most commonly used shorthand notations, along with examples and use cases.

| Shorthand | Description | Example | Use case |
| --- | --- | --- | --- |
| `+` | Converts a string to a number | `const idNumber = +id;` | When converting route parameters from strings to numbers |
| `?.` | Allows safe access to nested properties and methods | `const name = user?.address?.city;` | When accessing properties or methods that may be null or undefined |
| `??` | Provides a default value for null or undefined variables | `const name = username ?? 'Anonymous';` | When specifying a default value for optional parameters |
| `...` | Expands an array or object into individual elements | `const combinedArray = [...array1, ...array2];` | When combining arrays or objects, or when passing multiple arguments to a function |
| Destructuring assignment | Extracts values from complex objects or arrays | `const { name, age, address: { city } } = user;` | When extracting values from complex objects or arrays |
