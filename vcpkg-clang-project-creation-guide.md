# 📦 How to Create a Clang C Project Using vcpkg and CMake

## 🛠 Prerequisites

Make sure the following tools are installed on your system:

### ✅ Required Tools

- **CMake** (>= 3.15)
- **vcpkg**
- **Clang** (or your preferred C compiler)

### 🪟 Windows Installation Tips

You can use [Scoop](https://scoop.sh/) or [Winget](https://learn.microsoft.com/en-us/windows/package-manager/winget/) to install dependencies:

#### Using **Scoop**

```sh
scoop install cmake llvm vcpkg
```

> ℹ️ If `scoop` is not installed yet, follow instructions at: https://scoop.sh/

#### Using **Winget**

```sh
winget install Kitware.CMake
winget install LLVM.LLVM
winget install Microsoft.Vcpkg
```

---

## 🧱 Step-by-Step Setup

### 1️⃣ Initialize a New vcpkg Project

```sh
vcpkg new --application
```

This will generate the basic folder structure for your application.

---

### 2️⃣ Create `CMakeLists.txt` and `CMakePresets.json`

Use this preconfigured template:

📄 **Get the template files:**

```txt
https://gist.github.com/marcuwynu23/ac99687142e176d6b024fe7d796907e6
```

> Includes:
>
> - Clang setup
> - vcpkg integration
> - Recursive source file gathering
> - Clean and documented structure

---

### 3️⃣ Configure the Project

This command will run CMake and automatically trigger vcpkg installation for any missing dependencies.

```sh
cmake --preset=default
```

---

### 4️⃣ Build the Project

```sh
cmake --build build
```

---

## ✅ Done!

You now have a functioning Clang C project with modern CMake and vcpkg integration. 🎉
