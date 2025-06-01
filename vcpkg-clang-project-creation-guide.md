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

make a project folder and move inside it.

```sh
mkdir vcpkg-project && cd vcpkg-project
```

Initialize the vcpkg project

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

Create `src` then create file .cpp or c

```sh
mkdir src && touch src\main.c # or src\main.cpp
```

you need also to update the CMakelists.txt
in this part:

```txt
file(GLOB_RECURSE SOURCES "src/*.c")
```

This command will run CMake and automatically trigger vcpkg installation for any missing dependencies.

```sh
cmake --preset=default
```

---

### 4️⃣ Build the Project

```sh
cmake --build build
```

### adding dependencies

if you want to add dependency for example fmt
you use this `vcpkg add port <dependency>`:

```sh
 vcpkg add port fmt
```

then you need to run

```sh
cmake --preset=default
```

then find in log this section:

```txt
The package fmt provides CMake targets:
```

then get the below part like this:

```txt
    find_package(fmt CONFIG REQUIRED)
    target_link_libraries(main PRIVATE fmt::fmt)

    # Or use the header-only version
    find_package(fmt CONFIG REQUIRED)
    target_link_libraries(main PRIVATE fmt::fmt-header-only)
```

then add it in CMakelists.txt in `Linking External Libraries (via vcpkg)` section:

after adding you can now run:

```sh
cmake --build build
```

### how to remove dependencies

To remove depedency is by commentout or removing the
part that you add in CMakelists.txt `Linking External Libraries (via vcpkg)` section:

then remove in vcpkg.json:

```json
  "dependencies": [
    "fmt" // remove this
  ]
```

after that, run the `cmake --preset=default` and `cmake --build build`

---

## ✅ Done!

You now have a functioning Clang C project with modern CMake and vcpkg integration. 🎉

```

```
