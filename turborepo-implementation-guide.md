# Speeding Up Development with Turborepo: Using Turbo with Vite and Webpack

## Introduction

Modern frontend development often requires handling complex builds, large codebases, and multiple packages. This can lead to slow builds and inefficient workflows. **Turborepo** is a high-performance build system designed to speed up development by caching tasks and running them in parallel. This article explores how to use **Turborepo with Vite and Webpack** to enhance development speed and efficiency.

---

## What is Turborepo?

Turborepo is a monorepo build system that optimizes project workflows by providing:

- **Task Caching**: Prevents redundant work by storing previous task results.
- **Parallel Execution**: Runs multiple tasks at once to speed up builds.
- **Incremental Builds**: Only rebuilds parts of the project that have changed.
- **Optimized CI/CD Workflows**: Reduces build times in continuous integration pipelines.

Originally designed for **monorepos**, Turborepo can also be used for **single projects** to enhance performance.

---

## Setting Up Turborepo with Vite

### 1. Install Turborepo

To use Turborepo in a Vite project, install it as a development dependency:

```sh
npm install turbo --save-dev
# or
yarn add turbo --dev
# or
pnpm add turbo --save-dev
```

### 2. Configure Turborepo

Create a `turbo.json` file in the root directory:

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".vite", "dist"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

### 3. Add `packageManager` to `package.json`

Turborepo requires an explicit package manager version. Add the following field to your `package.json`:

#### For npm:

```json
"packageManager": "npm@9.9.0"
```

(Replace `9.9.0` with your actual npm version.)

#### For pnpm:

```json
"packageManager": "pnpm@8.6.0"
```

#### For yarn:

```json
"packageManager": "yarn@1.22.19"
```

### 4. Update Scripts in `package.json`

Modify your scripts to use Turborepo:

```json
{
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "lint": "turbo run lint",
    "test": "turbo run test"
  }
}
```

### 5. Run Vite with Turbo

Start the development server using:

```sh
npx turbo run dev
```

For production builds:

```sh
npx turbo run build
```

---

## Using Turborepo with Webpack

### 1. Install Turborepo

For Webpack-based projects, install Turbo:

```sh
npm install turbo --save-dev
```

### 2. Configure `turbo.json`

Create a `turbo.json` file:

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

### 3. Add `packageManager` to `package.json`

Ensure Webpack also has the correct package manager version:

```json
"packageManager": "npm@9.9.0"
```

(Adjust the version as needed.)

### 4. Update Webpack Scripts

Modify `package.json` scripts:

```json
{
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "lint": "turbo run lint",
    "test": "turbo run test"
  }
}
```

### 5. Run Webpack with Turbo

Start Webpack development server:

```sh
npx turbo run dev
```

For production builds:

```sh
npx turbo run build
```

---

## Benefits of Using Turbo with Vite and Webpack

| Feature                | Without Turbo                     | With Turbo                          |
| ---------------------- | --------------------------------- | ----------------------------------- |
| **Build Speed**        | Slower builds, full recompilation | **Smart caching & parallel builds** |
| **Incremental Builds** | Full project rebuild every time   | **Only changed files are rebuilt**  |
| **Parallel Execution** | Limited parallelism               | **Runs multiple tasks at once**     |
| **CI/CD Optimization** | Each build runs separately        | **Reuses cached builds in CI/CD**   |

---

## Conclusion

By integrating **Turborepo with Vite and Webpack**, you can significantly improve build performance, reduce redundant work, and accelerate development. Whether working on a **monorepo** or a **single project**, Turbo provides caching, parallel execution, and optimized workflows that make modern frontend development faster and more efficient.

Would you like to explore **Turborepo for full-stack applications**? Let us know in the comments!
