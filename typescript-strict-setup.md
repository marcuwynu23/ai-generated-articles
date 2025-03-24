# **Setting Up TypeScript for Maximum Strictness: No `any`, No `@ts-ignore`**  

When working on a TypeScript project, ensuring full type coverage is crucial for maintaining code quality and reducing potential runtime errors. This article will guide you through configuring TypeScript and ESLint to enforce the strictest possible rules—disallowing `any`, `@ts-ignore`, and other potential type loopholes.  

---

## **1. Why Enforce Strict TypeScript Rules?**  

By default, TypeScript provides some flexibility in type definitions, allowing developers to use `any` and `@ts-ignore` to bypass type checking. While this might seem convenient, it leads to:  

- **Loss of Type Safety** – Errors may not be caught during development.  
- **Reduced Code Maintainability** – Future developers (including yourself) may struggle to understand weakly-typed code.  
- **Harder Debugging** – Issues that could have been prevented by static analysis make their way to runtime.  

A strict TypeScript setup ensures that every variable, function, and class follows proper type definitions, making your code more reliable and maintainable.  

---

## **2. Configuring TypeScript for Maximum Strictness**  

### **Step 1: Creating a `tsconfig.json` File**  

In your project root, create a `tsconfig.json` file with the following configuration:  

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Node",
    "strict": true,
    "noImplicitAny": true,
    "noImplicitThis": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "allowUnreachableCode": false,
    "allowUnusedLabels": false,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitOverride": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noPropertyAccessFromIndexSignature": true,
    "useUnknownInCatchVariables": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": false,
    "allowJs": false,
    "checkJs": false,
    "declaration": true,
    "emitDeclarationOnly": false,
    "esModuleInterop": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### **Key Features of This Configuration:**  

- **`strict`: true** – Enables all strict type-checking options.  
- **`noImplicitAny`: true** – Prevents implicit `any` types.  
- **`noImplicitOverride`: true** – Forces explicit overrides for class methods.  
- **`noUncheckedIndexedAccess`: true** – Ensures all indexed properties are properly checked.  
- **`exactOptionalPropertyTypes`: true** – Prevents optional properties from being assigned `undefined` implicitly.  
- **`noPropertyAccessFromIndexSignature`: true** – Restricts property access without proper type definition.  
- **`useUnknownInCatchVariables`: true** – Encourages proper error handling instead of assuming `any` in `catch` blocks.  

This setup ensures **no weak typing** sneaks into your project, making it fully type-safe.  

---

## **3. Enforcing TypeScript Rules with ESLint**  

Even with strict TypeScript rules, developers might attempt to bypass them using `@ts-ignore`. To ensure no exceptions are made, we must configure ESLint properly.  

### **Step 1: Install ESLint Dependencies**  

Run the following command to install ESLint and TypeScript plugins:  

```sh
npm install --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

### **Step 2: Create an `.eslintrc.json` File**  

Add the following configuration to your project root:  

```json
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/strict"
  ],
  "rules": {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/ban-ts-comment": "error",
    "@typescript-eslint/explicit-function-return-type": "error",
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }]
  }
}
```

### **Key Features of This ESLint Configuration:**  

- **`@typescript-eslint/no-explicit-any`: "error"`** – Disallows `any` entirely.  
- **`@typescript-eslint/ban-ts-comment`: "error"`** – Prevents `@ts-ignore` and related comments.  
- **`@typescript-eslint/explicit-function-return-type`: "error"`** – Requires explicit return types for functions.  
- **`@typescript-eslint/no-unused-vars`: "error"`** – Ensures unused variables are not left in the code.  

---

## **4. Running ESLint to Enforce Rules**  

Once everything is set up, you can run ESLint on your TypeScript files with:  

```sh
npx eslint src --ext .ts
```

If any violations are found (such as an `any` type or `@ts-ignore`), ESLint will flag them and prevent them from being committed.  

---

## **5. Conclusion: Writing Fully Typed TypeScript Code**  

By setting up a strict TypeScript configuration and enforcing it with ESLint, you ensure:  

✅ **No implicit `any` types**  
✅ **No use of `@ts-ignore` or other TypeScript comment hacks**  
✅ **Explicit return types for functions**  
✅ **Unused variables are not left in the code**  
✅ **Fully type-safe, maintainable, and future-proof code**  

Following this setup will make your TypeScript codebase robust and eliminate potential runtime errors before they happen.  

Now you’re ready to build a TypeScript project that enforces strict type safety at every level!
