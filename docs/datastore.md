
The `DataStore` class provides a platform-agnostic key-value storage interface for saving and retrieving primitives, `Serializable` and `Savable` objects. It abstracts the underlying storage mechanism via [VStore](https://ngengine.org/nge-platforms/org/ngengine/platform/VStore.html), enabling consistent data persistence whether you're caching temporary data or storing it permanently.


## Creating a DataStore

!!! abstract "See [DataStore Javadoc](https://javadoc.ngengine.org/org/ngengine/store/DataStore.html)"

You can manually create a `DataStore` using:

```java
DataStore dataStore = new DataStore(/* String */ appName, /* AssetManager */ assetManager, /* String */ dataStoreName, /* boolean */ isCache);
```

However, if you're using the [a Component](./components/index.md), you’ll typically receive a preconfigured `DataStoreProvider`. This simplifies the process:

```java
DataStore dataStore = dataStoreProvider.getDataStore("myDataStore");      // For permanent storage
DataStore cacheStore = dataStoreProvider.getCacheStore("myDataStore");   // For cache storage
```

!!! abstract "See [DataStoreProvider Javadoc](https://javadoc.ngengine.org/org/ngengine/store/DataStoreProvider.html)"

## Using a DataStore

`DataStore` uses a simple key-value interface to store and retrieve data. You can store any object that implements either `Serializable` or `Savable`, or is a primitive, and later retrieve it as a fully reconstructed instance.

!!! example

    ```java
    DataStore dataStore = dataStoreProvider.getDataStore("myDataStore");

    // Serializable object
    String myString = "Hello, NGE!";

    // Savable object
    Vector3f myVector = new Vector3f(1, 2, 3);

    // Store data
    dataStore.write("myStringKey", myString);
    dataStore.write("myVectorKey", myVector);

    // Retrieve data
    String retrievedString = dataStore.read("myStringKey");
    Vector3f retrievedVector = dataStore.read("myVectorKey");
    ```

 When retrieving data, the DataStore attempts to load it from assets via `AssetManager` using these default paths:

* `cache/` — if the `DataStore` is marked as a cache store
* `data/` — for non-cache (persistent) stores

This allows you to pre-generate runtime data or caches and include them as part of your shipped assets.

!!! tip
    Pre-generating and shipping cache data can significantly improve first-time load performance.


## Accessing the Underlying VStore

!!! abstract "See [VStore Javadoc](https://ngengine.org/nge-platforms/org/ngengine/platform/VStore.html)"

In some cases, working directly with object serialization isn’t enough. You may want to use raw java input/output streams for more granular control. While you can create a `VStore` manually using the `nge-platform` module, `DataStore` provides access to its internal `VStore` for convenience:

```java
VStore vStore = dataStore.getVStore();
```

This gives you low-level access while maintaining a consistent interface across your project.
 