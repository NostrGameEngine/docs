In NGE, every user is inherently identified by their Nostr public key. 

However, for most games you'll want a more user-friendly handle, as well as access to metadata like avatars or display names. To bridge that gap, NGE extends the Nostr protocol’s metadata events and exposes them via the `PlayerManagerComponent`. 

With this component in place, you can look up any peer, by key, connection, or channel, and retrieve their up-to-date profile information in a way that feels natural for game development.

!!! note
    These abstractions let you treat Nostr identities like “normal” player objects in your game, complete with avatars, names, and other metadata.

## Registering the PlayerManagerComponent

!!! abstract "See [PlayerManagerComponent Javadoc](https://javadoc.ngengine.org/org/ngengine/player/PlayerManagerComponent.html)"

First, add the `PlayerManagerComponent` to your application’s `ComponentManager`. The `PlayerManagerComponent` needs a list of Nostr Relays that will be used to fetch and store player metadata, you can use any relay, but if you want to access the user's own existing metadata shared with the rest of the nostr network, you should use at least one of the well-known relays, where it is likely to be found.

Adding a component is is covered in [Getting Started](../getting-started.md), but here’s a quick refresher:

```java
Runnable appBuilder = NGEApplication.createApp(settings, app -> {
    ComponentManager mng = app.getComponentManager();

    // List of Nostr relays used to fetch player metadata
    Collection<String> metadataRelays = List.of(
        "wss://relay.ngengine.org",
        "wss://relay.snort.social",
        "wss://nostr.wine",
        "wss://nos.lol",
        "wss://relay.damus.io",
        "wss://relay.primal.net"
    );

    // Add the PlayerManagerComponent (must be done before using PlayerManager)
    mng.addAndEnableComponent(new PlayerManagerComponent(metadataRelays));

    // …other components…
});
appBuilder.run();
```

Once registered, you can retrieve the `PlayerManagerComponent` from any other component as seen in [Components section](./components/index.md):

```java
PlayerManagerComponent playerComp = mng.getComponent(PlayerManagerComponent.class);
PlayerManager playerManager = playerComp.getPlayerManager();
```

Be sure to make your other components [depend on the `PlayerManagerComponent`](../getting-started.md#using-components-from-components-and-dependencies) if you need to access player data. This ensures the `PlayerManagerComponent` is initialized before your component runs.


## Fetching Player Instances

The `PlayerManagerComponent` provides several `getPlayer(...)` methods you can use to fetch a player from different source objects:

```java
// By public key
Player p1 = playerManager.getPlayer(
    NostrPublicKey.fromBech32("npub1…")
);

// By signer (local user)
LocalPlayer me = playerManager.getPlayer(localSigner);

// By network connection
Player remote1 = playerManager.getPlayer((HostedConnection) conn);
Player remote2 = playerManager.getPlayer((RemotePeer) peer);

// By P2P channel
LocalPlayer chanPlayer = playerManager.getPlayer(p2pChannel);
```

!!! abstract "See [LocalPlayer Javadoc](https://javadoc.ngengine.org/org/ngengine/player/LocalPlayer.html) and [Player Javadoc](https://javadoc.ngengine.org/org/ngengine/player/Player.html)"

* **`Player`** read-only view of a remote peer’s profile.
* **`LocalPlayer`**  extends `Player` with setters, representing the local user identity (as a player, it is "your own identity").


!!! tip
    All fetched players are cached, so if you fetch the same player twice, you’ll get the same object instance (upgraded to `LocalPlayer` when applicable).


---

## Accessing and Updating Metadata

Loading metadata from relays is inherently asynchronous. 
NGE’s `Player` class provides sensible defaults immediately (e.g. placeholder name/avatar) and fills in real data as it arrives.

### Listening for Metadata Updates

Register an update listener to react when metadata has finished loading:

```java
player.addUpdateListener(() -> {
    System.out.printf("Metadata ready: %s – %s\n",player.getName(), player.getAvatarUrl());
    return CallbackPolicy.REMOVE_AFTER_CALL;  // one-time notification
});
```

!!! tip
    If you bind the player’s avatar `Texture` to a material in NGE, the engine will automatically swap in the real image once it arrives, no manual refresh needed.

### Blocking Until Ready

If you must have all metadata before proceeding , you can block the current thread:

```java
player.ensureReady();  // blocks until name, avatar, and other fields are loaded
```

!!! warning
     Blocking on the main thread can cause frame drops or UI freezes. Use sparingly.

### Modifying LocalPlayer Data

When you fetch the local player, you can update fields directly:

```java
LocalPlayer me = playerManager.getPlayer(mySigner);
me.setName("CoolGamer123");
me.setAvatarUrl("https://example.com/my-avatar.png");
me.pushMetadataUpdate();  // broadcasts your new metadata to relays
```

---

## Gamertag Conventions

A **gamertag** is a user-friendly handle that combines a chosen name with a short discriminator. It’s suitable for display or search, but **not** as a unique identifier (always use the Nostr public key for security).

A valid gamertag must:

1. Have exactly two parts, separated by `#`:
   * **Name** (3–21 characters)
   * **Discriminator** (4 digits)
2. Use only alphanumeric characters or underscores (`[A-Za-z0-9_]`) in the name.
3. Match the discriminator to the first four characters of the user’s Bech32 public key payload (characters 5–8 of the Nostr public key).

!!! example "Valid Gamertag"
    For key `npub1abcd1234…`, a valid gamertag could be `OstrichSlayer#1234`.

 