
# How to Build an Android App Using SDK Tools (aapt, adb, etc.) Step-by-Step

In this article, we will manually build an Android app using the Android SDK tools such as `aapt`, `adb`, `dx`, and more. This process will help you understand how Android apps are built under the hood, which is usually automated by build systems like Gradle or Android Studio.

## Prerequisites

1. **Install Android SDK Tools**: Ensure you have the Android SDK installed, including the `build-tools` (which contains tools like `aapt`, `adb`, `dx`, `zipalign`, `apksigner`, etc.).
   
   You can download the Android SDK [here](https://developer.android.com/studio#downloads) if you don't have it already.

2. **Set Environment Variables**: Set up the `ANDROID_HOME` environment variable and add the `platform-tools` and `build-tools` directories to your `PATH`.

3. **Create a Basic Android Project Structure**:
   You'll need to manually create a simple Android project structure similar to what Android Studio does.

## Step-by-Step Process

### 1. Create Project Directory Structure

Start by creating the required folders:

```
MyAndroidApp/
├── AndroidManifest.xml
├── res/
│   ├── layout/
│   │   └── activity_main.xml
│   └── values/
│       └── strings.xml
├── src/
│   └── com/
│       └── example/
│           └── myapp/
│               └── MainActivity.java
```

- **AndroidManifest.xml**: Describes the structure of the app, its components (like activities), permissions, etc.
- **res/**: Contains resources such as layouts, strings, images, etc.
- **src/**: Contains the source code for the application.

### 2. Write the Manifest (`AndroidManifest.xml`)

Here’s an example of a simple manifest file:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">
    
    <application
        android:label="@string/app_name"
        android:icon="@mipmap/ic_launcher">
        
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
    
</manifest>
```

### 3. Write the Layout (`res/layout/activity_main.xml`)

Define the layout for your activity (UI components):

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/hello_world" />
    
</LinearLayout>
```

### 4. Define Strings (`res/values/strings.xml`)

```xml
<resources>
    <string name="app_name">MyApp</string>
    <string name="hello_world">Hello World!</string>
</resources>
```

### 5. Write the MainActivity (`src/com/example/myapp/MainActivity.java`)

```java
package com.example.myapp;

import android.app.Activity;
import android.os.Bundle;
import android.widget.TextView;

public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        TextView textView = findViewById(R.id.textView);
        textView.setText("Hello from MainActivity!");
    }
}
```

### 6. Compile the Java Code (`javac`)

Compile the Java source code into `.class` files:

```bash
javac -source 1.7 -target 1.7 -bootclasspath $ANDROID_HOME/platforms/android-29/android.jar -d obj src/com/example/myapp/MainActivity.java
```

### 7. Convert `.class` Files to `.dex` (`dx`)

Convert the compiled Java `.class` files to Dalvik Executable (DEX) format using the `dx` tool:

```bash
$ANDROID_HOME/build-tools/<version>/dx --dex --output=classes.dex obj/
```

This produces the `classes.dex` file, which is the Dalvik bytecode that will run on the Android device.

### 8. Package Resources (`aapt`)

Package the resources (layouts, strings, etc.) and the manifest into an APK. This step creates the `resources.arsc` file, which contains compiled resources.

```bash
$ANDROID_HOME/build-tools/<version>/aapt package -f -m -J src -M AndroidManifest.xml -S res -I $ANDROID_HOME/platforms/android-29/android.jar -F MyApp.apk.unaligned
```

- `-M`: The manifest file.
- `-S`: The `res/` folder.
- `-I`: Android’s core framework JAR (from SDK).

### 9. Add the DEX File to the APK

Add the `classes.dex` file (generated from your compiled Java code) to the APK:

```bash
$ANDROID_HOME/build-tools/<version>/aapt add MyApp.apk.unaligned classes.dex
```

### 10. Align the APK (`zipalign`)

Before signing the APK, it must be aligned for optimal memory usage. Use the `zipalign` tool to do this:

```bash
$ANDROID_HOME/build-tools/<version>/zipalign -v 4 MyApp.apk.unaligned MyApp.apk
```

### 11. Sign the APK (`apksigner`)

To install the APK on a device, it must be signed. Use `apksigner` to sign the APK with a debug key or a production key (for real apps).

```bash
$ANDROID_HOME/build-tools/<version>/apksigner sign --ks my-release-key.jks --out MyApp-signed.apk MyApp.apk
```

- If you're using a debug key, use the default `debug.keystore` file:
  ```bash
  $ANDROID_HOME/build-tools/<version>/apksigner sign --ks ~/.android/debug.keystore --out MyApp-signed.apk MyApp.apk
  ```

### 12. Install the APK on the Device (`adb`)

Finally, use `adb` to install the APK onto a connected Android device or emulator:

```bash
adb install MyApp-signed.apk
```

### 13. Run the App

After installing, you can run the app manually from the device or through `adb`:

```bash
adb shell am start -n com.example.myapp/.MainActivity
```

This should launch your Android app on the device!

## Summary of Tools

- **`aapt`**: Android Asset Packaging Tool, used to compile and package resources.
- **`dx`**: Converts Java `.class` files to Dalvik bytecode (`.dex`).
- **`zipalign`**: Optimizes the APK for memory usage.
- **`apksigner`**: Signs the APK so it can be installed on an Android device.
- **`adb`**: Android Debug Bridge, used to install APKs, run apps, and interact with Android devices.

This manual process is equivalent to what the Android Gradle build system automates. You now have a deeper understanding of how Android apps are built under the hood using the command-line tools.
