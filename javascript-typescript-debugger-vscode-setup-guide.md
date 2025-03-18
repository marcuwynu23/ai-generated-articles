# **How to Set Up an External Terminal for Debugging in VS Code with Nodemon**

Debugging in **Visual Studio Code (VS Code)** is a powerful way to track down issues in your **Node.js** applications. However, by default, VS Code runs the debugger inside its integrated terminal, which may not always be ideal. If you want **an external terminal** for debugging, especially when using **Nodemon** to watch files and auto-restart the application, follow this guide.

## **Why Use an External Terminal for Debugging?**

Using an **external terminal** instead of VS Code's built-in terminal offers several advantages:

- **Better visibility** when debugging long-running processes.
- **Separate process handling**, making it easier to restart and debug multiple sessions.
- **Full terminal control**, allowing you to use shortcuts that might be blocked inside VS Code's integrated terminal.

---

## **Step 1: Install Nodemon**

Before setting up the debugger, make sure **Nodemon** is installed. If you haven’t installed it yet, run:

```sh
npm install -g nodemon
```

Or if you prefer a project-specific installation:

```sh
npm install --save-dev nodemon
```

---

## **Step 2: Configure Debugging in `launch.json`**

To use an **external terminal** and make it **watch all files**, update your **`launch.json`** file in **VS Code**.

### **Modify or Create `launch.json`**

1. Open **VS Code**.
2. Navigate to **Run and Debug (Ctrl+Shift+D)**.
3. Click on **"create a launch.json file"** if you don’t have one.
4. Use the following configuration:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug with External Terminal",
      "program": "${workspaceFolder}/yourFile.js",
      "restart": true,
      "console": "externalTerminal",
      "runtimeExecutable": "nodemon",
      "args": ["--inspect-brk", "--watch", "."],
      "internalConsoleOptions": "neverOpen",
      "env": {
        "FORCE_COLOR": "1"
      },
      "runtimeArgs": []
    }
  ]
}
```

---

## **Step 3: Understanding the Configuration**

Here's a breakdown of the key settings:

| **Key**                                     | **Description**                                                                     |
| ------------------------------------------- | ----------------------------------------------------------------------------------- |
| `"console": "externalTerminal"`             | Opens the debugger in an **external terminal** instead of VS Code's integrated one. |
| `"runtimeExecutable": "nodemon"`            | Runs the program using **Nodemon**, which allows auto-restarting on file changes.   |
| `"args": ["--inspect-brk", "--watch", "."]` | Enables debugging (`--inspect-brk`) and watches all files (`--watch "."`).          |
| `"internalConsoleOptions": "neverOpen"`     | Prevents the internal console from opening in VS Code.                              |
| `"restart": true`                           | Automatically restarts the debugger when the process stops.                         |

---

## **Step 4: Running the Debugger**

After setting up **`launch.json`**, you can start debugging:

1. Go to **Run and Debug (Ctrl+Shift+D)** in VS Code.
2. Select **"Debug with External Terminal"** from the dropdown.
3. Click **"Start Debugging" (F5)**.
4. A new external terminal will open, and your application will start with Nodemon.
5. If you make changes to any file, **Nodemon will detect the change and restart the app automatically**.

---

## **Step 5: Stopping the Previous Debug Session (Optional)**

By default, Nodemon restarts your app but does not close the previous Node.js instance. To **automatically close the previous session** before restarting, modify `launch.json` like this:

```json
"nodemonConfig": {
  "exec": "taskkill /F /IM node.exe & node"
}
```

This will **forcefully terminate the old Node.js process before launching a new one**.

---

## **Step 6: Debugging Tips**

- **Set breakpoints** in your code by clicking next to the line numbers in VS Code.
- **Use the Debug Console** to inspect variables and step through execution.
- **Use the external terminal** for better control over process execution.

---

## **Debugging TypeScript with Nodemon and TSX**

If you are using **TypeScript**, update your `launch.json` to use `tsx`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug TypeScript with External Terminal (tsx)",
      "runtimeExecutable": "nodemon",
      "args": [
        "--inspect-brk",
        "--watch",
        ".",
        "--ext",
        "ts,tsx",
        "--exec",
        "tsx src/index.ts"
      ],
      "console": "externalTerminal",
      "internalConsoleOptions": "neverOpen",
      "env": {
        "FORCE_COLOR": "1"
      },
      "restart": true
    }
  ]
}
```

This ensures that Nodemon watches `.ts` and `.tsx` files and executes them using `tsx` for a smoother TypeScript debugging experience.

---

## **Final Thoughts**

By following these steps, you've successfully set up **VS Code to debug Node.js applications in an external terminal using Nodemon**. This setup provides a **more flexible and visible debugging experience**, especially for long-running applications.

---

### **Additional Resources**

- [VS Code Debugging Documentation](https://code.visualstudio.com/docs/editor/debugging)
- [Nodemon Documentation](https://nodemon.io/)

Happy Debugging! 🚀
