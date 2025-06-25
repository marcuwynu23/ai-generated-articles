
# 🔗 Bridging Native Java and Web UI in Capacitor (Cloud URL)

When using Capacitor to load a **cloud-based web app (via `loadUrl`)**, you **cannot rely on Capacitor plugins** like `@capacitor/core` or `Plugins.XYZ` — because those plugins require a bundled app (`build` + `copy`). But there's another way.

In this article, you’ll learn how to **call native Java/Kotlin functions from your web UI** using `window.NativeAPI.method()` — just like Electron's `window.api` bridge.

---

## ✅ Use Case

You want to:
- Load your app from `https://yourdomain.com` into a Capacitor WebView
- Call native functions (e.g., camera, logs, storage) from your web app
- Avoid bundling your app during build

---

## 🛠 Step-by-Step: Create a Native Bridge

### 1. Create a Java class with native methods

```java
package com.your.package;

import android.webkit.JavascriptInterface;
import android.util.Log;

public class NativeBridge {

    @JavascriptInterface
    public void doSomething(String input) {
        Log.d("NativeBridge", "JS says: " + input);
        // Add your native logic here (e.g., open camera, toast, etc.)
    }
}
```

### 2. Inject the bridge into the WebView

In your `MainActivity.java`:

```java
import android.os.Bundle;
import com.getcapacitor.BridgeActivity;

public class MainActivity extends BridgeActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Inject the bridge into the WebView
        this.getBridge().getWebView().addJavascriptInterface(new NativeBridge(), "NativeAPI");

        // If you're loading from a remote URL
        this.getBridge().getWebView().loadUrl("https://yourdomain.com");
    }
}
```

> The second argument `"NativeAPI"` is the object exposed to JS as `window.NativeAPI`.

---

### 3. Call from your JavaScript (Web UI)

Anywhere in your React/Vue/JS app:

```js
if (window.NativeAPI && window.NativeAPI.doSomething) {
  window.NativeAPI.doSomething("Hello from Web UI!");
} else {
  console.warn("NativeAPI not available");
}
```

---

## 🧪 Output in Android Logcat

```
D/NativeBridge: JS says: Hello from Web UI!
```

---

## 🔁 (Optional) Call JS from Native (Reverse)

Want to call JS back from Java?

```java
this.getBridge().getWebView().post(() ->
  this.getBridge().getWebView().evaluateJavascript("window.alert('Hi from native!')", null)
);
```

---

## 🔒 Security Tip

For production:
- Always validate and sanitize JS input
- You can restrict which origins can access `NativeAPI`

---

## 🧩 Real-World Use Cases

You can expand this approach to:

| Use Case         | Native Code                                   |
|------------------|-----------------------------------------------|
| Take picture     | Use `Camera` and `MediaStore` APIs            |
| Toast message    | `Toast.makeText(...).show()`                  |
| Log and debug    | `Log.d(...)`                                  |
| Bluetooth/Serial | Use Android Bluetooth APIs                    |
| Save files       | Use internal storage and expose paths to JS   |

---

## 🧠 Summary

| If you use…                          | Then use…                              |
|-------------------------------------|----------------------------------------|
| `capacitor://` with bundled build   | `@capacitor/core` + plugins            |
| `loadUrl("https://yourdomain.com")` | `addJavascriptInterface()` + `window.NativeAPI` |

This approach gives you full control of native ↔ web integration — just like **Electron’s `window.api`**, but in Android via Capacitor.

---

## 📦 Sample File Structure

```
android/
└── app/
    └── src/
        └── main/
            └── java/
                └── com/
                    └── your/
                        └── package/
                            ├── MainActivity.java
                            └── NativeBridge.java
```

---

## 🚀 You're Done!

Now you can:
- Load your hosted app from the cloud
- Call native Android methods from JavaScript
- Build true hybrid apps without bundling frontend
