
# 🔍 Exploring Binary Files with `dumpbin`, `readelf`, and `objdump`

Binary executables, whether in Windows (`.exe`, `.dll`) or Linux (`.out`, `.so`, etc.), follow specific formats that define how they are loaded and executed. To inspect and understand these binary files, especially for debugging, security auditing, or reverse engineering, we use specialized tools.

In this article, you'll learn how to:
- Use **`dumpbin`** on Windows to inspect PE (Portable Executable) files
- Use **`readelf`** and **`objdump`** on Linux to inspect ELF (Executable and Linkable Format) files
- Understand their roles and compare their capabilities

---

## 📌 Why Analyze Binary Files?

Understanding binary files is useful for:
- Debugging and troubleshooting application errors
- Identifying symbol names and addresses
- Checking dependencies (linked libraries)
- Validating structure and headers
- Security analysis and reverse engineering
- Ensuring correct compilation (e.g. architecture, entry point)

---

## 🪟 `dumpbin` on Windows

### 📘 What Is It?
`dumpbin` is a command-line tool included with **Microsoft Visual Studio Developer Tools**. It allows inspection of PE-format binaries such as `.exe`, `.dll`, `.obj`, and `.lib`.

### 🛠️ How to Use

Open the **Developer Command Prompt for Visual Studio** and run:

```bash
dumpbin /HEADERS yourfile.exe
dumpbin /EXPORTS yourlib.dll
dumpbin /IMPORTS yourfile.exe
dumpbin /SYMBOLS yourobj.obj
```

### 🔧 Common Options

| Option         | Description                            |
|----------------|----------------------------------------|
| `/HEADERS`     | Displays PE header and section headers |
| `/EXPORTS`     | Lists exported functions (DLLs)        |
| `/IMPORTS`     | Lists imported DLLs and functions      |
| `/SYMBOLS`     | Displays symbol table (for `.obj` or `.lib`) |
| `/DEPENDENTS`  | Shows dependent DLLs                   |
| `/ALL`         | Displays all available information     |

---

## 🐧 `readelf` and `objdump` on Linux

Linux binaries are in **ELF (Executable and Linkable Format)**. Tools like `readelf` and `objdump` let you analyze their contents in depth.

---

### 📘 `readelf` — The Direct ELF Inspector

#### 🛠️ Usage Examples:

```bash
readelf -h mybinary           # ELF file header
readelf -S mybinary           # Section headers
readelf -l mybinary           # Program headers
readelf -s mybinary           # Symbol table
readelf -d mybinary           # Dynamic section (shared libs)
```

#### 📌 Highlights
- Fast and focused on ELF-specific structures
- Doesn't disassemble code — purely structural

---

### 📘 `objdump` — Disassembler + Inspector

#### 🛠️ Usage Examples:

```bash
objdump -x mybinary           # All headers and info
objdump -t mybinary           # Symbol table
objdump -d mybinary           # Disassembly (machine code)
objdump -p mybinary           # Dynamic section
```

#### 📌 Highlights
- More verbose than `readelf`
- Includes disassembly of code sections
- Supports multiple binary formats

---

## 🔍 Comparing Tools

| Feature                | `dumpbin` (Windows) | `readelf` (Linux) | `objdump` (Linux) |
|------------------------|---------------------|-------------------|-------------------|
| Format supported       | PE                  | ELF               | ELF + PE (with MinGW) |
| Header inspection      | ✔️                  | ✔️                | ✔️                |
| Symbol table           | ✔️                  | ✔️                | ✔️                |
| Imports/Exports        | ✔️                  | ✔️ (via `-d`)     | ✔️                |
| Disassembly            | ❌                 | ❌                | ✔️                |
| GUI                    | ❌                 | ❌                | ❌                |

---

## 🧠 Summary

Whether you're a systems programmer, reverse engineer, or just curious about how binaries work, these tools are essential:

- Use **`dumpbin`** on Windows to inspect PE headers, symbols, and DLL dependencies.
- Use **`readelf`** on Linux for accurate and structured ELF inspection.
- Use **`objdump`** when you need disassembly or a deeper look at symbols and sections.

By understanding what these tools reveal, you gain visibility into what compilers produce and how operating systems load and execute your code.
