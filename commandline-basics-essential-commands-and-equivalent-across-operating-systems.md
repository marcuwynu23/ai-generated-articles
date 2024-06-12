# **Command Line Basics**

Command Line Interfaces (CLI) provide a powerful way to interact with computers, enabling users to perform various tasks efficiently through text-based commands. While CLI may seem intimidating at first, mastering its basics can greatly enhance your productivity and proficiency in managing your system. In this article, we'll explore essential command line commands and their equivalents across three major operating systems: Windows, Linux, and macOS.

| Stream Name              | Symbol   | Description                                                                        |
| ------------------------ | -------- | ---------------------------------------------------------------------------------- |
| Standard Input (stdin)   | `stdin`  | Default input stream where commands read input. Usually connected to the keyboard. |
| Standard Output (stdout) | `stdout` | Default output stream where commands print normal output.                          |
| Standard Error (stderr)  | `stderr` | Stream used for error messages and diagnostic output.                              |

**1. Navigation Commands:**

| Command | Linux/macOS                | Windows             | Description             |
| ------- | -------------------------- | ------------------- | ----------------------- |
| `pwd`   | `pwd`                      | `cd`                | Print working directory |
| `ls`    | `ls [options] [directory]` | `dir [options]`     | List directory contents |
| `cd`    | `cd [directory]`           | `cd [directory]`    | Change directory        |
| `mkdir` | `mkdir [directory]`        | `mkdir [directory]` | Make directory          |
| `touch` | `touch [file]`             | `echo > [file]`     | Create new file         |
| `rm`    | `rm [options] [file]`      | `del [file]`        | Remove file             |
|         | `rm -r [directory]`        | `rmdir [directory]` | Remove directory        |

**2. File Manipulation Commands:**

| Command   | Linux/macOS                     | Windows                          | Description                      |
| --------- | ------------------------------- | -------------------------------- | -------------------------------- |
| `mv`      | `mv [source] [destination]`     | `move [source] [destination]`    | Move or rename file or directory |
| `cp`      | `cp [source] [destination]`     | `copy [source] [destination]`    | Copy file or directory           |
| `cat`     | `cat [file]`                    | `type [file]`                    | Display file contents            |
|           | `cat [file1] [file2] > [file3]` | `copy [file1] + [file2] [file3]` | Concatenate files                |
| `nano`    | `nano [file]`                   | -                                | Text editor in CLI (Linux/macOS) |
| `notepad` | -                               | `notepad [file]`                 | Text editor in CLI (Windows)     |

**3. Permissions and Users:**

| Command | Linux/macOS           | Windows                                     | Description                           |
| ------- | --------------------- | ------------------------------------------- | ------------------------------------- |
| `ls -l` | `ls -l [file]`        | `icacls [file]`                             | Display file permissions              |
| `chmod` | `chmod [mode] [file]` | `icacls [file] /grant [user]:[permissions]` | Change file permissions (Linux/macOS) |
|         |                       | `takeown /f [file]`                         | Take ownership of file (Windows)      |

**4. Redirection and Piping:**

| Command | Linux/macOS | Windows | Description                            |
| ------- | ----------- | ------- | -------------------------------------- |
| `>`     | `>`         | `>`     | Redirect output to file                |
| `>>`    | `>>`        | `>>`    | Append output to file                  |
| `\|`    | `\|`        | `\|`    | Pipe output to another command or file |
| `<`     | `<`         | `<`     | Redirect input from file               |

These symbols provide various ways to handle input and output streams in the command line interface.

**5. Basic Command Line Utilities:**

| Command | Linux/macOS                        | Windows                                    | Description                                    |
| ------- | ---------------------------------- | ------------------------------------------ | ---------------------------------------------- |
| `grep`  | `grep [pattern] [file]`            | `Select-String [pattern] [file]`           | Search for text patterns (Linux/macOS)         |
| `find`  | `find [directory] -name [pattern]` | `Get-ChildItem -Recurse -Filter [pattern]` | Search for files and directories (Linux/macOS) |
| `wc`    | `wc [file]`                        | `Measure-Object -Line [file]`              | Word count (Linux/macOS)                       |

**6. Managing Processes:**

| Command | Linux/macOS  | Windows(CMD) | Windows(Powershell)      | Description            |
| ------- | ------------ | ------------ | ------------------------ | ---------------------- |
| `ps`    | `ps`         | `tasklist`   | `Get-Process`            | List running processes |
| `kill`  | `kill [PID]` | `taskkill`   | `Stop-Process -ID [PID]` | Terminate a process    |

Certainly! Here's an updated section including service management commands:

**7. Service Management:**

| Command   | Linux/macOS             | Windows(CMD) | Windows(Powershell)             | Description               |
| --------- | ----------------------- | ------------ | ------------------------------- | ------------------------- |
| `service` | `service or system-ctl` | `sc`         | `Start-Service or Stop-Service` | cli service Manager tools |

**8. Package Management:**

| Command | Linux/macOS   | Windows  | Description     |
| ------- | ------------- | -------- | --------------- |
| `apt`   | `apt/pkg/yum` | `winget` | Package Manager |

**9. Environment Variables:**

| Command | Linux/macOS | Windows       | Windows              | Description                              |
| ------- | ----------- | ------------- | -------------------- | ---------------------------------------- |
| `env`   | `env`       | `set`         | `Get-ChildItem Env:` | List environment variables (Linux/macOS) |
| `$env:` | `export`    | `set or setx` | `$env:`              | Access environment variables (Windows)   |
