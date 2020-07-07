## Configure React Native Development Environment
1. Install NodeJS
> TODO
2. Install Java SDK
>TODO
3. Install Android SDK Manager
   - Install 
5. Install Yarn
>TODO

## Install React Native Camera
1. Install module
```
$ yarn add react-native-camera
```
2. Add permissions to `[project root]/android/src/main/AndroidManifest.xml` file
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

+    <uses-permission android:name="android.permission.INTERNET" />
+    <uses-permission android:name="android.permission.CAMERA" />
+    <uses-permission android:name="android.permission.RECORD_AUDIO" />
+    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
+    <uses-permission android:name="andorid.permission.WRITE_EXTERNAL_STORAGE" />
...
</manifest>
```
3. Set dimension strategy in `[project root]/android/app/build.gradle` file
```gradle
android {
    ...
    defaultConfig {
        ...
+       missingDimensionStrategy 'react-native-camera', 'general'
        ...
    }
    ...
}
```

## Install Anyline OCR SDK
1. Install module
```
$ yarn add anyline-ocr-react-native-module`
```
2. Line module with React Native
```
$ react-native link anyline-ocr-react-native-module
```
3. Change minimum SDK version in `[project root]/android/build.gradle` file
    
```gradle
buildscript {
ext {
    minSdkVersion = 21
    ....
}
```
4. Add Maven respository in `[project root]/android/build.gradle` file
```gradle
allprojects {
    repositories {
        ...
+       maven{
+           url 'https://anylinesdk.blob.core.windows.net/maven/'
+       }
    }
}
```
5. Change allowBackup attribute in `[project root]/android/app/src/main/AndroidManifest.xml` file
```xml
<manifest>
    ...
    <application
        ...
        android:allowBackup="true">
        ...
    </application>
</manifest>
```

6. Add permissions to `[project root]/android/app/src/main/AndroidManifest.xml`
```xml
+    <uses-permission android:name="android.permission.INTERNET" />
+    <uses-permission android:name="android.permission.CAMERA" />
+    <uses-permission android:name="android.permission.VIBRATE"/>
```
7. Add dependencies to `[project root/android/app/build.gradle]`
```gradle
    packagingOptions {
+        exclude 'META-INF/proguard/androidx-annotations.pro'
+        pickFirst 'lib/armeabi-v7a/libgnustl_shared.so'
+        pickFirst 'lib/x86/libgnustl_shared.so'
+        pickFirst 'lib/*/libc++_shared.so'
    }
```
## Rename React Native application ID
1. Change application id in `[project root]/android/app/build.gradle`
```gradle
android {
    ...
    defaultConfig {
        applicationId "com.[new.application.name]"
```
2. Change package name in `[project root]/android/app/src/main/AndroidManifest.xml`
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.[new.application.name]">
```
3. Create new directory under `[project root]/android/app/src/main/java/com`
```
$ mkdir new/application/name
```
4. Move java files from older folder to new folder
```
$ mv [project root]/android/app/src/main/java/com/[old/application/name] [project root]/android/app/src/main/java/com/[new/aplication/name]/
```
5. Change package name in `[project root]/android/app/src/main/com/[new/application/name/MainActivity.java]`
```java
package com.[new.application.name]
```

6. Change package name in `[project root]/andorid/app/src/main/com/[new/application/name/MainApplication.java]`
```java
package com.[new.application.name]
```