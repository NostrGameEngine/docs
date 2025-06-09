
**NGE** is a modern Java library, you can build and distribute your application in several ways, depending on your target platform and user base.

!!! tip
    The [template app](./getting-started.md) includes a **GitHub Actions** workflow that builds for all supported platforms using every available methods. 
    You can use this as a reference or adapt it directly for your own project.



## Building for Desktop

### Portable JAR

To build a standalone JAR file:


```bash
./gradlew build
```

or on Windows:

```bash
gradlew.bat build
```

The output will be located at:

```
dist/portable/yourapp-version-portable.jar
```

This is the traditional approach to distributing Java applications. It's simple, cross-platform, and requires only a single build step. 

However, it has a few limitations:

* Slightly longer startup time compared to native executables
* Requires users to have **Java Runtime Environment (JRE) version 21 or higher** installed

While you *can* manually bundle a JRE with your app, there's a better alternative, discussed next.


### Installer

This method packages the portable JAR **along with a JRE**, and wraps them in a platform-specific installer. It simplifies distribution and ensures your users don't need to install Java separately.


```bash
./gradlew buildInstaller
```

or on Windows:

```bash
gradlew.bat buildInstaller
```

The output will be located at:

```
dist/yourapp-version-platform.setup
```

!!! note
     You can only create an installer for the platform you’re currently on. To support multiple platforms, you’ll need to run this process on each one individually.



### Native Executable (Recommended)

The most optimized way to distribute your app is as a **native executable**, which provides:

* Significantly faster startup times
* No need for users to install a JRE
* No need to bundle a JRE with your app

!!! note
    Native builds require **GraalVM 24 or higher**. 
    
    You can install it easily using [SDKMAN](https://sdkman.io/):

    ```bash
    sdk install java 24-graal
    ```


Depending on the complexity of your app, you might need to run the *tracing agent* to help GraalVM understand your app's dependencies. 

To do that you need to run  

```bash
./gradlew traceNative
```

and use all the features of your app. This will update the tracing files for GraalVM.

When you are ready you can build the native executable with:


```bash
./gradlew buildNativeExecutable
```

or on Windows:

```bash
gradlew.bat buildNativeExecutable
```

The output will be located at:

```
dist/yourapp-version-platform.buildNative/
```

This directory includes the executable along with all required dependencies and resources that you will need to ship alongside with your app.


!!! note
    Native executables can **only** be built on the platform you're currently running. GraalVM does **not** support cross-compilation yet.



## Building for Android

TBD

## Building for the Web (Browser)

TBD


## Building for iOS
TBD